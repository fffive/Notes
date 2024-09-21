## 介绍

在 gRPC 中，客户端应用程序可以直接调用其他计算机上的服务器应用程序上的方法，就像它是本地对象一样，从而使您可以更轻松地创建分布式应用程序和服务。与许多 RPC 系统一样，gRPC 基于定义服务的理念，指定可以使用其参数和返回类型远程调用的方法。在服务器端，服务器实现此接口并运行 gRPC 服务器来处理客户端调用。在客户端，客户端有一个存根，它提供与服务器相同的方法。

![](grpcprocess.svg)
### 使用底层通信格式 -- Protocol Buffers

默认情况下，gRPC 使用 [Protocol Buffers](https://protobuf.dev/overview)，这是 Google 用于序列化结构化数据的成熟开源机制（尽管它可以与 JSON 等其他数据格式一起使用）。其性能也比JSON等要好，从压缩比中可以清楚发现。

#### 使用 Protocol Buffers

1. 定义要在 _proto 文件中_序列化的数据的结构_ ```
```proto title:example.proto
message Person {
	string name = 1; // 记住这是标志 不是值
	int32 id = 2;
	bool has_ponycopter = 3;
 }
```
2. 使用工具 _protoc_ 从 proto 定义中生成首选语言的数据访问类。

#### grpc插件

如果你需要使用grpc插件，使用`protoc --go_out=. --go-grpc_out=. proto/helloworld.proto`

```proto custom_server.proto
// 服务接口定义
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// 请求信息结构体
message HelloRequest {
  string name = 1;
}

// 返回接受信息
message HelloReply {
  string message = 1;
}
```

#### Protocol 协议版本

现在已经出现protocol 2 和 protocol 3，建议使用版本3。了解更多查看[protobuf](protobuf.md)

## 核心概念

### 服务定义

与许多 RPC 系统一样，gRPC 基于定义服务的想法，指定可以远程调用的方法及其参数和返回类型。

#### 四种服务方法
1. 一元RPC `rpc SayHello(HelloRequest) returns (HelloResponse);`
2. 服务器流式 RPC  `rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse);`
3. 客户端流式 RPC`rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse);`
4. 双向流式 RPC `rpc BidiHello(stream HelloRequest) returns (stream HelloResponse);
==想要了解更多：== [点击这里](#案例)

### RPC 生命周期 #todo


## 案例

### 实现流式RPC

#### protoc 进行编码

```proto title:stream.proto
syntax = "proto3";
package stream;
option go_package = ".;proto";

service Greeter {
    rpc GetStream(StreamReqData) returns (stream StreamResData);
    rpc PutStream(stream StreamReqData) returns (StreamResData);
    rpc AllStream(stream StreamReqData) returns (stream StreamResData);
}

message StreamReqData {
    string data = 1;
}

message StreamResData {
    string data = 1;
}
```

**执行命令**：`protoc -I . stream.proto --go_out=. --go-grpc_out=.`
==了解命令：==  [grpc-goexample-cmd](https://grpc.io/docs/languages/go/quickstart/#regenerate-grpc-code)
 
#### 服务端流

```go title:server.go 
type GreeterServer interface {
	// 服务端流
	GetStream(*StreamReqData, grpc.ServerStreamingServer[StreamResData]) error

	// 客户端流
	PutStream(grpc.ClientStreamingServer[StreamReqData, StreamResData]) error

	// 双向流
	AllStream(grpc.BidiStreamingServer[StreamReqData, StreamResData]) error
}
```

#### 客户端流

```go title:client.go
type GreeterClient interface {
	GetStream(ctx context.Context, in *StreamReqData, opts ...grpc.CallOption) (grpc.ServerStreamingClient[StreamResData], error)
	PutStream(ctx context.Context, opts ...grpc.CallOption) (grpc.ClientStreamingClient[StreamReqData, StreamResData], error)
	AllStream(ctx context.Context, opts ...grpc.CallOption) (grpc.BidiStreamingClient[StreamReqData, StreamResData], error)
}
```
