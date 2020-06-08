

[Go RPC 开发指南](https://books.studygolang.com/go-rpc-programming-guide/ )

[rpcx](https://rpcx.io/)

特性：

- > 简单易用 

- > 高性能  性能远高于Dubbo、Motan、Thrift等框架，是grpc性能的两倍

- > 交叉平台 支持多平台部署 多语言调用

- > 服务发现 除了直连外还支持Zookeeper、Etcd、Consul、mDNS等注册中心

- > 服务治理 支持Failover、Failfast、Failtry、Backup等失败模式，支持随机、轮询、权重、网络质量，一致性哈希，地理位置等路由算法

benchmark：

> 测试环境[CPU,Memory, Go version, OS]和测试结果[TPS，Latency：mean time， Latency：middle time]



启动server

```go
addr := flag.String("addr", "localhost:8972", "server address")
func Server(){
  // 生成一个server
	s := server.NewServer()
  // 注册
	s.Register(new(Arith), "")
	s.Serve("tcp", *addr)
}
```

**创建server对象**

**注册**

```
//注册的函数有四个,分为两类: 注册对象中的符合要求的所有方法, 注册某一个方法
// 注册对象中的符合要求的所有方法，调用的时候通过[类型名或指定的name]+方法名 调用 
Register(rcvr interface{}, metadata string) error; // 自动将类型的名字注册
RegisterName(name string, rcvr interface{}, metadata string) error //修改注册名为指定的name
//注册某一个方法,调用的时候通过servicePath + [方法名或指定的name] 调用 
RegisterFunction(servicePath string, fn interface{}, metadata string) error
RegisterFunctionName(servicePath string, name string, fn interface{}, metadata string) error
```

符合要求的方法：

1. method.PkgPath == ""
2. 函数参数为4个<!--其实是三个，第一个必须是context, 最后一个必须是指针-->
3. 函数有一个返回值，必须是error类型

Register方法将类型中符合要求的方法都注册到service.method集合中，并把第二个参数和第三个参数的类型注册到typePools.pools中，如果这个类型没有方法将返回错误。最后将这个类型的名字注册到server.serviceMap中。



启动client

```go
func Client(){
	option := client.DefaultOption
	option.SerializeType = protocol.ProtoBuffer

	d := client.NewPeer2PeerDiscovery("tcp@"+*addr, "")
	xclient := client.NewXClient("Arith", client.Failfast, client.RandomSelect, d, option)
	defer xclient.Close()

	args := &pb.ProtoArgs{A: 10, B: 20}
	reply := &pb.ProtoReply{}
	if err := xclient.Call(context.Background(), "Muls", args, reply); err != nil{
		fmt.Println(err)
	}
	log.Printf("%d * %d = %d", args.A, args.B, reply.C)
}
```

启动httpclient

```go
func HttpClient() {
	args := &pb.ProtoArgs{A: 10, B: 20}
	reply := HttpToTcp(args)
	log.Printf("%d * %d = %d", args.A, args.B, reply.C)
}

func HttpToTcp(args *pb.ProtoArgs) *pb.ProtoReply {
	reply := &pb.ProtoReply{}

	cc := codec.MsgpackCodec{}
	data, _ := cc.Encode(args)
	req, err := http.NewRequest("POST", "http://127.0.0.1:8972/", bytes.NewReader(data))
	if err != nil {
		log.Fatal("failed to create request: ", err)
	}
	h := req.Header
	h.Set(gateway.XMessageID, "10000")
	h.Set(gateway.XMessageType, "0")
	h.Set(gateway.XSerializeType, "3")
	h.Set(gateway.XServicePath, "Arith")
	h.Set(gateway.XServiceMethod, "Mul")

	res, err := http.DefaultClient.Do(req)
	if err != nil {
		log.Fatal("failed to read response: ", err)
	}
	defer res.Body.Close()
	replyData, err := ioutil.ReadAll(res.Body)
	err = cc.Decode(replyData, reply)
	if err != nil {
		log.Fatal("failed to decode reply: ", err)
	}

	return reply
}
```



通过http访问tcp接口

```go
func HttpServer(){
	r := gin.Default()
	r.GET("/mul", CallMul)
	r.Run(":8080")
}

func CallMul(ctx *gin.Context){
	a, err := strconv.Atoi(ctx.Query("a"))
	if err != nil {
		panic(err)
	}
	b, err := strconv.Atoi(ctx.Query("b"))
	if err != nil {
		panic(err)
	}
	args := &pb.ProtoArgs{A: int32(a), B: int32(b)}
	reply := HttpToTcp(args)
	log.Printf("%d * %d = %d", args.A, args.B, reply.C)
	ctx.JSON(http.StatusOK, gin.H{"status": "ok", "reply": reply.C})
}
```



### 设置超时

服务端设置超时

```go
s := server.NewServer(server.WithReadTimeout(10 * time.Second), server.WithWriteTimeout(10*time.Second))
```

client设置超时

```
// client的Option可以设置超时
type Option struct {
    //ConnectTimeout sets timeout for dialing
    ConnectTimeout time.Duration
    // ReadTimeout sets readdeadline for underlying net.Conns
    ReadTimeout time.Duration
    // WriteTimeout sets writedeadline for underlying net.Conns
    WriteTimeout time.Duration
}
// context.Context

```



### 插件

Metrics

```
imoprt"github.com/rcrowley/go-metrics"
```

限流

```
import (
	"net"
	"time"

	"github.com/juju/ratelimit"
)

// RateLimitingPlugin can limit connecting per unit time
type RateLimitingPlugin struct {
	FillInterval time.Duration
	Capacity     int64
	bucket       *ratelimit.Bucket
}

// NewRateLimitingPlugin creates a new RateLimitingPlugin
func NewRateLimitingPlugin(fillInterval time.Duration, capacity int64) *RateLimitingPlugin {
	tb := ratelimit.NewBucket(fillInterval, capacity)

	return &RateLimitingPlugin{
		FillInterval: fillInterval,
		Capacity:     capacity,
		bucket:       tb}
}

// HandleConnAccept can limit connecting rate
func (plugin *RateLimitingPlugin) HandleConnAccept(conn net.Conn) (net.Conn, bool) {
	return conn, plugin.bucket.TakeAvailable(1) > 0
}
```

