# Zeromq学习记录
## 1、拯救世界（Fixing the World）
如何解释ZeroMQ？    
- 用它可以完成的精彩的事情来解释：ZeroMQ是打了激素的套接字（It's sockets on steroids）；ZeroMQ是带有路由的邮箱。
- 用人们对它的评价来解释：ZeroMQ很快；复杂性不见了，ZeroMQ让事情变得简单；ZeroMQ解放了思维。
- 通过比较来解释：ZeroMQ更小、更简单。

要拯救这个世界，需要做两件事：
- 解决“如何在任何地点连接任何代码到任何代码（how to connect any code to any code, anywhere）”的问题
- 将解决方法封装到尽可能简单的建造块中，让人们容易理解和使用
>ZeroMQ看起来像个嵌入的网络库，但是起到了并发框架的作用。ZeroMQ提供可以在进程内、进程间、TCP和多播等各种传输端点间传递消息的套接字。使用扇出、发布-订阅、任务分布、请求-回应等模式，可以对套接字进行N到N连接。ZeroMQ很快，足以用于集群产品。ZeroMQ的异步IO模型让你可以扩展到多核应用，就像异步消息处理任务一样。ZeroMQ有多种语言的API，可以运行在大多数操作系统中。ZeroMQ来自iMatix，使用LGPL开源协议。
## 2、请求回应模式
从一个简单例子开始：客户端发送Hello给服务器；服务器回应World。
1. 服务器代码
```
#include <zmq.h>
#include <zmq_helpers.h>
#include <stdio.h>
#pragma comment(lib,"libzmq-v100-mt.lib")
int main(void){
    void* ctx = zmq_ctx_new();
    void* responder = zmq_socket(ctx,ZMQ_REP);
    zmq_bind(responder,"tcp://*:5555");
    while(1){
        zmq_msg_t request;
        zmq_msg_init(&request);
        zmq_recvmsg(responder,&request,0);
        printf("收到%s\n",(char*)zmq_msg_data(&request));
        zmq_msg_close(&request);
        s_sleep(1000);
        zmq_msg_t reply;
        zmq_msg_init_size(&reply,6);
        memcpy(zmq_msg_data(&reply),"World",6);
        printf("发送World\n");
        zmq_sendmsg(responder,&reply,0);
        zmq_msg_close(&reply);
    }
    zmq_close(responder);
    zmq_ctx_destroy(ctx);
    return 0;
}
```