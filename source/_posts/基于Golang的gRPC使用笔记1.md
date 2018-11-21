---
title: gRPC使用 -- Golang版示例
catalog: true
date: 2018-11-12 14:00:00
subtitle:
header-img:
tags:
- gRPC
- golang
catagories:
- Hexo
---
# RPC入门

## RPC框架原理
```
    RPC框架的目的就是让远程服务调用更加简单,透明，RPC框架负责屏蔽底层的传输方式(TCP或者UDP),
    序列化方式(XML/JSON/二进制)和通信细节。服务调用者可以向本地接口一样调用远程的服务提供者，
    而不需要关心底层通信细节和调用过程.
```
![rpc原理图](rpc原理图.png)

## gRPC

### gRPC简介
```
    gRPC 是一个高性能、开源和通用的 RPC 框架，面向服务端和移动端,基于HTTP/2设计.
```
![gRPC调用示例](gRPC调用示例.png)

### gRPC特点
![gRPC特点](gRPC特点.png)

### Golang gRPC 示例

#### 1、安装gRPC runtime
```
go get google.golang.org/grpc
```

#### 2、protocal buffer安装
```
从https://github.com/google/protobuf/releases下载安装包，
例如：protobuf-cpp-3.0.0-beta-3.zip，解压后
./configure
make && make install
再添加环境变量：export LD_LIBRARY_PATH=/usr/local/lib，之后protoc命令即可运行    
```

#### 3、安装GoLang protoc 插件
```
go get -a github.com/golang/protobuf/protoc-gen-go
```

#### 4、定义service
一个RPC service就是一个能够通过参数和返回值进行远程调用的method，我们可以简单地将它理解成一个函数。因为gRPC是通过将数据编码成protocal buffer来实现传输的。因此，我们通过protocal buffers interface definitioin language(IDL)来定义service method，同时将参数和返回值也定义成protocal buffer message类型。具体实现如下所示，包含下面代码的文件叫helloworld.proto：
```
syntax = "proto3";
 
option java_package = "io.grpc.examples";
 
package helloworld;
 
// The greeter service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}
 
// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}
 
// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

#### 5.生成Golang通用代码
```
接着，根据上述定义的service，我们可以利用protocal buffer compiler ，
即protoc生成相应的服务器端和客户端的GoLang代码。生成的代码中包含了客户端能够进行RPC的方法以及服务器端需要进行实现的接口。
假设现在所在的目录是$GOPATH/src/helloworld/helloworld，我们将通过如下命令生成gRPC对应的GoLang代码：
protoc -I ./ helloworld.proto --go_out=plugins=grpc:.
```

#### 6.生成相关的RPC的客户端和服务端
在目录$GOPATH/src/helloworld/下创建server.go 和client.go，分别用于服务器和客户端的实现

##### 服务端代码
```
package main
 
// server.go
 
import (
    "log"
    "net"
 
    "golang.org/x/net/context"
    "google.golang.org/grpc"
    pb "helloworld/helloworld"
)
 
const (
    port = ":50051"
)
 
type server struct {}
 
func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
    return &pb.HelloReply{Message: "Hello " + in.Name}, nil
}
 
func main() {
    lis, err := net.Listen("tcp", port)
    if err != nil {
        log.Fatal("failed to listen: %v", err)
    }
    s := grpc.NewServer()
    pb.RegisterGreeterServer(s, &server{})
    s.Serve(lis)
}
```

##### 客户端代码
```
package main
 
//client.go
 
import (
    "log"
    "os"
 
    "golang.org/x/net/context"
    "google.golang.org/grpc"
    pb "helloworld/helloworld"
)
 
const (
    address     = "localhost:50051"
    defaultName = "world"
)
 
func main() {
    conn, err := grpc.Dial(address, grpc.WithInsecure())
    if err != nil {
        log.Fatal("did not connect: %v", err)
    }
    defer conn.Close()
    c := pb.NewGreeterClient(conn)
 
    name := defaultName
    if len(os.Args) >1 {
        name = os.Args[1]
    }
    r, err := c.SayHello(context.Background(), &pb.HelloRequest{Name: name})
    if err != nil {
        log.Fatal("could not greet: %v", err)
    }
    log.Printf("Greeting: %s", r.Message)
}
```
这里需要注意的是包pb是我们之前生成的helloworld.pb.go所在的包，
并非必须如上述代码所示在$GOPATH/src/helloworld/helloworld目录下

#### 7.运行示例
```
go run server.go
go run client.go
```
