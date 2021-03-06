evpp
---

`evpp` is a modern C++ network library for developing high performance network services in TCP/UDP/HTTP protocols.
`evpp` provides a TCP Server to support multi-threaded nonblocking event-drive server and also a HTTP, UDP Server to support http and udp prococol.

# Features

1. Modern C++11 interface
1. Modern functional/bind style callback instead of C-style function pointer.
1. A nonblocking multi-threaded TCP server
1. A nonblocking TCP client
1. A nonblocking multi-threaded HTTP server based on the buildin http server of libevent
1. A nonblocking HTTP client
1. A nonblocking multi-threaded UDP server
1. EventLoop/Thread Pool/Timer

And also provides some libraries base on `evpp`:

1. `evmc` a nonblocking async C++ memcached (or membase cluster) client library.
2. `evnsq` a nonblocking async C++ NSQ client library. See [evnsq readme](apps/evnsq/readme.md) for details.

TODO:

1. A async redis client
2. Add `zipkin` tracing support
3. Add examples : asio chat room


# Getting Started

Please see [Quick Start](docs/quick_start.md)

# Examples

## A echo TCP server

```cpp
#include <evpp/exp.h>
#include <evpp/tcp_server.h>
#include <evpp/tcp_conn.h>
#include <evpp/buffer.h>

void OnMessage(const evpp::TCPConnPtr& conn,
               evpp::Buffer* msg,
               evpp::Timestamp ts) {
    std::string s = msg->NextAllString();
    LOG_INFO << "Received a message [" << s << "]";
    conn->Send(s);
    if (s == "quit" || s == "exit") {
        conn->Close();
    }
}

int main(int argc, char* argv[]) {
    std::string addr = std::string("0.0.0.0:9999");
    evpp::EventLoop loop;
    evpp::TCPServer server(&loop, addr, "TCPEcho", 0);
    server.SetMessageCallback(&OnMessage);
    server.Init();
    server.Start();
    loop.Run();
    return 0;
}
```

### A echo HTTP server

```cpp
#include <evpp/exp.h>
#include <evpp/http/http_server.h>

void RequestHandler(evpp::EventLoop* loop,
                    const evpp::http::ContextPtr& ctx,
                    const evpp::http::HTTPSendResponseCallback& cb) {
    cb(ctx->body.ToString());
}

int main(int argc, char* argv[]) {
    std::vector<int> ports = {9009, 23456, 23457};
    int thread_num = 2;
    evpp::http::Server server(thread_num);
    server.RegisterHandler("/echo", &RequestHandler);
    server.Init(ports);
    server.Start();
    while (!server.IsStopped()) {
        usleep(1);
    }
    return 0;
}

```


### A echo UDP server

```cpp
#include <evpp/exp.h>
#include <evpp/udp/udp_server.h>
#include <evpp/udp/udp_message.h>

void DefaultHandler(evpp::EventLoop* loop, evpp::udp::MessagePtr& msg) {
    evpp::udp::SendMessage(msg);
}

int main(int argc, char* argv[]) {
    std::vector<int> ports = {1053, 5353};
    evpp::udp::Server server;
    server.SetMessageHandler(&DefaultHandler);
    server.Init(ports);
    server.Start();

    while (!server.IsStopped()) {
        usleep(1);
    }
    return 0;
}
```

### More examples

Please see the source code in [`examples`](https://github.com/Qihoo360/evpp/tree/master/examples).

# Thanks

Thanks for the support of [Qihoo360](http://www.360.cn "http://www.360.cn").

Thanks for [libevent](https://github.com/libevent/libevent), [glog](https://github.com/google/glog), [gtest](https://github.com/google/googletest) projects.

`evpp` is highly inspired by [muduo](https://github.com/chenshuo/muduo "https://github.com/chenshuo/muduo"). Thanks for the great work of [Chen Shuo](https://github.com/chenshuo "https://github.com/chenshuo")

