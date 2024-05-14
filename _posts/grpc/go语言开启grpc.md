---
title: go-micro开启grpc
date: 2023-12-05 23:01:45
tags: 微服务
categories: 
- 微服务
- grpc配置
---





## 开启go mod

```bash
go env -w GO111MODULE=on

#阿里云
go env -w GOPROXY=https://mirrors.aliyun.com/goproxy/,direct
```

## 安装protobuf编译器

[官网教程](https://grpc.io/docs/protoc-installation/)
> 必须保证proto的版本>=3

```bash
sudo apt update
sudo apt install protobuf-compiler
protoc --version  # Ensure compiler version is 3+
```

# 安装protoc-grpc插件
[官网教程](https://grpc.io/docs/languages/go/quickstart/)
```bash
go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2
```

## 配置环境变量
这一步主要是使用protoc-gen-go
```bash
#在.bashrc文件最下面添加
export PATH="$PATH:$(go env GOPATH)/bin"

#shell里运行
source ~/.bashrc
```

[https://github.com/go-micro/generator](https://github.com/go-micro/generator)



# micro框架（V4版本）
官网网址：[https://github.com/go-micro](https://github.com/go-micro)
包含

- [Framework](https://github.com/go-micro/go-micro) - The core Go framework for development
- [Generator](https://github.com/go-micro/generator) - Protobuf code generation to use with protoc
- [Examples](https://github.com/go-micro/examples) - Individual examples for use cases & features
- [Dashboard](https://github.com/go-micro/dashboard) - Browser based access for Go Micro services
- [Plugins](https://github.com/go-micro/plugins) - Implementations for redis, kafka, etcd, gRPC..
- [Demo](https://github.com/go-micro/demo) - A full feaured demo app
- [CLI](https://github.com/go-micro/cli) - Command line interfac

## 安装protobuf编译器
```bash
sudo apt update
sudo apt install protobuf-compiler
protoc --version  # Ensure compiler version is 3+
```

## 安装proto-micro插件
```bash
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install github.com/go-micro/generator/cmd/protoc-gen-micro@latest
```

# 生成micro文件

```bash
protoc --proto_path=. --go_out=. --micro_out=. *.proto
```


**example**

1.准备greeter.proto文件
```
syntax = "proto3";

package greeter;
option go_package = "/proto;greeter";

service Greeter {
	rpc Hello(Request) returns (Response) {}
}

message Request {
	string name = 1;
}

message Response {
	string msg = 1;
}
```

2.生成code
```
protoc --proto_path=. --micro_out=. --go_out=. greeter.proto
```



## Plugins注册到consul

(1)查看文件[https://github.com/go-micro/plugins/blob/main/v4/registry/consul/go.mod](https://github.com/go-micro/plugins/blob/main/v4/registry/consul/go.mod)
(2)导入包名`github.com/go-micro/plugins/v4/registry/consul`
(3)代码示例
```go
package main

import (
    "time"
    "github.com/go-micro/plugins/v4/registry/consul"
    "go-micro.dev/v4"
    "filestore-server/service/account/handler"
    proto "filestore-server/service/account/proto"
)

func main() {
    registry := consul.NewRegistry() //consul

    // 创建服务时指定注册中心
    service := micro.NewService(
        micro.Name("go.micro.service.account"),
        micro.RegisterTTL(time.Second*10),      // 声明超时时间, 避免consul不主动删掉已失去心跳的服务节点
		micro.RegisterInterval(time.Second*5),
        micro.Registry(registry),
    )

    service.Init()

    //注册服务
    proto.RegisterUserServiceHandler(service.Server(), new(handler.User))

    //开启服务
    if err := service.Run(); err != nil {
        panic(err)
    }
}
```


# micro使用grpc协议
[https://github.com/go-micro/plugins/blob/main/v4/server/grpc/README.md](https://github.com/go-micro/plugins/blob/main/v4/server/grpc/README.md)

