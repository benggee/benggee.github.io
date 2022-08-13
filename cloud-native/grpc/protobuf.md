# 1. protobuf的使用

## 官方使用方式

protoc-gen-go工具安装：

```shell
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.26
$ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.1
```

将gopath的bin目录添加到系统path

```shell
$ export PATH="$PATH:$(go env GOPATH)/bin"
```

通过proto文件生成pb.go

```shell
$ protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative helloworld.proto
```

这种方式生成的pb文件有两个

```shell
$ helloworld.pb.go
$ helloworld_grpc.pb.go
```

helloworld.pb.go里面生成了对应的参数信息

helloworld_grpc.pb.go里面生成了对应的方法

# 2. 异常

## 2.1. 缺少选项

```shell
 WARNING: Missing 'go_package' option in "helloworld.proto"
```

在proto文件加入go_package选项

```go
syntax = "proto3"; 
package helloworld;
option go_package = "../helloword";
```



## 2.2. 接口实现问题

```
Cannot use 'ret' (type *HelloWorldServiceWithGRPC) as the type helloworld.HelloWorldServerServer Type does not implement 'helloworld.HelloWorldServerServer' as some methods are missing: mustEmbedUnimplementedHelloWorldServerServer()
```

这种情况，可以将UnsafeHelloWorldServerServer嵌入到实现的struct里，如下：

```go
type HelloWorldService struct {
  ...
	helloworld.UnsafeHelloWorldServerServer
}
```



## 2.3. 版本问题

```shell
../../protos/helloworld/helloworld_grpc.pb.go:15:11: undefined: grpc.SupportPackageIsVersion7
../../protos/helloworld/helloworld_grpc.pb.go:34:5: undefined: grpc.ClientConnInterface
../../protos/helloworld/helloworld_grpc.pb.go:37:31: undefined: grpc.ClientConnInterface
../../protos/helloworld/helloworld_grpc.pb.go:167:35: undefined: grpc.ServiceRegistrar
```

出现这种情况有两种解决方法：

#### （1）升级gRPC版本

> ```
> // This is a compile-time assertion to ensure that this generated file
> // is compatible with the grpc package it is being compiled against.
> // Requires gRPC-Go v1.32.0 or later.
> ```

但这种方式，可能会导致其它组件编译不成功，例如：

```shell
../../vendor/github.com/coreos/etcd/clientv3/balancer/picker/err.go:37:44: undefined: balancer.PickOptions
../../vendor/github.com/coreos/etcd/clientv3/balancer/picker/roundrobin_balanced.go:55:54: undefined: balancer.PickOptions
# github.com/coreos/etcd/clientv3/balancer/resolver/endpoint
../../vendor/github.com/coreos/etcd/clientv3/balancer/resolver/endpoint/endpoint.go:114:78: undefined: resolver.BuildOption
../../vendor/github.com/coreos/etcd/clientv3/balancer/resolver/endpoint/endpoint.go:182:31: undefined: resolver.ResolveNowOption
```

#### （2）降protoc-gen-go版本

```shell
$ go install github.com/golang/protobuf/protoc-gen-go@1.3.0 
```

 然后使用下面的命令生成pb文件

```shell
 $ protoc --go_out=plugins=grpc:. helloworld.proto 
```

