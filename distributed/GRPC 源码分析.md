# GRPC 源码分析

## GRPC server

首先从一个简单的示例开始说起

```go
// server启动时候绑定的端口号
const (
	port = ":50051"
)

// server is used to implement helloworld.GreeterServer in Protobuf 
type server struct {
	pb.UnimplementedGreeterServer
}

// SayHello implements helloworld.GreeterServer
func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
	log.Printf("Received: %v", in.GetName())
	return &pb.HelloReply{Message: "Hello " + in.GetName()}, nil
}

func main() {
	lis, err := net.Listen("tcp", port)
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
  	// NewServer creates a gRPC server which has no service registered and has not
	// started to accept requests yet.
  	// 创建server示例，通过ServerOption列表给server中的opts对象和拦截器(interceptors)
	s := grpc.NewServer()
 	// 注册server， 
	pb.RegisterGreeterServer(s, &server{})
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
```

```sequence {}
Note over Server: listen port
Note over Server: s:=grpc.NewServer()
Server-->pb: pb.RegisterGreeterServer(s, &server{})
pb-->Server: s.RegisterService(&_Greeter_serviceDesc, srv)
Note over Server: srv is s?
Note left of Server: No, panic!
Note over Server: Yes, register service by name
Note over Server: start server
Note over Server: wait for conn
Client-->Server: conn
Note over Server: create a new goroutine \n deal conn
Note over Server: set timeout by opts \n valid creds \n sets up a http/2 transport \n cancel timeout
Note over Server: create a new goroutine \n valid creds \n sets up a http/2 transport \n cancel timeout
```

<details>  
<summary>s := grpc.NewServer()</summary>
<div class="highlight highlight-source-go">
<pre><code>

	func NewServer(opt ...ServerOption) *Server {
		opts := defaultServerOptions
		for _, o := range opt {
			o.apply(&opts)
		}
		s := &Server{
			lis:    make(map[net.Listener]bool),
			opts:   opts,
			conns:  make(map[transport.ServerTransport]bool),
			m:      make(map[string]*service),
			quit:   grpcsync.NewEvent(),
			done:   grpcsync.NewEvent(),
			czData: new(channelzData),
		}
		chainUnaryServerInterceptors(s)
		chainStreamServerInterceptors(s)
		s.cv = sync.NewCond(&s.mu)
		if EnableTracing {
			_, file, line, _ := runtime.Caller(1)
			s.events = trace.NewEventLog("grpc.Server", fmt.Sprintf("%s:%d", file, line))
		}
	
		if s.opts.numServerWorkers > 0 {
			s.initServerWorkers()
		}
	
		if channelz.IsOn() {
			s.channelzID = channelz.RegisterServer(&channelzServer{s}, "")
		}
		return s
	}

</code></pre>
</div>
</details>


<details>  
<summary>pb.RegisterGreeterServer(s, &server{})</summary>
<div class="highlight highlight-source-go">
<pre><code>
	
	func RegisterGreeterServer(s *grpc.Server, srv GreeterServer) {
		s.RegisterService(&_Greeter_serviceDesc, srv)
	}
	
	// RegisterService registers a service and its implementation to the gRPC
	// server. It is called from the IDL generated code. This must be called before
	// invoking Serve.
	func (s *Server) RegisterService(sd *ServiceDesc, ss interface{}) {
		ht := reflect.TypeOf(sd.HandlerType).Elem()
		st := reflect.TypeOf(ss)
		if !st.Implements(ht) {
			logger.Fatalf("grpc: Server.RegisterService found the handler of type %v that does not satisfy %v", st, ht)
		}
		s.register(sd, ss)
	}
	
	func (s *Server) register(sd *ServiceDesc, ss interface{}) {
		s.mu.Lock()
		defer s.mu.Unlock()
		s.printf("RegisterService(%q)", sd.ServiceName)
		if s.serve {
			logger.Fatalf("grpc: Server.RegisterService after Server.Serve for %q", sd.ServiceName)
		}
		if _, ok := s.m[sd.ServiceName]; ok {
			logger.Fatalf("grpc: Server.RegisterService found duplicate service registration for %q", sd.ServiceName)
		}
		srv := &service{
			server: ss,
			md:     make(map[string]*MethodDesc),
			sd:     make(map[string]*StreamDesc),
			mdata:  sd.Metadata,
		}
		for i := range sd.Methods {
			d := &sd.Methods[i]
			srv.md[d.MethodName] = d
		}
		for i := range sd.Streams {
			d := &sd.Streams[i]
			srv.sd[d.StreamName] = d
		}
		s.m[sd.ServiceName] = srv
	}
</code></pre></div>
</details>

<details>  
<summary>s.Serve(lis)</summary>
<div class="highlight highlight-source-go">
<pre><code>

	func (s *Server) Serve(lis net.Listener) error {
		s.mu.Lock()
		s.printf("serving")
		s.serve = true
		if s.lis == nil {
			// Serve called after Stop or GracefulStop.
			s.mu.Unlock()
			lis.Close()
			return ErrServerStopped
		}

		s.serveWG.Add(1)
		defer func() {
			s.serveWG.Done()
			if s.quit.HasFired() {
				// Stop or GracefulStop called; block until done and return nil.
				<-s.done.Done()
			}
		}()
	
		ls := &listenSocket{Listener: lis}
		s.lis[ls] = true
	
		if channelz.IsOn() {
			ls.channelzID = channelz.RegisterListenSocket(ls, s.channelzID, lis.Addr().String())
		}
		s.mu.Unlock()
	
		defer func() {
			s.mu.Lock()
			if s.lis != nil && s.lis[ls] {
				ls.Close()
				delete(s.lis, ls)
			}
			s.mu.Unlock()
		}()
	
		var tempDelay time.Duration // how long to sleep on accept failure
	
		for {
			rawConn, err := lis.Accept()
			if err != nil {
				if ne, ok := err.(interface {
					Temporary() bool
				}); ok && ne.Temporary() {
					if tempDelay == 0 {
						tempDelay = 5 * time.Millisecond
					} else {
						tempDelay *= 2
					}
					if max := 1 * time.Second; tempDelay > max {
						tempDelay = max
					}
					s.mu.Lock()
					s.printf("Accept error: %v; retrying in %v", err, tempDelay)
					s.mu.Unlock()
					timer := time.NewTimer(tempDelay)
					select {
					case <-timer.C:
					case <-s.quit.Done():
						timer.Stop()
						return nil
					}
					continue
				}
				s.mu.Lock()
				s.printf("done serving; Accept = %v", err)
				s.mu.Unlock()
	
				if s.quit.HasFired() {
					return nil
				}
				return err
			}
			tempDelay = 0
			// Start a new goroutine to deal with rawConn so we don't stall this Accept
			// loop goroutine.
			//
			// Make sure we account for the goroutine so GracefulStop doesn't nil out
			// s.conns before this conn can be added.
			s.serveWG.Add(1)
			go func() {
				s.handleRawConn(rawConn)
				s.serveWG.Done()
			}()
		}
	}
</code></pre></div>
</details>

```go
// Server is a gRPC server to serve RPC requests.
type Server struct {
	opts serverOptions
	'''  //省略
	m      map[string]*service // service name -> service info
	events trace.EventLog
	'''  //省略
	serverWorkerChannels []chan *serverWorkerData
}

type serverOptions struct {
	creds                 credentials.TransportCredentials
	codec                 baseCodec
	cp                    Compressor
	dc                    Decompressor
	unaryInt              UnaryServerInterceptor
	streamInt             StreamServerInterceptor
	chainUnaryInts        []UnaryServerInterceptor
	chainStreamInts       []StreamServerInterceptor
	inTapHandle           tap.ServerInHandle
	statsHandler          stats.Handler
	maxConcurrentStreams  uint32
	maxReceiveMessageSize int
	maxSendMessageSize    int
	unknownStreamDesc     *StreamDesc
	keepaliveParams       keepalive.ServerParameters
	keepalivePolicy       keepalive.EnforcementPolicy
	initialWindowSize     int32
	initialConnWindowSize int32
	writeBufferSize       int
	readBufferSize        int
	connectionTimeout     time.Duration
	maxHeaderListSize     *uint32
	headerTableSize       *uint32
	numServerWorkers      uint32
}
```



## GRPC client



### GRPC gateway



## GRPC middleware

鉴权

限流

监控

日志

拦截器