gRPC



>安装

个人使用的MBP，所以百度了下Mac上安装`protoc`命令的过程，参照地址如下：https://blog.csdn.net/u014534808/article/details/80203018



> 项目结构图

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gft5rcgd99j30a0078754.jpg)

> 创建.proto文件

```protobuf
syntax = "proto3"; // 指定proto版本
package pb; // 包名

message HelloRequest {
  string username = 1; // 定义参数
}

message HelloResponse {
  string message = 1; // 定义返回值
}

service HelloService { // 定义服务接口
  rpc SayHello(HelloRequest) returns (HelloResponse){}
  rpc SayBye(HelloRequest) returns (HelloResponse){}
}

```

​	如上面代码所示，正是gRPC = HTTP2 + Protocol Buffer 