# GRPC  GO API

###  [常量](https://pkg.go.dev/google.golang.org/grpc?tab=doc#pkg-constants)

```
const (
	SupportPackageIsVersion3 = true
	SupportPackageIsVersion4 = true
	SupportPackageIsVersion5 = true
	SupportPackageIsVersion6 = true
)
```

SupportPackageIsVersion 兼容的版本，目前最新的是version 6

为了兼容性，旧的版本依然支持，但是将来可能被移除。这些变量只是标记兼容的版本，不建议在代码里引用

```
const PickFirstBalancerName = "pick_first"
```

PickFirstBalancerName 见明知意

```
const Version = "1.30.0"
```

当前Grpc的版本

###  [Variables](https://pkg.go.dev/google.golang.org/grpc?tab=doc#pkg-variables)

```
var DefaultBackoffConfig = BackoffConfig{
	MaxDelay: 120 * time.Second,
}
```

DefaultBackoffConfig 默认的当后台连接失败后的处理逻辑，详情参考 https://github.com/grpc/grpc/blob/master/doc/connection-backoff.md.

**不推荐使用：**使用ConnectParams 替代，该变量在1.X版本依然支持，后期可能会被取消

```
var EnableTracing bool
```

EnableTracing  该变量决定是否使用golang.org/x/net/trace库去跟踪RPCs调用，一般debug的时候使用。该变量必须在发送和接受RPCs之前设置。

```
var (
	// ErrClientConnClosing indicates that the operation is illegal because
	// the ClientConn is closing.
	//
	// Deprecated: this error should not be relied upon by users; use the status
	// code of Canceled instead.
	ErrClientConnClosing = status.Error(codes.Canceled, "grpc: the client connection is closing")
)
var ErrClientConnTimeout = errors.New("grpc: timed out when dialing")
```

ErrClientConnTimeout 建立连接超时报的错误

**不推荐使用：**grpc不会返回这个错误，也不建议用户使用这个变量

```
var ErrServerStopped = errors.New("grpc: the server has been stopped")
```

ErrServerStopped 连接中断错误.

### func [Code](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L754) 

```go
func Code(err error) codes.Code
```

Code 返回错误码，如果err的类型不是grpc提供err类型，返回codes.Unknown

**不推荐使用：**建议使用status.Code代替

### func [ErrorDesc](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L762) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ErrorDesc)

```
func ErrorDesc(err error) string
```

ErrorDesc  返回错的描述，如果err的类型不是rpc提供的则返回err.Error()，如果err是nil的话返回空字符串

### func [Errorf](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L770) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#Errorf)

```
func Errorf(c codes.Code, format string, a ...interface{}) error
```

Errorf  根据提供的参数返回一个err.

**不推荐使用：**建议使用status.Errorf .

### func [Invoke](https://github.com/grpc/grpc-go/blob/v1.30.0/call.go#L59) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#Invoke)

```
func Invoke(ctx context.Context, method string, args, reply interface{}, cc *ClientConn, opts ...CallOption) error
```

Invoke 发送请求并接收返回结果

**不推荐使用：**建议使用 ClientConn.Invoke.

### func [Method](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L1718) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#Method)

```
func Method(ctx context.Context) (string, bool)
```

Method  从context中获取method的值，返回结果的格式"/service/method".

### func [MethodFromServerStream](https://github.com/grpc/grpc-go/blob/v1.30.0/stream.go#L1518) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#MethodFromServerStream)

```
func MethodFromServerStream(stream ServerStream) (string, bool)
```

MethodFromServerStream 从输入流中获取method，返回结果的格式"/service/method".

### func [NewContextWithServerTransportStream](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L1536) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#NewContextWithServerTransportStream)

```
func NewContextWithServerTransportStream(ctx context.Context, stream ServerTransportStream) context.Context
```

NewContextWithServerTransportStream 根据参数ctx生成一个新的context 并将steam赋值给它，上面Method方法就是接收这样的ctx.

这个API还不稳定.

### func [SendHeader](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L1692) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#SendHeader)

```go
func SendHeader(ctx context.Context, md metadata.MD) error
// metadata.MD 是一个map[string][]string
// 如果ctx中没有stream返回 status.Errorf(codes.Internal, "grpc: failed to fetch the stream from the context %v", ctx)
```

SendHeader 该方法最多调用一次，这个md和SetHeader()将被发送.

### func [SetHeader](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L1679) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#SetHeader)

```
func SetHeader(ctx context.Context, md metadata.MD) error
```

SetHeader 将md内容设置成header，如果多次调用，所有的md将被整合成一个，当下面的情况发生的时候，header将被发送出

```
- grpc.SendHeader() is called;
- The first response is sent out;
- An RPC status is sent out (error or success).
```

### func [SetTrailer](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L1705) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#SetTrailer)

```
func SetTrailer(ctx context.Context, md metadata.MD) error
```

SetTrailer 将md设置stream的尾部，当rpc返回的时候将发送出，多次调用多个md将被合成一个 //TODO .

### type [BackoffConfig](https://github.com/grpc/grpc-go/blob/v1.30.0/backoff.go#L41) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#BackoffConfig)

```
type BackoffConfig struct {
	// MaxDelay is the upper bound of backoff delay.
	MaxDelay time.Duration
}
```

BackoffConfig 定义默认的gRPC backoff策略的参数.

**不推荐使用：**推荐使用 ConnectParams ，将来可能被删除.

### type [CallOption](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L175) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#CallOption)

```
type CallOption interface {
	// contains filtered or unexported methods
}
```

CallOption 在调用开始前添加配置或在调用结束后抽取信息.

### func [CallContentSubtype](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L377) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#CallContentSubtype)

```
func CallContentSubtype(contentSubtype string) CallOption
```

CallContentSubtype returns a CallOption that will set the content-subtype for a call. For example, if content-subtype is "json", the Content-Type over the wire will be "application/grpc+json". The content-subtype is converted to lowercase before being included in Content-Type. See Content-Type on https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md#requests for more details.

If ForceCodec is not also used, the content-subtype will be used to look up the Codec to use in the registry controlled by RegisterCodec. See the documentation on RegisterCodec for details on registration. The lookup of content-subtype is case-insensitive. If no such Codec is found, the call will result in an error with code codes.Internal.

If ForceCodec is also used, that Codec will be used for all request and response messages, with the content-subtype set to the given contentSubtype here for requests.

### func [CallCustomCodec](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L430) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#CallCustomCodec)

```
func CallCustomCodec(codec Codec) CallOption
```

CallCustomCodec 功能和ForceCodec一样,只是接受参数是grpc.Codec 而不是 encoding.Codec.

Deprecated: 推荐使用 ForceCodec.

### func [FailFast](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L266) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#FailFast)

```
func FailFast(failFast bool) CallOption
```

FailFast 和 WaitForReady相反，失败后立即返回.

Deprecated: use WaitForReady.

### func [ForceCodec](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L408) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ForceCodec)

```
func ForceCodec(codec encoding.Codec) CallOption
```

ForceCodec 返回一个CallOption 为所有的请求和结果设置给定的Codec

See Content-Type on https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md#requests for more details. Also see the documentation on RegisterCodec and CallContentSubtype for more details on the interaction between Codec and content-subtype.

This function is provided for advanced users; prefer to use only CallContentSubtype to select a registered codec instead.

This is an EXPERIMENTAL API.

### func [Header](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L195) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#Header)

```
func Header(md *metadata.MD) CallOption
```

Header 返回一个CallOptions 检测每一个rpc的head.

### func [MaxCallRecvMsgSize](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L285) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#MaxCallRecvMsgSize)

```
func MaxCallRecvMsgSize(bytes int) CallOption
```

MaxCallRecvMsgSize 返回一个CallOption 设置client允许接受消息的最大字节数.

### func [MaxCallSendMsgSize](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L304) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#MaxCallSendMsgSize)

```
func MaxCallSendMsgSize(bytes int) CallOption
```

MaxCallSendMsgSize 返回一个CallOption 设置client可以发送消息的最大字节数.

### func [MaxRetryRPCBufferSize](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L452) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#MaxRetryRPCBufferSize)

```
func MaxRetryRPCBufferSize(bytes int) CallOption
```

MaxRetryRPCBufferSize 返回一个CallOption 设置缓存重试rpc请求的最大内存。

This API is EXPERIMENTAL.

### func [Peer](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L231) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#Peer)

```
func Peer(p *peer.Peer) CallOption
```

Peer 返回一个CallOption 检测一个rpc请求的用户信息，peer字段在rpc完成后被设置。

### func [PerRPCCredentials](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L323) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#PerRPCCredentials)

```
func PerRPCCredentials(creds credentials.PerRPCCredentials) CallOption
```

PerRPCCredentials 返回一个 CallOption 为一个请求设置 credentials.PerRPCCredentials。

### func [Trailer](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L213) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#Trailer)

```
func Trailer(md *metadata.MD) CallOption
```

Trailer 返回一个 CallOption 检测请求的尾部信息.

### func [UseCompressor](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L345) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#UseCompressor)

```
func UseCompressor(name string) CallOption
```

UseCompressor 返回一个 CallOption 设置请求的压缩方式。如果 WithCompressor 也被设置, UseCompressor 拥有更高的优先级。

This API is EXPERIMENTAL.

### func [WaitForReady](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L259) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WaitForReady)

```
func WaitForReady(waitForReady bool) CallOption
```

WaitForReady 当连接中断或连接不上服务器的时候，设置是等待连接还是立即返回失败，默认是立即返回失败。详情可参考 https://github.com/grpc/grpc/blob/master/doc/wait-for-ready.md.

### type [ClientConn](https://github.com/grpc/grpc-go/blob/v1.30.0/clientconn.go#L474) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ClientConn)

```
type ClientConn struct {
	// contains filtered or unexported fields
}
```

ClientConn 表示到概念端点的虚拟连接，以执行RPCs。

ClientConn可以根据配置，负载等情况自由地与端点建立零个或多个实际连接。它还可以自由确定要使用的实际端点，并且可以在每个RPC中进行更改，从而允许客户端负载平衡。

ClientConn封装了一系列功能，包括名称解析，TCP连接建立（带有重试和退避）和TLS握手。 它还通过重新解析名称并重新连接来处理已建立连接上的错误。

### func [Dial](https://github.com/grpc/grpc-go/blob/v1.30.0/clientconn.go#L103) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#Dial)

```
func Dial(target string, opts ...DialOption) (*ClientConn, error)
```

Dial 创建到给定目标的客户端连接。

### func [DialContext](https://github.com/grpc/grpc-go/blob/v1.30.0/clientconn.go#L123) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#DialContext)

```
func DialContext(ctx context.Context, target string, opts ...DialOption) (conn *ClientConn, err error)
```

DialContext创建到给定目标的客户端连接。 默认情况下，它是非阻塞拨号（该功能不会等待建立连接，并且连接发生在后台）。 要使其成为阻止拨号，请使用WithBlock() 拨号选项。

在非阻塞情况下，ctx不会对连接起作用。 它仅控制设置步骤。

在阻塞情况下，ctx可用于取消或终止挂起的连接。 该函数返回后，ctx的取消和到期将不再起作用。 用户应调用ClientConn.Close终止此函数返回后的所有挂起操作。

详情参考 https://github.com/grpc/grpc/blob/master/doc/naming.md. e.g. to use dns resolver, a "dns:///" prefix should be applied to the target.

### func (*ClientConn) [Close](https://github.com/grpc/grpc-go/blob/v1.30.0/clientconn.go#L982) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ClientConn.Close)

```
func (cc *ClientConn) Close() error
```

关闭将ClientConn和所有基础连接关闭。

### func (*ClientConn) [GetMethodConfig](https://github.com/grpc/grpc-go/blob/v1.30.0/clientconn.go#L867) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ClientConn.GetMethodConfig)

```
func (cc *ClientConn) GetMethodConfig(method string) MethodConfig
```

GetMethodConfig 获取输入参数的方法配置。 如果输入参数完全匹配（即 /service/method), 我们将返回相应的MethodConfig。 如果输入参数不完全匹配，将会在服务（即/service/)下查找默认配置。 如果该服务有默认的MethodConfig，将其返回。 否则，返回一个空的MethodConfig。

### func (*ClientConn) [GetState](https://github.com/grpc/grpc-go/blob/v1.30.0/clientconn.go#L524) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ClientConn.GetState)

```
func (cc *ClientConn) GetState() connectivity.State
```

GetState 返回ClientConn的 connectivity.State. This is an EXPERIMENTAL API.

### func (*ClientConn) [Invoke](https://github.com/grpc/grpc-go/blob/v1.30.0/call.go#L29) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ClientConn.Invoke)

```
func (cc *ClientConn) Invoke(ctx context.Context, method string, args, reply interface{}, opts ...CallOption) error
```

Invoke 发送一个RPC请求。Invoke返回的所有错误都来自status包。

### func (*ClientConn) [NewStream](https://github.com/grpc/grpc-go/blob/v1.30.0/stream.go#L141) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ClientConn.NewStream)

```
func (cc *ClientConn) NewStream(ctx context.Context, desc *StreamDesc, method string, opts ...CallOption) (ClientStream, error)
```

NewStream为客户端创建一个新的Stream。 这通常由生成的代码调用。 ctx用于流的生存期。

为了不发生内存泄漏，下面几项至少有一项要执行：

```go
1. Call Close on the ClientConn.
2. Cancel the context provided.
3. Call RecvMsg until a non-nil error is returned. A protobuf-generated
   client-streaming RPC, for instance, might use the helper function
   CloseAndRecv (note that CloseSend does not Recv, therefore is not
   guaranteed to release all resources).
4. Receive a non-nil, non-io.EOF error from Header or SendMsg.
```

### func (*ClientConn) [ResetConnectBackoff](https://github.com/grpc/grpc-go/blob/v1.30.0/clientconn.go#L972) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ClientConn.ResetConnectBackoff)

```
func (cc *ClientConn) ResetConnectBackoff()
```

ResetConnectBackoff在瞬态故障中唤醒所有子通道，并使它们立即尝试另一个连接。 无论当前状态如何，它都将重置用于后续尝试的退避时间。

通常，不应使用此功能。 默认情况下，典型的服务或网络中断会导致合理的客户端重新连接策略。 但是，如果以前不可用的网络可用，则可以使用它来触发立即重新连接。

This API is EXPERIMENTAL.

### func (*ClientConn) [Target](https://github.com/grpc/grpc-go/blob/v1.30.0/clientconn.go#L774) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ClientConn.Target)

```
func (cc *ClientConn) Target() string
```

Target返回ClientConn的连接的地址。 这是一个实验性API。

### func (*ClientConn) [WaitForStateChange](https://github.com/grpc/grpc-go/blob/v1.30.0/clientconn.go#L509) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ClientConn.WaitForStateChange)

```
func (cc *ClientConn) WaitForStateChange(ctx context.Context, sourceState connectivity.State) bool
```

WaitForStateChange等待，直到从源状态更改ClientConn的connectivity.State。 在前一种情况下返回true值，在后一种情况下返回false。 这是一个实验性API。

### type [ClientConnInterface](https://github.com/grpc/grpc-go/blob/v1.30.0/clientconn.go#L451) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ClientConnInterface)

```
type ClientConnInterface interface {
	// Invoke performs a unary RPC and returns after the response is received
	// into reply.
	Invoke(ctx context.Context, method string, args interface{}, reply interface{}, opts ...CallOption) error
	// NewStream begins a streaming RPC.
	NewStream(ctx context.Context, desc *StreamDesc, method string, opts ...CallOption) (ClientStream, error)
}
```

ClientConnInterface定义客户端执行普通和流式RPC所需的功能。 它由*ClientConn实现，仅应由生成的代码引用。

### type [ClientStream](https://github.com/grpc/grpc-go/blob/v1.30.0/stream.go#L78) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ClientStream)

```
type ClientStream interface {
	// Header returns the header metadata received from the server if there
	// is any. It blocks if the metadata is not ready to read.
	Header() (metadata.MD, error)
	// Trailer returns the trailer metadata from the server, if there is any.
	// It must only be called after stream.CloseAndRecv has returned, or
	// stream.Recv has returned a non-nil error (including io.EOF).
	Trailer() metadata.MD
	// CloseSend closes the send direction of the stream. It closes the stream
	// when non-nil error is met. It is also not safe to call CloseSend
	// concurrently with SendMsg.
	CloseSend() error
	// Context returns the context for this stream.
	//
	// It should not be called until after Header or RecvMsg has returned. Once
	// called, subsequent client-side retries are disabled.
	Context() context.Context
	// SendMsg is generally called by generated code. On error, SendMsg aborts
	// the stream. If the error was generated by the client, the status is
	// returned directly; otherwise, io.EOF is returned and the status of
	// the stream may be discovered using RecvMsg.
	//
	// SendMsg blocks until:
	//   - There is sufficient flow control to schedule m with the transport, or
	//   - The stream is done, or
	//   - The stream breaks.
	//
	// SendMsg does not wait until the message is received by the server. An
	// untimely stream closure may result in lost messages. To ensure delivery,
	// users should ensure the RPC completed successfully using RecvMsg.
	//
	// It is safe to have a goroutine calling SendMsg and another goroutine
	// calling RecvMsg on the same stream at the same time, but it is not safe
	// to call SendMsg on the same stream in different goroutines. It is also
	// not safe to call CloseSend concurrently with SendMsg.
	SendMsg(m interface{}) error
	// RecvMsg blocks until it receives a message into m or the stream is
	// done. It returns io.EOF when the stream completes successfully. On
	// any other error, the stream is aborted and the error contains the RPC
	// status.
	//
	// It is safe to have a goroutine calling SendMsg and another goroutine
	// calling RecvMsg on the same stream at the same time, but it is not
	// safe to call RecvMsg on the same stream in different goroutines.
	RecvMsg(m interface{}) error
}
```

ClientStream 定义流的RPC客户端行为，从ClientStream方法返回的所有错误均与状态包兼容。

### func [NewClientStream](https://github.com/grpc/grpc-go/blob/v1.30.0/stream.go#L153) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#NewClientStream)

```
func NewClientStream(ctx context.Context, desc *StreamDesc, cc *ClientConn, method string, opts ...CallOption) (ClientStream, error)
```

NewClientStream是ClientConn.NewStream的包装。

### type [Codec](https://github.com/grpc/grpc-go/blob/v1.30.0/codec.go#L42) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#Codec)

```
type Codec interface {
	// Marshal returns the wire format of v.
	Marshal(v interface{}) ([]byte, error)
	// Unmarshal parses the wire format into v.
	Unmarshal(data []byte, v interface{}) error
	// String returns the name of the Codec implementation.  This is unused by
	// gRPC.
	String() string
}
```

编解码器定义gRPC用于编码和解码消息的接口。 请注意，此接口的实现必须是线程安全的。 可以从并发goroutine中调用编解码器的方法。

不推荐使用：改用encoding.Codec。

### type [Compressor](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L49) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#Compressor)

```
type Compressor interface {
	// Do compresses p into w.
	Do(w io.Writer, p []byte) error
	// Type returns the compression algorithm the Compressor uses.
	Type() string
}
```

Compressor定义了gRPC用来压缩消息的接口。

不推荐使用：改用encoding编码。

### func [NewGZIPCompressor](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L63) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#NewGZIPCompressor)

```
func NewGZIPCompressor() Compressor
```

NewGZIPCompressor 创建基于 GZIP 的 Compressor。

不推荐使用：改用encoding/gzip.

### func [NewGZIPCompressorWithLevel](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L74) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#NewGZIPCompressorWithLevel)

```
func NewGZIPCompressorWithLevel(level int) (Compressor, error)
```

NewGZIPCompressorWithLevel类似于NewGZIPCompressor，但指定了gzip压缩级别，而不是假定DefaultCompression。如果级别有效，则返回的错误将为nil。

不推荐使用：改用 encoding/gzip.

### type [CompressorCallOption](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L351) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#CompressorCallOption)

```
type CompressorCallOption struct {
	CompressorType string
}
```

CompressorCallOption是一个CallOption，用于指示要使用的压缩器。 这是一个实验性API。

### type [ConnectParams](https://github.com/grpc/grpc-go/blob/v1.30.0/backoff.go#L52) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ConnectParams)

```
type ConnectParams struct {
	// Backoff specifies the configuration options for connection backoff.
	Backoff backoff.Config
	// MinConnectTimeout is the minimum amount of time we are willing to give a
	// connection to complete.
	MinConnectTimeout time.Duration
}
```

ConnectParams定义用于连接和重试的参数。 鼓励用户使用它而不是上面定义的BackoffConfig类型。 See here for more details: https://github.com/grpc/grpc/blob/master/doc/connection-backoff.md.

This API is EXPERIMENTAL.

### type [ContentSubtypeCallOption](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L384) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ContentSubtypeCallOption)

```
type ContentSubtypeCallOption struct {
	ContentSubtype string
}
```

ContentSubtypeCallOption是一个CallOption，它指定消息的content-subtype。 这是一个实验性API。

### type [CustomCodecCallOption](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L438) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#CustomCodecCallOption)

```
type CustomCodecCallOption struct {
	Codec Codec
}
```

CustomCodecCallOption是一个CallOption，它指示用于编组消息的编解码器。这是一个实验性API。

### type [Decompressor](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L108) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#Decompressor)

```
type Decompressor interface {
	// Do reads the data from r and uncompress them.
	Do(r io.Reader) ([]byte, error)
	// Type returns the compression algorithm the Decompressor uses.
	Type() string
}
```

Decompressor 定义gRPC用于解压缩消息的接口。

不推荐使用：使用包编码。

### func [NewGZIPDecompressor](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L122) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#NewGZIPDecompressor)

```
func NewGZIPDecompressor() Decompressor
```

NewGZIPDecompressor基于GZIP创建一个解压缩器。

不推荐使用：使用包 encoding/gzip.

### type [DialOption](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L79) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#DialOption)

```
type DialOption interface {
	// contains filtered or unexported methods
}
```

DialOption配置我们如何建立连接。

### func [FailOnNonTempDialError](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L408) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#FailOnNonTempDialError)

```
func FailOnNonTempDialError(f bool) DialOption
```

FailOnNonTempDialError返回一个DialOption，它指定gRPC是否在非临时拨号错误时失败。 如果f为true，并且Dialer返回非临时错误，则gRPC将失败与网络地址的连接，并且不会尝试重新连接。 FailOnNonTempDialError的默认值为false。

FailOnNonTempDialError仅会影响初始拨号，除非您也使用WithBlock()，否则它不会做任何有用的事情。

这是一个实验性API。

### func [WithAuthority](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L475) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithAuthority)

```
func WithAuthority(a string) DialOption
```

WithAuthority返回一个DialOption，该DialOption指定将用作：authority伪标题的值。 该值仅与WithInsecure一起使用，如果存在TransportCredentials，则该值无效。

### func [WithBackoffConfig](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L264) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithBackoffConfig)

```
func WithBackoffConfig(b BackoffConfig) DialOption
```

WithBackoffConfig将拨号程序配置为在连接失败后使用提供的参数。不推荐使用：改用WithConnectParams。 整个1.x将受支持。

### func [WithBackoffMaxDelay](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L256) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithBackoffMaxDelay)

```
func WithBackoffMaxDelay(md time.Duration) DialOption
```

WithBackoffMaxDelay configures the dialer to use the provided maximum delay when backing off after failed connection attempts.

Deprecated: use WithConnectParams instead. Will be supported throughout 1.x.

### func [WithBalancerName](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L211) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithBalancerName)

```
func WithBalancerName(balancerName string) DialOption
```

WithBalancerName设置将初始化ClientConn的负载均衡。 将使用在balancerName中注册的Balancer。 如果balancerName未注册任何balancer，则此函数会报错。

负载均衡不能被服务配置指定的平衡器选项覆盖。

弃用：改用WithDefaultServiceConfig和WithDisableServiceConfig。 在将来的1.x版本中将被删除。

### func [WithBlock](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L283) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithBlock)

```
func WithBlock() DialOption
```

WithBlock返回一个DialOption，它使Dial的调用方阻塞，直到基础连接建立为止。 否则，Dial将立即返回，并且服务器连接在后台发生。

### func [WithChainStreamInterceptor](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L466) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithChainStreamInterceptor)

```
func WithChainStreamInterceptor(interceptors ...StreamClientInterceptor) DialOption
```

WithChainStreamInterceptor返回一个DialOption，该DialOption指定用于流式RPC的链接拦截器。 第一个拦截器将是最外面的，而最后一个拦截器将是真实调用的最里面的包装器。 通过此方法添加的所有拦截器都将被链接，并且由WithStreamInterceptor定义的拦截器将始终位于链的前面。

### func [WithChainUnaryInterceptor](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L447) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithChainUnaryInterceptor)

```
func WithChainUnaryInterceptor(interceptors ...UnaryClientInterceptor) DialOption
```

WithChainUnaryInterceptor返回一个DialOption，该DialOption指定一元RPC的链接拦截器。 第一个拦截器将是最外面的，而最后一个拦截器将是真实调用的最里面的包装器。 此方法添加的所有拦截器都将被链接起来，并且由WithUnaryInterceptor定义的拦截器将始终位于链的前面。

### func [WithChannelzParentID](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L486) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithChannelzParentID)

```
func WithChannelzParentID(id int64) DialOption
```

WithChannelzParentID返回一个DialOption，该DialOption指定当前ClientConn父级的channelz ID。 此功能用于创建嵌套通道(例如grpclb dial), 此API是实验性的。

### func [WithCodec](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L171) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithCodec)

```
func WithCodec(c Codec) DialOption
```

WithCodec returns a DialOption which sets a codec for message marshaling and unmarshaling.

Deprecated: use WithDefaultCallOptions(ForceCodec(_)) instead. Will be supported throughout 1.x.

### func [WithCompressor](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L180) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithCompressor)

```
func WithCompressor(cp Compressor) DialOption
```

WithCompressor returns a DialOption which sets a Compressor to use for message compression. It has lower priority than the compressor set by the UseCompressor CallOption.

Deprecated: use UseCompressor instead. Will be supported throughout 1.x.

### func [WithConnectParams](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L243) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithConnectParams)

```
func WithConnectParams(p ConnectParams) DialOption
```

WithConnectParams configures the dialer to use the provided ConnectParams.

The backoff configuration specified as part of the ConnectParams overrides all defaults specified in https://github.com/grpc/grpc/blob/master/doc/connection-backoff.md. Consider using the backoff.DefaultConfig as a base, in cases where you want to override only a subset of the backoff configuration.

This API is EXPERIMENTAL.

### func [WithContextDialer](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L364) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithContextDialer)

```
func WithContextDialer(f func(context.Context, string) (net.Conn, error)) DialOption
```

WithContextDialer returns a DialOption that sets a dialer to create connections. If FailOnNonTempDialError() is set to true, and an error is returned by f, gRPC checks the error's Temporary() method to decide if it should try to reconnect to the network address.

### func [WithCredentialsBundle](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L343) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithCredentialsBundle)

```
func WithCredentialsBundle(b credentials.Bundle) DialOption
```

WithCredentialsBundle returns a DialOption to set a credentials bundle for the ClientConn.WithCreds. This should not be used together with WithTransportCredentials.

This API is experimental.

### func [WithDecompressor](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L196) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithDecompressor)

```
func WithDecompressor(dc Decompressor) DialOption
```

WithDecompressor returns a DialOption which sets a Decompressor to use for incoming message decompression. If incoming response messages are encoded using the decompressor's Type(), it will be used. Otherwise, the message encoding will be used to look up the compressor registered via encoding.RegisterCompressor, which will then be used to decompress the message. If no compressor is registered for the encoding, an Unimplemented status error will be returned.

Deprecated: use encoding.RegisterCompressor instead. Will be supported throughout 1.x.

### func [WithDefaultCallOptions](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L160) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithDefaultCallOptions)

```
func WithDefaultCallOptions(cos ...CallOption) DialOption
```

WithDefaultCallOptions returns a DialOption which sets the default CallOptions for calls over the connection.

### func [WithDefaultServiceConfig](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L512) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithDefaultServiceConfig)

```
func WithDefaultServiceConfig(s string) DialOption
```

WithDefaultServiceConfig returns a DialOption that configures the default service config, which will be used in cases where:

\1. WithDisableServiceConfig is also used. 2. Resolver does not return a service config or if the resolver returns an

```
invalid service config.
```

This API is EXPERIMENTAL.

### func [WithDialer](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L381) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithDialer)

```
func WithDialer(f func(string, time.Duration) (net.Conn, error)) DialOption
```

WithDialer returns a DialOption that specifies a function to use for dialing network addresses. If FailOnNonTempDialError() is set to true, and an error is returned by f, gRPC checks the error's Temporary() method to decide if it should try to reconnect to the network address.

Deprecated: use WithContextDialer instead. Will be supported throughout 1.x.

### func [WithDisableHealthCheck](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L546) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithDisableHealthCheck)

```
func WithDisableHealthCheck() DialOption
```

WithDisableHealthCheck disables the LB channel health checking for all SubConns of this ClientConn.

This API is EXPERIMENTAL.

### func [WithDisableRetry](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L528) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithDisableRetry)

```
func WithDisableRetry() DialOption
```

WithDisableRetry returns a DialOption that disables retries, even if the service config enables them. This does not impact transparent retries, which will happen automatically if no data is written to the wire or if the RPC is unprocessed by the remote server.

Retry support is currently disabled by default, but will be enabled by default in the future. Until then, it may be enabled by setting the environment variable "GRPC_GO_RETRY" to "on".

This API is EXPERIMENTAL.

### func [WithDisableServiceConfig](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L498) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithDisableServiceConfig)

```
func WithDisableServiceConfig() DialOption
```

WithDisableServiceConfig returns a DialOption that causes gRPC to ignore any service config provided by the resolver and provides a hint to the resolver to not fetch service configs.

Note that this dial option only disables service config from resolver. If default service config is provided, gRPC will use the default service config.

### func [WithInitialConnWindowSize](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L143) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithInitialConnWindowSize)

```
func WithInitialConnWindowSize(s int32) DialOption
```

WithInitialConnWindowSize returns a DialOption which sets the value for initial window size on a connection. The lower bound for window size is 64K and any value smaller than that will be ignored.

### func [WithInitialWindowSize](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L134) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithInitialWindowSize)

```
func WithInitialWindowSize(s int32) DialOption
```

WithInitialWindowSize returns a DialOption which sets the value for initial window size on a stream. The lower bound for window size is 64K and any value smaller than that will be ignored.

### func [WithInsecure](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L305) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithInsecure)

```
func WithInsecure() DialOption
```

WithInsecure returns a DialOption which disables transport security for this ClientConn. Note that transport security is required unless WithInsecure is set.

### func [WithKeepaliveParams](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L424) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithKeepaliveParams)

```
func WithKeepaliveParams(kp keepalive.ClientParameters) DialOption
```

WithKeepaliveParams returns a DialOption that specifies keepalive parameters for the client transport.

### func [WithMaxHeaderListSize](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L536) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithMaxHeaderListSize)

```
func WithMaxHeaderListSize(s uint32) DialOption
```

WithMaxHeaderListSize returns a DialOption that specifies the maximum (uncompressed) size of header list that the client is prepared to accept.

### func [WithMaxMsgSize](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L154) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithMaxMsgSize)

```
func WithMaxMsgSize(s int) DialOption
```

WithMaxMsgSize returns a DialOption which sets the maximum message size the client can receive.

Deprecated: use WithDefaultCallOptions(MaxCallRecvMsgSize(s)) instead. Will be supported throughout 1.x.

### func [WithNoProxy](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L315) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithNoProxy)

```
func WithNoProxy() DialOption
```

WithNoProxy returns a DialOption which disables the use of proxies for this ClientConn. This is ignored if WithDialer or WithContextDialer are used.

This API is EXPERIMENTAL.

### func [WithPerRPCCredentials](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L332) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithPerRPCCredentials)

```
func WithPerRPCCredentials(creds credentials.PerRPCCredentials) DialOption
```

WithPerRPCCredentials returns a DialOption which sets credentials and places auth state on each outbound RPC.

### func [WithReadBufferSize](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L125) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithReadBufferSize)

```
func WithReadBufferSize(s int) DialOption
```

WithReadBufferSize lets you set the size of read buffer, this determines how much data can be read at most for each read syscall.

The default value for this buffer is 32KB. Zero will disable read buffer for a connection so data framer can access the underlying conn directly.

### func [WithResolvers](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L602) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithResolvers)

```
func WithResolvers(rs ...resolver.Builder) DialOption
```

WithResolvers allows a list of resolver implementations to be registered locally with the ClientConn without needing to be globally registered via resolver.Register. They will be matched against the scheme used for the current Dial only, and will take precedence over the global registry.

This API is EXPERIMENTAL.

### func [WithReturnConnectionError](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L295) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithReturnConnectionError)

```
func WithReturnConnectionError() DialOption
```

WithReturnConnectionError returns a DialOption which makes the client connection return a string containing both the last connection error that occurred and the context.DeadlineExceeded error. Implies WithBlock()

This API is EXPERIMENTAL.

### func [WithServiceConfig](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L228) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithServiceConfig)

```
func WithServiceConfig(c <-chan ServiceConfig) DialOption
```

WithServiceConfig returns a DialOption which has a channel to read the service configuration.

Deprecated: service config should be received through name resolver or via WithDefaultServiceConfig, as specified at https://github.com/grpc/grpc/blob/master/doc/service_config.md. Will be removed in a future 1.x release.

### func [WithStatsHandler](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L393) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithStatsHandler)

```
func WithStatsHandler(h stats.Handler) DialOption
```

WithStatsHandler returns a DialOption that specifies the stats handler for all the RPCs and underlying network connections in this ClientConn.

### func [WithStreamInterceptor](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L455) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithStreamInterceptor)

```
func WithStreamInterceptor(f StreamClientInterceptor) DialOption
```

WithStreamInterceptor returns a DialOption that specifies the interceptor for streaming RPCs.

### func [WithTimeout](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L354) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithTimeout)

```
func WithTimeout(d time.Duration) DialOption
```

WithTimeout returns a DialOption that configures a timeout for dialing a ClientConn initially. This is valid if and only if WithBlock() is present.

Deprecated: use DialContext instead of Dial and context.WithTimeout instead. Will be supported throughout 1.x.

### func [WithTransportCredentials](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L324) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithTransportCredentials)

```
func WithTransportCredentials(creds credentials.TransportCredentials) DialOption
```

WithTransportCredentials returns a DialOption which configures a connection level security credentials (e.g., TLS/SSL). This should not be used together with WithCredentialsBundle.

### func [WithUnaryInterceptor](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L436) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithUnaryInterceptor)

```
func WithUnaryInterceptor(f UnaryClientInterceptor) DialOption
```

WithUnaryInterceptor returns a DialOption that specifies the interceptor for unary RPCs.

### func [WithUserAgent](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L416) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithUserAgent)

```
func WithUserAgent(s string) DialOption
```

WithUserAgent returns a DialOption that specifies a user agent string for all the RPCs.

### func [WithWriteBufferSize](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L114) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WithWriteBufferSize)

```
func WithWriteBufferSize(s int) DialOption
```

WithWriteBufferSize determines how much data can be batched before doing a write on the wire. The corresponding memory allocation for this buffer will be twice the size to keep syscalls low. The default value for this buffer is 32KB.

Zero will disable the write buffer such that each write will be on underlying connection. Note: A Send call may not directly translate to a write.

### type [EmptyCallOption](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L188) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#EmptyCallOption)

```
type EmptyCallOption struct{}
```

EmptyCallOption does not alter the Call configuration. It can be embedded in another structure to carry satellite data for use by interceptors.

### type [EmptyDialOption](https://github.com/grpc/grpc-go/blob/v1.30.0/dialoptions.go#L87) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#EmptyDialOption)

```
type EmptyDialOption struct{}
```

EmptyDialOption does not alter the dial configuration. It can be embedded in another structure to build custom dial options.

This API is EXPERIMENTAL.

### type [EmptyServerOption](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L165) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#EmptyServerOption)

```
type EmptyServerOption struct{}
```

EmptyServerOption does not alter the server configuration. It can be embedded in another structure to build custom server options.

This API is EXPERIMENTAL.

### type [FailFastCallOption](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L273) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#FailFastCallOption)

```
type FailFastCallOption struct {
	FailFast bool
}
```

FailFastCallOption is a CallOption for indicating whether an RPC should fail fast or not. This is an EXPERIMENTAL API.

### type [ForceCodecCallOption](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L416) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ForceCodecCallOption)

```
type ForceCodecCallOption struct {
	Codec encoding.Codec
}
```

ForceCodecCallOption is a CallOption that indicates the codec used for marshaling messages.

This is an EXPERIMENTAL API.

### type [HeaderCallOption](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L202) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#HeaderCallOption)

```
type HeaderCallOption struct {
	HeaderAddr *metadata.MD
}
```

HeaderCallOption is a CallOption for collecting response header metadata. The metadata field will be populated *after* the RPC completes. This is an EXPERIMENTAL API.

### type [MaxRecvMsgSizeCallOption](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L292) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#MaxRecvMsgSizeCallOption)

```
type MaxRecvMsgSizeCallOption struct {
	MaxRecvMsgSize int
}
```

MaxRecvMsgSizeCallOption is a CallOption that indicates the maximum message size in bytes the client can receive. This is an EXPERIMENTAL API.

### type [MaxRetryRPCBufferSizeCallOption](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L459) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#MaxRetryRPCBufferSizeCallOption)

```
type MaxRetryRPCBufferSizeCallOption struct {
	MaxRetryRPCBufferSize int
}
```

MaxRetryRPCBufferSizeCallOption is a CallOption indicating the amount of memory to be used for caching this RPC for retry purposes. This is an EXPERIMENTAL API.

### type [MaxSendMsgSizeCallOption](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L311) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#MaxSendMsgSizeCallOption)

```
type MaxSendMsgSizeCallOption struct {
	MaxSendMsgSize int
}
```

MaxSendMsgSizeCallOption is a CallOption that indicates the maximum message size in bytes the client can send. This is an EXPERIMENTAL API.

### type [MethodConfig](https://github.com/grpc/grpc-go/blob/v1.30.0/service_config.go#L44) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#MethodConfig)

```
type MethodConfig struct {
	// WaitForReady indicates whether RPCs sent to this method should wait until
	// the connection is ready by default (!failfast). The value specified via the
	// gRPC client API will override the value set here.
	WaitForReady *bool
	// Timeout is the default timeout for RPCs sent to this method. The actual
	// deadline used will be the minimum of the value specified here and the value
	// set by the application via the gRPC client API.  If either one is not set,
	// then the other will be used.  If neither is set, then the RPC has no deadline.
	Timeout *time.Duration
	// MaxReqSize is the maximum allowed payload size for an individual request in a
	// stream (client->server) in bytes. The size which is measured is the serialized
	// payload after per-message compression (but before stream compression) in bytes.
	// The actual value used is the minimum of the value specified here and the value set
	// by the application via the gRPC client API. If either one is not set, then the other
	// will be used.  If neither is set, then the built-in default is used.
	MaxReqSize *int
	// MaxRespSize is the maximum allowed payload size for an individual response in a
	// stream (server->client) in bytes.
	MaxRespSize *int
	// contains filtered or unexported fields
}
```

MethodConfig defines the configuration recommended by the service providers for a particular method.

Deprecated: Users should not use this struct. Service config should be received through name resolver, as specified here https://github.com/grpc/grpc/blob/master/doc/service_config.md

### type [MethodDesc](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L66) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#MethodDesc)

```
type MethodDesc struct {
	MethodName string
	Handler    methodHandler
}
```

MethodDesc represents an RPC service's method specification.

### type [MethodInfo](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L573) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#MethodInfo)

```
type MethodInfo struct {
	// Name is the method name only, without the service name or package name.
	Name string
	// IsClientStream indicates whether the RPC is a client streaming RPC.
	IsClientStream bool
	// IsServerStream indicates whether the RPC is a server streaming RPC.
	IsServerStream bool
}
```

MethodInfo contains the information of an RPC including its method name and type.

### type [PeerCallOption](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L238) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#PeerCallOption)

```
type PeerCallOption struct {
	PeerAddr *peer.Peer
}
```

PeerCallOption is a CallOption for collecting the identity of the remote peer. The peer field will be populated *after* the RPC completes. This is an EXPERIMENTAL API.

### type [PerRPCCredsCallOption](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L330) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#PerRPCCredsCallOption)

```
type PerRPCCredsCallOption struct {
	Creds credentials.PerRPCCredentials
}
```

PerRPCCredsCallOption is a CallOption that indicates the per-RPC credentials to use for the call. This is an EXPERIMENTAL API.

### type [PreparedMsg](https://github.com/grpc/grpc-go/blob/v1.30.0/preloader.go#L29) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#PreparedMsg)

```
type PreparedMsg struct {
	// contains filtered or unexported fields
}
```

PreparedMsg is responsible for creating a Marshalled and Compressed object.

This API is EXPERIMENTAL.

### func (*PreparedMsg) [Encode](https://github.com/grpc/grpc-go/blob/v1.30.0/preloader.go#L37) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#PreparedMsg.Encode)

```
func (p *PreparedMsg) Encode(s Stream, msg interface{}) error
```

Encode marshalls and compresses the message using the codec and compressor for the stream.

### type [Server](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L98) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#Server)

```
type Server struct {
	// contains filtered or unexported fields
}
```

Server is a gRPC server to serve RPC requests.

### func [NewServer](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L485) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#NewServer)

```
func NewServer(opt ...ServerOption) *Server
```

NewServer creates a gRPC server which has no service registered and has not started to accept requests yet.

### func (*Server) [GetServiceInfo](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L591) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#Server.GetServiceInfo)

```
func (s *Server) GetServiceInfo() map[string]ServiceInfo
```

GetServiceInfo returns a map from service names to ServiceInfo. Service names include the package names, in the form of <package>.<service>.

### func (*Server) [GracefulStop](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L1614) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#Server.GracefulStop)

```
func (s *Server) GracefulStop()
```

GracefulStop stops the gRPC server gracefully. It stops the server from accepting new connections and RPCs and blocks until all the pending RPCs are finished.

### func (*Server) [RegisterService](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L536) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#Server.RegisterService)

```
func (s *Server) RegisterService(sd *ServiceDesc, ss interface{})
```

RegisterService registers a service and its implementation to the gRPC server. It is called from the IDL generated code. This must be called before invoking Serve.

### func (*Server) [Serve](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L655) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#Server.Serve)

```
func (s *Server) Serve(lis net.Listener) error
```

Serve accepts incoming connections on the listener lis, creating a new ServerTransport and service goroutine for each. The service goroutines read gRPC requests and then call the registered handlers to reply to them. Serve returns when lis.Accept fails with fatal errors. lis will be closed when this method returns. Serve will return a non-nil error unless Stop or GracefulStop is called.

### func (*Server) [ServeHTTP](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L873) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#Server.ServeHTTP)

```
func (s *Server) ServeHTTP(w http.ResponseWriter, r *http.Request)
```

ServeHTTP implements the Go standard library's http.Handler interface by responding to the gRPC request r, by looking up the requested gRPC method in the gRPC server s.

The provided HTTP request must have arrived on an HTTP/2 connection. When using the Go standard library's server, practically this means that the Request must also have arrived over TLS.

To share one port (such as 443 for https) between gRPC and an existing http.Handler, use a root http.Handler such as:

```
if r.ProtoMajor == 2 && strings.HasPrefix(
	r.Header.Get("Content-Type"), "application/grpc") {
	grpcServer.ServeHTTP(w, r)
} else {
	yourMux.ServeHTTP(w, r)
}
```

Note that ServeHTTP uses Go's HTTP/2 server implementation which is totally separate from grpc-go's HTTP/2 server. Performance and features may vary between the two paths. ServeHTTP does not support some gRPC features available through grpc-go's HTTP/2 server, and it is currently EXPERIMENTAL and subject to change.

### func (*Server) [Stop](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L1570) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#Server.Stop)

```
func (s *Server) Stop()
```

Stop stops the gRPC server. It immediately closes all open connections and listeners. It cancels all active RPCs on the server side and the corresponding pending RPCs on the client side will get notified by connection errors.

### type [ServerOption](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L157) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ServerOption)

```
type ServerOption interface {
	// contains filtered or unexported methods
}
```

A ServerOption sets options such as credentials, codec and keepalive parameters, etc.

### func [ChainStreamInterceptor](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L351) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ChainStreamInterceptor)

```
func ChainStreamInterceptor(interceptors ...StreamServerInterceptor) ServerOption
```

ChainStreamInterceptor returns a ServerOption that specifies the chained interceptor for streaming RPCs. The first interceptor will be the outer most, while the last interceptor will be the inner most wrapper around the real call. All stream interceptors added by this method will be chained.

### func [ChainUnaryInterceptor](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L330) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ChainUnaryInterceptor)

```
func ChainUnaryInterceptor(interceptors ...UnaryServerInterceptor) ServerOption
```

ChainUnaryInterceptor returns a ServerOption that specifies the chained interceptor for unary RPCs. The first interceptor will be the outer most, while the last interceptor will be the inner most wrapper around the real call. All unary interceptors added by this method will be chained.

### func [ConnectionTimeout](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L399) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ConnectionTimeout)

```
func ConnectionTimeout(d time.Duration) ServerOption
```

ConnectionTimeout returns a ServerOption that sets the timeout for connection establishment (up to and including HTTP/2 handshaking) for all new connections. If this is not set, the default is 120 seconds. A zero or negative value will result in an immediate timeout.

This API is EXPERIMENTAL.

### func [Creds](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L308) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#Creds)

```
func Creds(c credentials.TransportCredentials) ServerOption
```

Creds returns a ServerOption that sets credentials for server connections.

### func [CustomCodec](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L245) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#CustomCodec)

```
func CustomCodec(codec Codec) ServerOption
```

CustomCodec returns a ServerOption that sets a codec for message marshaling and unmarshaling.

This will override any lookups by content-subtype for Codecs registered with RegisterCodec.

### func [HeaderTableSize](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L417) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#HeaderTableSize)

```
func HeaderTableSize(s uint32) ServerOption
```

HeaderTableSize returns a ServerOption that sets the size of dynamic header table for stream.

This API is EXPERIMENTAL.

### func [InTapHandle](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L359) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#InTapHandle)

```
func InTapHandle(h tap.ServerInHandle) ServerOption
```

InTapHandle returns a ServerOption that sets the tap handle for all the server transport to be created. Only one can be installed.

### func [InitialConnWindowSize](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L217) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#InitialConnWindowSize)

```
func InitialConnWindowSize(s int32) ServerOption
```

InitialConnWindowSize returns a ServerOption that sets window size for a connection. The lower bound for window size is 64K and any value smaller than that will be ignored.

### func [InitialWindowSize](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L209) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#InitialWindowSize)

```
func InitialWindowSize(s int32) ServerOption
```

InitialWindowSize returns a ServerOption that sets window size for stream. The lower bound for window size is 64K and any value smaller than that will be ignored.

### func [KeepaliveEnforcementPolicy](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L236) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#KeepaliveEnforcementPolicy)

```
func KeepaliveEnforcementPolicy(kep keepalive.EnforcementPolicy) ServerOption
```

KeepaliveEnforcementPolicy returns a ServerOption that sets keepalive enforcement policy for the server.

### func [KeepaliveParams](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L224) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#KeepaliveParams)

```
func KeepaliveParams(kp keepalive.ServerParameters) ServerOption
```

KeepaliveParams returns a ServerOption that sets keepalive and max-age parameters for the server.

### func [MaxConcurrentStreams](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L301) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#MaxConcurrentStreams)

```
func MaxConcurrentStreams(n uint32) ServerOption
```

MaxConcurrentStreams returns a ServerOption that will apply a limit on the number of concurrent streams to each ServerTransport.

### func [MaxHeaderListSize](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L407) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#MaxHeaderListSize)

```
func MaxHeaderListSize(s uint32) ServerOption
```

MaxHeaderListSize returns a ServerOption that sets the max (uncompressed) size of header list that the server is prepared to accept.

### func [MaxMsgSize](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L279) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#MaxMsgSize)

```
func MaxMsgSize(m int) ServerOption
```

MaxMsgSize returns a ServerOption to set the max message size in bytes the server can receive. If this is not set, gRPC uses the default limit.

Deprecated: use MaxRecvMsgSize instead.

### func [MaxRecvMsgSize](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L285) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#MaxRecvMsgSize)

```
func MaxRecvMsgSize(m int) ServerOption
```

MaxRecvMsgSize returns a ServerOption to set the max message size in bytes the server can receive. If this is not set, gRPC uses the default 4MB.

### func [MaxSendMsgSize](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L293) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#MaxSendMsgSize)

```
func MaxSendMsgSize(m int) ServerOption
```

MaxSendMsgSize returns a ServerOption to set the max message size in bytes the server can send. If this is not set, gRPC uses the default `math.MaxInt32`.

### func [NumStreamWorkers](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L429) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#NumStreamWorkers)

```
func NumStreamWorkers(numServerWorkers uint32) ServerOption
```

NumStreamWorkers returns a ServerOption that sets the number of worker goroutines that should be used to process incoming streams. Setting this to zero (default) will disable workers and spawn a new goroutine for each stream.

This API is EXPERIMENTAL.

### func [RPCCompressor](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L258) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#RPCCompressor)

```
func RPCCompressor(cp Compressor) ServerOption
```

RPCCompressor returns a ServerOption that sets a compressor for outbound messages. For backward compatibility, all outbound messages will be sent using this compressor, regardless of incoming message compression. By default, server messages will be sent using the same compressor with which request messages were sent.

Deprecated: use encoding.RegisterCompressor instead.

### func [RPCDecompressor](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L269) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#RPCDecompressor)

```
func RPCDecompressor(dc Decompressor) ServerOption
```

RPCDecompressor returns a ServerOption that sets a decompressor for inbound messages. It has higher priority than decompressors registered via encoding.RegisterCompressor.

Deprecated: use encoding.RegisterCompressor instead.

### func [ReadBufferSize](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L201) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ReadBufferSize)

```
func ReadBufferSize(s int) ServerOption
```

ReadBufferSize lets you set the size of read buffer, this determines how much data can be read at most for one read syscall. The default value for this buffer is 32KB. Zero will disable read buffer for a connection so data framer can access the underlying conn directly.

### func [StatsHandler](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L369) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#StatsHandler)

```
func StatsHandler(h stats.Handler) ServerOption
```

StatsHandler returns a ServerOption that sets the stats handler for the server.

### func [StreamInterceptor](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L338) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#StreamInterceptor)

```
func StreamInterceptor(i StreamServerInterceptor) ServerOption
```

StreamInterceptor returns a ServerOption that sets the StreamServerInterceptor for the server. Only one stream interceptor can be installed.

### func [UnaryInterceptor](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L317) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#UnaryInterceptor)

```
func UnaryInterceptor(i UnaryServerInterceptor) ServerOption
```

UnaryInterceptor returns a ServerOption that sets the UnaryServerInterceptor for the server. Only one unary interceptor can be installed. The construction of multiple interceptors (e.g., chaining) can be implemented at the caller.

### func [UnknownServiceHandler](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L381) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#UnknownServiceHandler)

```
func UnknownServiceHandler(streamHandler StreamHandler) ServerOption
```

UnknownServiceHandler returns a ServerOption that allows for adding a custom unknown service handler. The provided method is a bidi-streaming RPC service handler that will be invoked instead of returning the "unimplemented" gRPC error whenever a request is received for an unregistered service or method. The handling function and stream interceptor (if set) have full access to the ServerStream, including its Context.

### func [WriteBufferSize](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L190) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#WriteBufferSize)

```
func WriteBufferSize(s int) ServerOption
```

WriteBufferSize determines how much data can be batched before doing a write on the wire. The corresponding memory allocation for this buffer will be twice the size to keep syscalls low. The default value for this buffer is 32KB. Zero will disable the write buffer such that each write will be on underlying connection. Note: A Send call may not directly translate to a write.

### type [ServerStream](https://github.com/grpc/grpc-go/blob/v1.30.0/stream.go#L1290) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ServerStream)

```
type ServerStream interface {
	// SetHeader sets the header metadata. It may be called multiple times.
	// When call multiple times, all the provided metadata will be merged.
	// All the metadata will be sent out when one of the following happens:
	//  - ServerStream.SendHeader() is called;
	//  - The first response is sent out;
	//  - An RPC status is sent out (error or success).
	SetHeader(metadata.MD) error
	// SendHeader sends the header metadata.
	// The provided md and headers set by SetHeader() will be sent.
	// It fails if called multiple times.
	SendHeader(metadata.MD) error
	// SetTrailer sets the trailer metadata which will be sent with the RPC status.
	// When called more than once, all the provided metadata will be merged.
	SetTrailer(metadata.MD)
	// Context returns the context for this stream.
	Context() context.Context
	// SendMsg sends a message. On error, SendMsg aborts the stream and the
	// error is returned directly.
	//
	// SendMsg blocks until:
	//   - There is sufficient flow control to schedule m with the transport, or
	//   - The stream is done, or
	//   - The stream breaks.
	//
	// SendMsg does not wait until the message is received by the client. An
	// untimely stream closure may result in lost messages.
	//
	// It is safe to have a goroutine calling SendMsg and another goroutine
	// calling RecvMsg on the same stream at the same time, but it is not safe
	// to call SendMsg on the same stream in different goroutines.
	SendMsg(m interface{}) error
	// RecvMsg blocks until it receives a message into m or the stream is
	// done. It returns io.EOF when the client has performed a CloseSend. On
	// any non-EOF error, the stream is aborted and the error contains the
	// RPC status.
	//
	// It is safe to have a goroutine calling SendMsg and another goroutine
	// calling RecvMsg on the same stream at the same time, but it is not
	// safe to call RecvMsg on the same stream in different goroutines.
	RecvMsg(m interface{}) error
}
```

ServerStream defines the server-side behavior of a streaming RPC.

All errors returned from ServerStream methods are compatible with the status package.

### type [ServerTransportStream](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L1548) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ServerTransportStream)

```
type ServerTransportStream interface {
	Method() string
	SetHeader(md metadata.MD) error
	SendHeader(md metadata.MD) error
	SetTrailer(md metadata.MD) error
}
```

ServerTransportStream is a minimal interface that a transport stream must implement. This can be used to mock an actual transport stream for tests of handler code that use, for example, grpc.SetHeader (which requires some stream to be in context).

See also NewContextWithServerTransportStream.

This API is EXPERIMENTAL.

### func [ServerTransportStreamFromContext](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L1560) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ServerTransportStreamFromContext)

```
func ServerTransportStreamFromContext(ctx context.Context) ServerTransportStream
```

ServerTransportStreamFromContext returns the ServerTransportStream saved in ctx. Returns nil if the given context has no stream associated with it (which implies it is not an RPC invocation context).

This API is EXPERIMENTAL.

### type [ServiceConfig](https://github.com/grpc/grpc-go/blob/v1.30.0/service_config.go#L79) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ServiceConfig)

```
type ServiceConfig struct {
	serviceconfig.Config

	// LB is the load balancer the service providers recommends. The balancer
	// specified via grpc.WithBalancerName will override this.  This is deprecated;
	// lbConfigs is preferred.  If lbConfig and LB are both present, lbConfig
	// will be used.
	LB *string

	// Methods contains a map for the methods in this service.  If there is an
	// exact match for a method (i.e. /service/method) in the map, use the
	// corresponding MethodConfig.  If there's no exact match, look for the
	// default config for the service (/service/) and use the corresponding
	// MethodConfig if it exists.  Otherwise, the method has no MethodConfig to
	// use.
	Methods map[string]MethodConfig
	// contains filtered or unexported fields
}
```

ServiceConfig is provided by the service provider and contains parameters for how clients that connect to the service should behave.

Deprecated: Users should not use this struct. Service config should be received through name resolver, as specified here https://github.com/grpc/grpc/blob/master/doc/service_config.md

### type [ServiceDesc](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L72) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ServiceDesc)

```
type ServiceDesc struct {
	ServiceName string
	// The pointer to the service interface. Used to check whether the user
	// provided implementation satisfies the interface requirements.
	HandlerType interface{}
	Methods     []MethodDesc
	Streams     []StreamDesc
	Metadata    interface{}
}
```

ServiceDesc represents an RPC service's specification.

### type [ServiceInfo](https://github.com/grpc/grpc-go/blob/v1.30.0/server.go#L583) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#ServiceInfo)

```
type ServiceInfo struct {
	Methods []MethodInfo
	// Metadata is the metadata specified in ServiceDesc when registering service.
	Metadata interface{}
}
```

ServiceInfo contains unary RPC method info, streaming RPC method info and metadata for a service.

### type [Stream](https://github.com/grpc/grpc-go/blob/v1.30.0/stream.go#L65) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#Stream)

```
type Stream interface {
	// Deprecated: See ClientStream and ServerStream documentation instead.
	Context() context.Context
	// Deprecated: See ClientStream and ServerStream documentation instead.
	SendMsg(m interface{}) error
	// Deprecated: See ClientStream and ServerStream documentation instead.
	RecvMsg(m interface{}) error
}
```

Stream defines the common interface a client or server stream has to satisfy.

Deprecated: See ClientStream and ServerStream documentation instead.

### type [StreamClientInterceptor](https://github.com/grpc/grpc-go/blob/v1.30.0/interceptor.go#L39) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#StreamClientInterceptor)

```
type StreamClientInterceptor func(ctx context.Context, desc *StreamDesc, cc *ClientConn, method string, streamer Streamer, opts ...CallOption) (ClientStream, error)
```

StreamClientInterceptor intercepts the creation of ClientStream. It may return a custom ClientStream to intercept all I/O operations. streamer is the handler to create a ClientStream and it is the responsibility of the interceptor to call it. This is an EXPERIMENTAL API.

### type [StreamDesc](https://github.com/grpc/grpc-go/blob/v1.30.0/stream.go#L53) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#StreamDesc)

```
type StreamDesc struct {
	StreamName string
	Handler    StreamHandler

	// At least one of these is true.
	ServerStreams bool
	ClientStreams bool
}
```

StreamDesc represents a streaming RPC service's method specification.

### type [StreamHandler](https://github.com/grpc/grpc-go/blob/v1.30.0/stream.go#L50) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#StreamHandler)

```
type StreamHandler func(srv interface{}, stream ServerStream) error
```

StreamHandler defines the handler called by gRPC server to complete the execution of a streaming RPC. If a StreamHandler returns an error, it should be produced by the status package, or else gRPC will use codes.Unknown as the status code and err.Error() as the status message of the RPC.

### type [StreamServerInfo](https://github.com/grpc/grpc-go/blob/v1.30.0/interceptor.go#L64) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#StreamServerInfo)

```
type StreamServerInfo struct {
	// FullMethod is the full RPC method string, i.e., /package.service/method.
	FullMethod string
	// IsClientStream indicates whether the RPC is a client streaming RPC.
	IsClientStream bool
	// IsServerStream indicates whether the RPC is a server streaming RPC.
	IsServerStream bool
}
```

StreamServerInfo consists of various information about a streaming RPC on server side. All per-rpc information may be mutated by the interceptor.

### type [StreamServerInterceptor](https://github.com/grpc/grpc-go/blob/v1.30.0/interceptor.go#L77) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#StreamServerInterceptor)

```
type StreamServerInterceptor func(srv interface{}, ss ServerStream, info *StreamServerInfo, handler StreamHandler) error
```

StreamServerInterceptor provides a hook to intercept the execution of a streaming RPC on the server. info contains all the information of this RPC the interceptor can operate on. And handler is the service method implementation. It is the responsibility of the interceptor to invoke handler to complete the RPC.

### type [Streamer](https://github.com/grpc/grpc-go/blob/v1.30.0/interceptor.go#L34) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#Streamer)

```
type Streamer func(ctx context.Context, desc *StreamDesc, cc *ClientConn, method string, opts ...CallOption) (ClientStream, error)
```

Streamer is called by StreamClientInterceptor to create a ClientStream.

### type [TrailerCallOption](https://github.com/grpc/grpc-go/blob/v1.30.0/rpc_util.go#L220) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#TrailerCallOption)

```
type TrailerCallOption struct {
	TrailerAddr *metadata.MD
}
```

TrailerCallOption is a CallOption for collecting response trailer metadata. The metadata field will be populated *after* the RPC completes. This is an EXPERIMENTAL API.

### type [UnaryClientInterceptor](https://github.com/grpc/grpc-go/blob/v1.30.0/interceptor.go#L31) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#UnaryClientInterceptor)

```
type UnaryClientInterceptor func(ctx context.Context, method string, req, reply interface{}, cc *ClientConn, invoker UnaryInvoker, opts ...CallOption) error
```

UnaryClientInterceptor intercepts the execution of a unary RPC on the client. invoker is the handler to complete the RPC and it is the responsibility of the interceptor to call it. This is an EXPERIMENTAL API.

### type [UnaryHandler](https://github.com/grpc/grpc-go/blob/v1.30.0/interceptor.go#L54) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#UnaryHandler)

```
type UnaryHandler func(ctx context.Context, req interface{}) (interface{}, error)
```

UnaryHandler defines the handler invoked by UnaryServerInterceptor to complete the normal execution of a unary RPC. If a UnaryHandler returns an error, it should be produced by the status package, or else gRPC will use codes.Unknown as the status code and err.Error() as the status message of the RPC.

### type [UnaryInvoker](https://github.com/grpc/grpc-go/blob/v1.30.0/interceptor.go#L26) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#UnaryInvoker)

```
type UnaryInvoker func(ctx context.Context, method string, req, reply interface{}, cc *ClientConn, opts ...CallOption) error
```

UnaryInvoker is called by UnaryClientInterceptor to complete RPCs.

### type [UnaryServerInfo](https://github.com/grpc/grpc-go/blob/v1.30.0/interceptor.go#L43) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#UnaryServerInfo)

```
type UnaryServerInfo struct {
	// Server is the service implementation the user provides. This is read-only.
	Server interface{}
	// FullMethod is the full RPC method string, i.e., /package.service/method.
	FullMethod string
}
```

UnaryServerInfo consists of various information about a unary RPC on server side. All per-rpc information may be mutated by the interceptor.

### type [UnaryServerInterceptor](https://github.com/grpc/grpc-go/blob/v1.30.0/interceptor.go#L60) [¶](https://pkg.go.dev/google.golang.org/grpc?tab=doc#UnaryServerInterceptor)

```
type UnaryServerInterceptor func(ctx context.Context, req interface{}, info *UnaryServerInfo, handler UnaryHandler) (resp interface{}, err error)
```

UnaryServerInterceptor provides a hook to intercept the execution of a unary RPC on the server. info contains all the information of this RPC the interceptor can operate on. And handler is the wrapper of the service method implementation. It is the responsibility of the interceptor to invoke handler to complete the RPC.