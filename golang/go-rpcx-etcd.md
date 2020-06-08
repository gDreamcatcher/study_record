```go
s := server.NewServer()
addRegistryPlugin(s) // 添加etcd插件

func addRegistryPlugin(s *server.Server) {
	r := &serverplugin.EtcdV3RegisterPlugin{
		ServiceAddress: "tcp@" + *addr,   //服务的地址，注册到etcd的值
		EtcdServers:    []string{*etcdAddr},  //etcd服务的地址
		BasePath:       *basePath,  //base path for rpcx server, for example com/example/rpcx
		UpdateInterval: time.Minute,
	}
	err := r.Start()
	if err != nil {
		log.Fatal(err)
	}
	s.Plugins.Add(r)
}

// github.com/smallnest/rpcx/serverplugin/etcdv3.go 47 line
// Start starts to connect etcd cluster
func (p *EtcdV3RegisterPlugin) Start() error {
	if p.done == nil {
		p.done = make(chan struct{})
	}
	if p.dying == nil {
		p.dying = make(chan struct{})
	}

	if p.kv == nil {
		kv, err := libkv.NewStore(etcd.ETCDV3, p.EtcdServers, p.Options)
		if err != nil {
			log.Errorf("cannot create etcd registry: %v", err)
			return err
		}
		p.kv = kv
	}

	err := p.kv.Put(p.BasePath, []byte("rpcx_path"), &store.WriteOptions{IsDir: true})
	if err != nil && !strings.Contains(err.Error(), "Not a file") {
		log.Errorf("cannot create etcd path %s: %v", p.BasePath, err)
		return err
	}

	if p.UpdateInterval > 0 {
		ticker := time.NewTicker(p.UpdateInterval)
		go func() {
			defer p.kv.Close()

			// refresh service TTL
			for {
				select {
				case <-p.dying:
					close(p.done)
					return
				case <-ticker.C:
					var data []byte
					if p.Metrics != nil {
						clientMeter := metrics.GetOrRegisterMeter("clientMeter", p.Metrics)
						data = []byte(strconv.FormatInt(clientMeter.Count()/60, 10))
					}
					//set this same metrics for all services at this server
					for _, name := range p.Services {
						nodePath := fmt.Sprintf("%s/%s/%s", p.BasePath, name, p.ServiceAddress)
						kvPair, err := p.kv.Get(nodePath)
						if err != nil {
							log.Infof("can't get data of node: %s, because of %v", nodePath, err.Error())

							p.metasLock.RLock()
							meta := p.metas[name]
							p.metasLock.RUnlock()

							err = p.kv.Put(nodePath, []byte(meta), &store.WriteOptions{TTL: p.UpdateInterval * 2})
							if err != nil {
								log.Errorf("cannot re-create etcd path %s: %v", nodePath, err)
							}

						} else {
							v, _ := url.ParseQuery(string(kvPair.Value))
							v.Set("tps", string(data))
							p.kv.Put(nodePath, []byte(v.Encode()), &store.WriteOptions{TTL: p.UpdateInterval * 2})
						}
					}
				}
			}
		}()
	}

	return nil
}
```

```
// New creates a new Etcd client given a list
// of endpoints and an optional tls config
func New(addrs []string, options *store.Config) (store.Store, error) {
	s := &Etcd{}

	var (
		entries []string
		err     error
	)

	entries = store.CreateEndpoints(addrs, "http")
	cfg := &etcd.Config{
		Endpoints:               entries,
		Transport:               etcd.DefaultTransport,
		HeaderTimeoutPerRequest: 3 * time.Second,
	}

	// Set options
	if options != nil {
		if options.TLS != nil {
			setTLS(cfg, options.TLS, addrs)
		}
		if options.ConnectionTimeout != 0 {
			setTimeout(cfg, options.ConnectionTimeout)
		}
		if options.Username != "" {
			setCredentials(cfg, options.Username, options.Password)
		}
	}

	c, err := etcd.New(*cfg)
	if err != nil {
		log.Fatal(err)
	}

	s.client = etcd.NewKeysAPI(c)

	// Periodic Cluster Sync
	go func() {
		for {
			if err := c.AutoSync(context.Background(), periodicSync); err != nil {
				return
			}
		}
	}()

	return s, nil
}
```

