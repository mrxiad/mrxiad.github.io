---
title: asio普通的异步服务器
date: 2023-11-13 15:52:00
tags: c++
categories: 
- 后端
- 网络编程
---







> 本文主要实现一个异步应答服务器





> 分文两个类
>
> session：session类主要是处理客户端消息收发的会话类，属于服务端
>
>
> server：server类为服务器接收连接的管理类，用于管理多个session，属于服务端



# session类

**成员**

```cpp
 class Session
{
public:
    Session(boost::asio::io_context& ioc):_socket(ioc){
    }
    tcp::socket& Socket() {
        return _socket;
    }
    void Start();
private:
    //读操作回调函数需要2个参数
    void handle_read(const boost::system::error_code & error, size_t bytes_transfered);
    //写操作只需要1个参数
    void handle_write(const boost::system::error_code& error);
    tcp::socket _socket;
    enum {max_length = 1024};
    char _data[max_length];
};
```

-  _data用来接收客户端传递的数据
-  _socket为单独处理客户端读写的socket。
-  handle_read和handle_write分别为读回调函数和写回调函数。

> 注意，读写回调函数的参数数量不同



**具体实现**



## (1)`start函数`

```cpp
void session::Start() {
    //初始化
    memset(_data, 0, max_length);
    //读操作，读完调用handle_read
    _socket.async_read_some(boost::asio::buffer(_data, max_length),
        std::bind(&session::handle_read, this, placeholders::_1,
            placeholders::_2)
    );
}
```

在Start方法中我们调用异步读操作，监听对端发送的消息。当对端发送数据后，触发handle_read函数





## (2)`handle_read函数`

```cpp
void session::handle_read(const boost::system::error_code& error, size_t bytes_transfered) {
    if (!error) {
        cout << "server receive data is " << _data << endl;

        //读完后，异步调用写操作，返回消息给发送者，返回后，调用handle_write函数
        boost::asio::async_write(_socket, boost::asio::buffer(_data, bytes_transfered),
            std::bind(&session::handle_write, this, placeholders::_1));
    }
    else {
        delete this;
    }
}
```

handle_read函数内将收到的数据发送给对端，当发送完成后触发handle_write回调函数。





## (3)`handle_write函数`

```cpp
void session::handle_write(const boost::system::error_code& error) {
    if (!error) {
        memset(_data, 0, max_length);

        //发送完后，调用异步读操作，读取发送者发送的消息，读取完后，再次调用handle_read函数，形成循环
        _socket.async_read_some(boost::asio::buffer(_data, max_length), std::bind(&session::handle_read,
            this, placeholders::_1, placeholders::_2));
    }
    else {
        delete this;
    }
}
```

handle_write函数内又一次监听了读事件，如果对端有数据发送过来则触发handle_read，我们再将收到的数据发回去。从而达到应答式服务的效果。



# server类

server类是服务器接收连接的管理类

```cpp
class server {
public:
    //构造函数，初始化acceptor对象
    server(boost::asio::io_context& ioc, short port);
private:

    //服务器开始接受连接
    void start_accept();

    //新连接触发后的回调函数
    void handle_accept(session* new_session, const boost::system::error_code& error);

    boost::asio::io_context& _ioc;
    tcp::acceptor _acceptor;
};
```

- start_accept将要接收连接的acceptor绑定到服务上，其内部就是将accpeptor对应的socket描述符绑定到epoll或iocp模型上，实现事件驱动。
- handle_accept为新连接到来后触发的回调函数。





## （a) `构造函数`

```cpp
server::server(boost::asio::io_context& ioc, short port) :_ioc(ioc),
//初始化acceptor对象（协议+监听端口，表示监听主机上的所有 IPv4 地址上的指定端口）
_acceptor(ioc, tcp::endpoint(tcp::v4(), port)) {
    start_accept();//启动接受连接
}
```

- 初始化ioc和accecptor
- 启动接受连接



## (b)`start_accept函数`

```cpp
void server::start_accept() {
    //保存新连接的socket对象
    session* new_session = new session(_ioc);

    //异步接受连接，调用handle_accept函数
    _acceptor.async_accept(new_session->Socket(),
        std::bind(&server::handle_accept, this, new_session, placeholders::_1));
}
```

- 保存socket对象
- 异步接受连接，然后调用`handle_accecpt函数`
- 使用异步接受连接，因为防止阻塞

> 注意：开启异步接受连接，不代表立马会有客户端连接过来，而是等客户端连接过来才触发async_accept函数，连接完后调用handle_accept函数！！！！



## (c)`handle_accept函数`

```cpp
void Server::handle_accept(Session* new_session, const boost::system::error_code& error) {
    if (!error) {
        //开启异步监听客户端的消息，这一步不会阻塞
        new_session->Start();
    }
    else {
        delete new_session;
    }
    //继续接受新连接
    start_accept();
}

```

> 注意，start是异步读操作，不会阻塞





# 启动服务器



main.cpp

```cpp
#include <boost/asio.hpp>
#include <iostream>
#include <boost/array.hpp>
#include "server.h"
#include "session.h"
using boost::asio::ip::tcp;
namespace asio = boost::asio;
int main() {
    
    try {
		asio::io_context io_context;
		server s(io_context, 8080);
		io_context.run();
	}
	catch (std::exception& e) {
		std::cerr << e.what() << std::endl;
	}
    
    return 0;
}
```



**io_context.run()的作用**

1. **启动事件循环**：
	- `io_context.run()` 调用开始一个事件处理循环，负责执行所有排队的异步操作的处理程序。
2. **执行异步操作处理程序**：
	- 这个循环处理由异步操作（如异步读取、写入、连接接受）触发的处理程序（handlers）。
3. **阻塞行为**：
	- 函数执行时是阻塞的，意味着它会持续运行，直到所有异步操作完成或 `io_context` 被停止。
4. **确保线程安全**：
	- 所有异步操作的处理程序都在 `io_context.run()` 提供的上下文中安全地执行，这有助于维护线程安全。
5. **驱动程序的核心**：
	- `io_context.run()` 是 Boost.Asio 应用程序的驱动力，没有它，异步操作不会执行。
6. **处理所有异步事件**：
	- 包括网络 I/O 操作、定时器事件等，都是在 `io_context.run()` 的循环中被处理。
7. **应用程序的持续运行**：
	- 主线程在调用 `run` 后会在事件循环中阻塞，这保证了应用程序可以持续处理异步事件，直到不再需要处理或被显式停止。





# 隐患

该demo示例为仿照asio官网编写的，其中存在隐患，就是当服务器即将发送数据前(调用async_write前)，此刻客户端中断，服务器此时调用async_write会触发发送回调函数，判断ec为非0进而执行delete this逻辑回收session。但要注意的是客户端关闭后，在tcp层面会触发读就绪事件，服务器会触发读事件回调函数。在读事件回调函数中判断错误码ec为非0，进而再次执行delete操作，从而造成二次析构，这是极度危险的。





# 总结

这个demo介绍了异步读写的相关操作，下面是对于常用函数的总结，以及对异步的理解



## 常用函数

### **（a)开启异步读**

```cpp
void session::Start() {
    memset(_data, 0, max_length);
    _socket.async_read_some(boost::asio::buffer(_data, max_length),
        std::bind(&session::handle_read, this, placeholders::_1, placeholders::_2));
}
```

- 开启异步读就好，然后结束start





### **(b)异步读**async_read_some

```cpp
void session::Start() {
    memset(_data, 0, max_length);
    //指定读到哪，读多少
    _socket.async_read_some(boost::asio::buffer(_data, max_length),
                            //读完后回调，去写
        std::bind(&session::handle_read, this, placeholders::_1, placeholders::_2));
}

//回调函数处理异步写
void session::handle_read(const boost::system::error_code& error, size_t bytes_transferred) {
    if (!error) {
        cout << "server receive data is " << _data << endl;
        boost::asio::async_write(_socket, boost::asio::buffer(_data, bytes_transferred),
            std::bind(&session::handle_write, this, placeholders::_1));
    } else {
        delete this;
    }
}
```

- 开启异步读
- 参数（buffer，回调函数）
- buffer需要数组首地址，以及读取的**最大长度**(并不是读完max_lengtg才会回调！！)
- 回调函数需要两个参数error和bytes_transferred，因为不能保证数据一次性发完



### **(c)异步写async_write**

```cpp
void session::handle_read(const boost::system::error_code& error, size_t bytes_transferred) {
    if (!error) {
        cout << "server receive data is " << _data << endl;
        //异步发送数据给客户端，发送完回调
        boost::asio::async_write(_socket, boost::asio::buffer(_data, bytes_transferred),
                                 //回调，开启异步读
            std::bind(&session::handle_write, this, placeholders::_1));
    } else {
        delete this;
    }
}

void session::handle_write(const boost::system::error_code& error) {
    if (!error) {
        memset(_data, 0, max_length);
        //异步读
        _socket.async_read_some(boost::asio::buffer(_data, max_length),
            std::bind(&session::handle_read, this, placeholders::_1, placeholders::_2));
    } else {
        delete this;
    }
}
```

- 异步写函数在异步读的回调函数中
- 异步写发送数据给客户端，然后回调函数中调用异步读
- 参数（buffer，回调函数）
- buffer需要首地址，以及要发送的长度
- 回调函数只需要一个参数，那就是error，因为write一定会发送完的



### **(d)异步接受连接async_accept**

```cpp
void server::start_accept() {
    session* new_session = new session(_ioc);
    _acceptor.async_accept(new_session->Socket(),
                           //创建新连接后，回调
        std::bind(&server::handle_accept, this, new_session, placeholders::_1));
}

void server::handle_accept(session* new_session, const boost::system::error_code& error) {
    if (!error) {
        new_session->Start();
    } else {
        delete new_session;
    }
    start_accept();//接着等待接受新连接
}
```

- 创建一个新的 `session` 对象，准备处理新连接。
- 使用 `_acceptor.async_accept` 异步等待新的连接请求。
- 当有新的连接时，`handle_accept` 会被调用。



## (e)bind绑定回调函数的参数问题

>### Boost Asio 异步操作的回调函数
>
>Boost Asio 的异步操作，如 `async_read`、`async_write` 等，通常要求你提供一个回调函数。这个回调函数通常接受两个参数：
>
>**Error Code** (`boost::system::error_code`)：表明异步操作成功或失败的错误码。
>
>**Bytes Transferred** (`size_t`)：传输的字节数。





1. **绑定所有参数**

	```cpp
	std::bind(&YourClass::YourCallbackFunction, this, placeholders::_1, placeholders::_2)
	```

	

2. **绑定部分参数**

	```cpp
	std::bind(&YourClass::YourCallbackFunction, this, placeholders::_1)
	//只有错误码会被传递给回调函数。
	```

	

3. **不绑定任何参数**

	```cpp
	std::bind(&YourClass::YourCallbackFunction, this)
	```

	------







## 对异步的理解

- ### 异步操作的特点

	1. **事件驱动**：
		- 异步操作是基于事件的。这意味着操作（如读取、写入、接受连接等）等待特定的网络事件发生，如数据到达或连接建立。
	2. **非阻塞行为**：
		- 异步操作不会阻塞程序的其余部分。它们在后台“监听”或等待事件发生，而程序可以继续执行其他任务。
	3. **回调函数**：
		- 当相应的事件发生（例如，数据到达用于读取的套接字），异步操作完成，并触发定义好的回调函数。
		- 回调函数通常用于处理事件结果，如读取数据或发送响应。
	
	### 异步读取和写入的循环
	
	- 当 `async_read_some` 或类似函数被调用时，它开始监听数据到达事件。如果没有数据到达，这个函数不会执行其回调。
	- 在异步读取的回调函数中启动异步写入是一种常见的模式。这确保了服务器在处理完一个请求后立即准备发送响应。
	- 完成异步写入后，通常会再次启动异步读取操作，维持与客户端的持续通信。
	
	### 异步接受连接
	
	- `async_accept` 类似地监听新的连接请求。如果没有新的连接尝试，它会保持在监听状态，直到有新的连接请求到达。
	- 一旦接受到新的连接，将调用 `async_accept` 指定的回调函数来处理这个新连接。

