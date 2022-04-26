---
layout: post
title:  python 에서 gRPC 테스트 해보기
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [python, gRPC]
categories : [Programming]
---

### gRPC란
* RPC 란
    - Remote Procedure Call의 약자로 간편하게 다른 서비스의 함수 프로시저등을 호출할 수 있는 프로토콜이다.
      다양한 언어를 지원하므로 언어가 다르거나 한 다른 서비스에 있는 함수나 프로시저를 호출하여 사용할 수 있다.
      다양한 언어를 사용하는 MSA 구조에서 더욱 좋을 것 같다.
* gRPC
    - gRPC는 구글에서 만든 RPC로 TCP/IP 와 HTTP/2를 프로토콜을 사용한다.
    - gRPC는 IDL(Interface Definition Language)로 protocol buffer를 사용하는데 .proto 파일에 직렬화할 데이터를 정의한다.

---  
### gRPC proto 파일  

```
syntax = "proto3";

package hello;

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// Message Definition
message HelloRequest {
  string name = 1;
  int32 num = 2;
  bool has_boolean = 3;
}

message HelloReply {
  string message = 1;
  int32 id = 2;
  bool has_boolean = 3;
}
```

파일을 작성 후 

```
$ python3 -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. hello.proto
```
명령어를 입력하면

hello_pb2.py  
hello_pb2_grpc.py  

파일이 생성된다.  
해당 파일은 python gRPC server와 client에 import 하여 사용된다.  

---
### python gRPC   

server  
```
from concurrent import futures
import time

import grpc

import hello_pb2
import hello_pb2_grpc

class Greeter(hello_pb2_grpc.GreeterServicer):
    def SayHello(self, request, context):
        return hello_pb2.HelloReply(message=f'Hello, {request.name}!, {request.num}!, {request.has_boolean}!')


def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    hello_pb2_grpc.add_GreeterServicer_to_server(Greeter(), server)
    server.add_insecure_port('localhost:50051')
    server.start()
    try:
        while True:
            time.sleep(60)
    except KeyboardInterrupt:
        server.stop(0)


if __name__ == '__main__':
    serve()
    
```   

client
```
from __future__ import print_function

import grpc

import hello_pb2
import hello_pb2_grpc


def run():
    channel = grpc.insecure_channel('localhost:50051')
    stub = hello_pb2_grpc.GreeterStub(channel)
    response = stub.SayHello(hello_pb2.HelloRequest(name='python', num=1, has_boolean=True))
    print("Greeter client received: " + response.message)


if __name__ == '__main__':
    run()
```   

파일을 작성 후 run 해보면 다음과 같이 결과값을 확인할 수 있다.  

```
Greeter client received: Hello, python!, 1!, True!
```


---