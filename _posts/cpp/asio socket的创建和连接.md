---
title: asio socket的创建和连接
date: 2023-11-11 15:52:40
tags: c++
categories: 
- 后端
- 网络编程
---



# 基础知识

### TCP和IP协议

1. **TCP（传输控制协议）**：TCP是一种网络通信协议，它在互联网协议套件（TCP/IP）中负责确保数据的可靠传输。TCP提供了一种可靠的、面向连接的通信方式。这意味着在数据开始传输之前，两个网络设备（比如计算机）之间必须首先建立一个连接。TCP还负责确保数据的完整性和顺序正确。
2. **IP协议（互联网协议）**：IP协议是另一种关键的网络通信协议，负责在网络上路由和传输数据包。IP有两个主要版本：IPv4和IPv6。IPv4是目前最广泛使用的版本，但由于地址空间限制，IPv6正在逐渐被采用。

### Socket

- **Socket是什么**：Socket是网络通信的端点。您可以将其想象为电话插座，它是网络上两个程序（运行在不同计算机上或同一计算机上的不同进程）进行数据交换的接口。
- **Socket的用途**：Socket使得程序可以读写网络上的数据。在创建Socket时，您需要指定使用的协议（如TCP）。一旦建立了Socket连接，数据就可以通过这个连接在网络上进行传输。

### TCP连接和IP协议的关系

- 在进行TCP网络通信时，通常会使用IP协议（无论是IPv4还是IPv6）。这是因为TCP负责在两个端点之间建立一个稳定的连接，并确保数据可靠地传输，而IP协议负责将数据包路由到正确的目的地。
- 当您在编程中创建一个TCP Socket时，您需要指定使用的IP协议版本。例如，在Boost.Asio中，您可以创建一个使用IPv4的TCP Socket（`asio::ip::tcp::socket`），这意味着它将使用IPv4协议来路由数据。



# 网络编程基本流程



## 服务端

1）socket——创建socket对象。

2）bind——绑定本机ip+port。

3）listen——监听来电，若在监听到来电，则建立起连接。

4）accept——再创建一个socket对象给其收发消息。原因是现实中服务端都是面对多个客户端，那么为了区分各个客户端，则每个客户端都需再分配一个socket对象进行收发消息。

5）read、write——就是收发消息了。



## 客户端

1）socket——创建socket对象。

2）connect——根据服务端ip+port，发起连接请求。

3）write、read——建立连接后，就可发收消息了。



# 终端节点的创建



## 客户端

`客户端`可以通过对端的ip和端口构造一个endpoint，用这个endpoint和`服务端`通信。

步骤如下：

1. 设置服务器`IP地址`（string）和 `PORT`（unsigned short）
2. 定义` boost::system::error_code`状态码，用于存储解析` IP`时候可能发生的错误
3. 解析IP地址,获取`asio::ip::address`对象
4. 如果解析失败，反悔错误码
5. 解析成功，根据`ip::address`和`PORT`建立TCP连接，获取`asio::ip::tcp::endpoint`对象

```cpp
#include "endpoint.h"
#include <iostream>
#include <boost/asio.hpp>
#include <boost/system/error_code.hpp>

// 使用命名空间别名，简化Boost.Asio的使用
namespace asio = boost::asio;

// 定义client_end_point函数
int client_end_point() {
    // Step 1: 设置服务器IP地址和端口号
    // 使用本地回环地址127.0.0.1进行测试
    std::string raw_ip_address = "127.0.0.1"; 
    unsigned short port_num = 3333;           

    // Step 2: 用于存储解析IP地址时可能发生的错误的变量
    boost::system::error_code ec;

    // Step 3: 解析IP地址
    asio::ip::address ip_address = asio::ip::address::from_string(raw_ip_address, ec);

    // Step 4: 检查IP地址是否正确解析
    if (ec.value() != 0) {
        // 如果解析出错，输出错误信息并返回错误码
        std::cout << "Failed to parse the IP address. Error code = "
                  << ec.value() << ". Message: " << ec.message();
        return ec.value();
    }

    // Step 5: 使用解析好的IP地址和端口号创建TCP端点
    asio::ip::tcp::endpoint ep(ip_address, port_num);

    // Step 6: 输出端点信息，验证创建是否成功
    std::cout << "Endpoint created: IP Address = " << ep.address() 
              << ", Port = " << ep.port() << std::endl;

    // 如果一切正常，返回0
    return 0;
}

// 主函数
int main() {
    // 调用client_end_point函数并处理返回结果
    int result = client_end_point();

    // 检查client_end_point函数的返回值，确定是否出错
    if (result != 0) {
        std::cerr << "Error occurred during endpoint creation. Error code: " << result << std::endl;
        return result;
    }

    // 端点创建成功
    std::cout << "Endpoint creation successful." << std::endl;

    // 正常结束程序
    return 0;
}
```

------



## 服务端

步骤如下：

1. 定义端口号（unsigned short）
2. 根据本机IPv6`asio::ip::address_v6::any()`协议创建`asio::ip::address`对象
3. 根据`ip::address`和`Port`创建`asio::ip::tcp::endpoint`对象

```cpp
#include "endpoint.h"
#include <iostream>
#include <boost/asio.hpp>
#include <boost/system/error_code.hpp>

// 使用命名空间别名，简化Boost.Asio的使用
namespace asio = boost::asio;

// 定义server_end_point函数
int server_end_point() {
    // Step 1: 获取协议端口号
    // 这里我们假设服务器应用程序已经获取了端口号
    unsigned short port_num = 3333;

    // Step 2: 创建一个特殊的asio::ip::address对象
    // 它指定了主机上所有可用的IP地址
    // 注意，这里我们假设服务器工作在IPv6协议上
    asio::ip::address ip_address = asio::ip::address_v6::any();

    // Step 3: 使用指定的IP地址和端口号创建TCP端点
    asio::ip::tcp::endpoint ep(ip_address, port_num);

    // Step 4: 端点创建完成，可以用来指定服务器应用程序希望监听传入连接的IP地址和端口号
    std::cout << "Server endpoint created: IP Address = " << ep.address() 
              << ", Port = " << ep.port() << std::endl;

    // 如果一切正常，返回0
    return 0;
}

// 主函数
int main() {
    // 调用server_end_point函数并处理返回结果
    int result = server_end_point();

    // 检查server_end_point函数的返回值，确定是否出错
    if (result != 0) {
        std::cerr << "Error occurred during server endpoint creation. Error code: " << result << std::endl;
        return result;
    }

    // 服务器端点创建成功
    std::cout << "Server endpoint creation successful." << std::endl;

    // 正常结束程序
    return 0;
}
```

------

# Socket

## 客户端

步骤：

1. 创建`asio::io_context`对象
2. 创建一个代表TCP协议的`asio::ip::tcp`对象protocol
3. 创建TCP套接字对象`asio::ip::tcp::socket`,将它与`asio::io_context`对象关联
4. 定义一个`boost::system::error_code`变量
5. 打开套接字（使用TCP协议protocol，错误码error_code)

```cpp
#include <iostream>
#include <boost/asio.hpp>
#include <boost/system/error_code.hpp>

// 使用命名空间别名，简化Boost.Asio的使用
namespace asio = boost::asio;

// 定义create_tcp_socket函数
int create_tcp_socket() {
    // Step 1: 创建asio::io_context对象
    // io_service（或io_context）是Boost.Asio中的核心类，用于处理所有I/O操作
    asio::io_context ios;

    // Step 2: 创建一个代表TCP协议的asio::ip::tcp对象
    // 这里我们使用IPv4作为底层协议
    asio::ip::tcp protocol = asio::ip::tcp::v4();

    // Step 3: 实例化一个活动的TCP套接字对象
    // 套接字用于网络通信，这里我们将它与io_service对象关联
    asio::ip::tcp::socket sock(ios);

    // Step 4: 定义一个error_code变量，用于存储打开套接字时可能发生的错误
    boost::system::error_code ec;

    // Step 5: 打开套接字
    // 使用之前定义的TCP协议（IPv4）打开套接字
    sock.open(protocol, ec);

    // Step 6: 检查套接字是否成功打开
    if (ec.value() != 0) {
        // 如果打开失败，输出错误信息并返回错误码
        std::cout << "Failed to open the socket! Error code = "
                  << ec.value() << ". Message: " << ec.message();
        return ec.value();
    }

    // 如果一切正常，返回0
    return 0;
}

// 主函数
int main() {
    // 调用create_tcp_socket函数并处理返回结果
    int result = create_tcp_socket();

    // 检查create_tcp_socket函数的返回值，确定是否出错
    if (result != 0) {
        std::cerr << "Error occurred during socket creation. Error code: " << result << std::endl;
        return result;
    }

    // 套接字创建成功
    std::cout << "Socket creation successful." << std::endl;

    // 正常结束程序
    return 0;
}
```

------



## 服务端

需要生成一个acceptor的socket，用来接收新的连接。

步骤：

1. 创建`asio::io_context`对象
2. 创建一个代表TCP协议的`asio::ip::tcp`对象protocol
3. 实例化一个接收器（Acceptor） 套接字对象`asio::ip::tcp::acceptor`,参数是ios
4. 定义一个`boost::system::error_code`变量
5. 打开套接字（使用TCP协议protocol，错误码error_code)

```cpp
#include <iostream>
#include <boost/asio.hpp>
#include <boost/system/error_code.hpp>

// 使用命名空间别名，简化Boost.Asio的使用
namespace asio = boost::asio;

// 定义create_acceptor_socket函数
int create_acceptor_socket() {
    // Step 1: 创建asio::io_context对象
    // io_service（或io_context）是Boost.Asio中的核心类，用于处理所有I/O操作
    asio::io_context ios;

    // Step 2: 创建一个代表TCP协议的asio::ip::tcp对象
    // 这里我们使用IPv6作为底层协议
    asio::ip::tcp protocol = asio::ip::tcp::v6();

    // Step 3: 实例化一个接收器（Acceptor）套接字对象
    // 接收器套接字用于监听和接受传入的TCP连接
    asio::ip::tcp::acceptor acceptor(ios);

    // Step 4: 定义一个error_code变量，用于存储打开接收器套接字时可能发生的错误
    boost::system::error_code ec;

    // Step 5: 打开接收器套接字
    // 使用之前定义的TCP协议（IPv6）打开接收器
    acceptor.open(protocol, ec);

    // Step 6: 检查接收器套接字是否成功打开
    if (ec.value() != 0) {
        // 如果打开失败，输出错误信息并返回错误码
        std::cout << "Failed to open the acceptor socket! Error code = "
                  << ec.value() << ". Message: " << ec.message();
        return ec.value();
    }

    // 如果一切正常，返回0
    return 0;
}

// 主函数
int main() {
    // 调用create_acceptor_socket函数并处理返回结果
    int result = create_acceptor_socket();

    // 检查create_acceptor_socket函数的返回值，确定是否出错
    if (result != 0) {
        std::cerr << "Error occurred during acceptor socket creation. Error code: " << result << std::endl;
        return result;
    }

    // 接收器套接字创建成功
    std::cout << "Acceptor socket creation successful." << std::endl;

    // 正常结束程序
    return 0;
}
```



------

# 绑定acceptor

acceptor类型的socket，服务器要将其绑定到指定的端点,所有连接这个端点的连接都可以被接收到

步骤：

1. 设置端口号port
2. 创建一个端点 `asio::ip::tcp::endpoint`,利用IP，prot
3. 创建并打开一个接收器套接字`asio::ip::tcp::acceptor`，根据`ios`和`协议`
4. 定义一个error_code变量
5.  将接收器套接字（accecptor）绑定到端点（endpoint）， `acceptor.bind(ep, ec)`



```cpp
#include <iostream>
#include <boost/asio.hpp>
#include <boost/system/error_code.hpp>

// 使用命名空间别名，简化Boost.Asio的使用
namespace asio = boost::asio;

// 定义bind_acceptor_socket函数
int bind_acceptor_socket() {
    // Step 1: 获取协议端口号
    // 这里我们假设服务器应用程序已经获取了端口号
    unsigned short port_num = 3333;

    // Step 2: 创建一个端点
    // 使用asio::ip::address_v4::any()表示服务器将接受发送到本机任何IPv4地址的连接
    asio::ip::tcp::endpoint ep(asio::ip::address_v4::any(), port_num);

    // Step 3: 创建并打开一个接收器套接字
    asio::io_context ios;
    asio::ip::tcp::acceptor acceptor(ios, ep.protocol());

    // Step 4: 定义一个error_code变量，用于存储绑定操作过程中可能发生的错误
    boost::system::error_code ec;

    // Step 5: 将接收器套接字绑定到端点
    acceptor.bind(ep, ec);

    // Step 6: 检查接收器套接字是否成功绑定
    if (ec.value() != 0) {
        // 如果绑定失败，输出错误信息并返回错误码
        std::cout << "Failed to bind the acceptor socket. Error code = "
                  << ec.value() << ". Message: " << ec.message();
        return ec.value();
    }

    // 如果一切正常，返回0
    return 0;
}

// 主函数
int main() {
    // 调用bind_acceptor_socket函数并处理返回结果
    int result = bind_acceptor_socket();

    // 检查bind_acceptor_socket函数的返回值，确定是否出错
    if (result != 0) {
        std::cerr << "Error occurred during acceptor socket binding. Error code: " << result << std::endl;
        return result;
    }

    // 接收器套接字绑定成功
    std::cout << "Acceptor socket binding successful." << std::endl;

    // 正常结束程序
    return 0;
}
```



------



# 客户端连接到指定的端点

步骤：

1.  获取目标服务器的IP地址和端口号
2. 创建一个指向目标应用程序的端点endpoint，需要ip和端口号
3. 创建并打开一个套接字socket，需要ios和IP协议
4. socket连接endpoint

```cpp
#include <iostream>
#include <boost/asio.hpp>
#include <boost/system/error_code.hpp>

// 使用命名空间别名，简化Boost.Asio的使用
namespace asio = boost::asio;

// 定义connect_to_end函数
int connect_to_end() {
    // Step 1: 获取目标服务器的IP地址和端口号
    std::string raw_ip_address = "127.0.0.1";
    unsigned short port_num = 3333;

    try {
        // Step 2: 创建一个指向目标服务器应用程序的端点
        asio::ip::tcp::endpoint ep(asio::ip::address::from_string(raw_ip_address), port_num);

        asio::io_context ios;

        // Step 3: 创建并打开一个套接字
        asio::ip::tcp::socket sock(ios, ep.protocol());

        // Step 4: 连接套接字
        sock.connect(ep);

        // 连接成功，套接字现在可以用于发送和接收数据
    }
    catch (boost::system::system_error& e) {
        std::cout << "Error occurred! Error code = " << e.code()
                  << ". Message: " << e.what();
        return e.code().value();
    }
    return 0;
}

// 主函数
int main() {
    // 调用connect_to_end函数并处理返回结果
    int result = connect_to_end();

    // 检查connect_to_end函数的返回值，确定是否出错
    if (result != 0) {
        std::cerr << "Error occurred during connection. Error code: " << result << std::endl;
        return result;
    }

    // 连接成功
    std::cout << "Connection successful." << std::endl;

    // 正常结束程序
    return 0;
}
```





# 服务端接收连接

步骤：

1. 设置端口号
2. 创建端点endpoint，需要IP地址和Port
3. 创建`接收器套接字`，需要ios和IP协议
4. 接收器套接字`accecptor`绑定指定`endpoint`
5. 开始监听传入的连接请求 `acceptor.listen(BACKLOG_SIZE)`
6.  创建一个活动套接字sock,需要ios
7. 接收连接请求，将活动套接字连接到客户端`acceptor.accept(sock)`

```cpp
#include <iostream>
#include <boost/asio.hpp>
#include <boost/system/error_code.hpp>

// 使用命名空间别名，简化Boost.Asio的使用
namespace asio = boost::asio;

// 定义accept_new_connection函数
int accept_new_connection() {
    const int BACKLOG_SIZE = 30; // 等待连接请求的队列大小

    // Step 1: 获取协议端口号
    unsigned short port_num = 3333;

    // Step 2: 创建一个服务器端点
    asio::ip::tcp::endpoint ep(asio::ip::address_v4::any(), port_num);

    asio::io_context ios;

    try {
        // Step 3: 实例化并打开一个接收器套接字
        asio::ip::tcp::acceptor acceptor(ios, ep.protocol());

        // Step 4: 将接收器套接字绑定到服务器端点
        acceptor.bind(ep);

        // Step 5: 开始监听传入的连接请求
        acceptor.listen(BACKLOG_SIZE);

        // Step 6: 创建一个活动套接字
        asio::ip::tcp::socket sock(ios);

        // Step 7: 接收连接请求，将活动套接字连接到客户端
        acceptor.accept(sock);

        // 连接成功，套接字现在可以用于与客户端进行数据交换
    }
    catch (boost::system::system_error& e) {
        std::cout << "Error occurred! Error code = " << e.code()
                  << ". Message: " << e.what();
        return e.code().value();
    }
    return 0;
}

// 主函数
int main() {
    // 调用accept_new_connection函数并处理返回结果
    int result = accept_new_connection();

    // 检查accept_new_connection函数的返回值，确定是否出错
    if (result != 0) {
        std::cerr << "Error occurred during accepting connection. Error code: " << result << std::endl;
        return result;
    }

    // 接收连接成功
    std::cout << "Accepting connection successful." << std::endl;

    // 正常结束程序
    return 0;
}
```







# 概念和疑问

## 概念



- [ ] 端点，套接字，接收器套接字对于客户端和服务端的作用

- **客户端端点创建** (`client_end_point`):
	- 设置目标服务器的IP和端口，创建客户端端点以连接服务器。
- **服务器端点创建** (`server_end_point`):
	- 设置端口号，创建服务器端点以便监听来自客户端的连接。
- **创建TCP套接字** (`create_tcp_socket`):
	- 创建套接字，用于客户端的连接或服务器的通信。
- **创建接收器套接字** (`create_acceptor_socket`):
	- 创建接收器套接字，用于服务器端接受客户端的连接请求。
- **绑定接收器套接字** (`bind_acceptor_socket`):
	- 将接收器套接字绑定到一个特定的端点，用于监听来自该端点的连接请求。
- **连接到端点** (`connect_to_end`):
	- 客户端使用套接字连接到指定的服务器端点。
- **接收新连接** (`accept_new_connection`):
	- 服务端监听并接受来自客户端的新连接请求。**创建一个新的套接字与客户端进行通信**。（注意，创建新的套接字通信）



- [ ] 端点，套接字，接收器套接字代码

	

| 组件                      | 功能                       | 参数                               | 示例代码                                                     | 应用场景                                                  |
| ------------------------- | -------------------------- | ---------------------------------- | ------------------------------------------------------------ | --------------------------------------------------------- |
| boost::system::error_code | 错误代码对象               | -                                  | boost::system::error_code ec;                                | 用于处理网络操作中的错误，如套接字打开或绑定失败。        |
| asio::ip::address         | 表示一个IP地址             | 1.IP地址字符串<br />2.错误代码对象 | asio::ip::address ip_address = asio::ip::address::from_string("127.0.0.1", ec); | 定义网络通信的IP地址，用于创建端点。                      |
| asio::ip::tcp::endpoint   | 表示一个TCP网络端点        | 1.ip_address<br />2.port_num       | asio::ip::tcp::endpoint ep(ip_address, port_num);            | 用于定义TCP连接的目标地址和端口，客户端和服务器都会使用。 |
| asio::io_context          | 提供I/O功能                | -                                  | asio::io_context ios;                                        | 用于管理异步I/O操作，是Boost.Asio程序的核心。             |
| asio::ip::tcp::socket     | TCP套接字，用于网络通信    | 1.`io_context`对象<br />2.协议     | asio::ip::tcp::socket sock(ios);                             | 用于客户端发起连接和服务器与客户端之间的数据交换。        |
| asio::ip::tcp::acceptor   | 接收器套接字，用于接受连接 | 1.`io_context`对象<br />2.协议     | asio::ip::tcp::acceptor acceptor(ios, protocol);             | 服务器端使用，用于监听和接受来自客户端的连接。            |



## 疑问



#### 1. 关于地址解析

**问题**：根据string类型的ip地址解析，创建address对象，如果这个ip不是本机ip，是别的服务器的ip，就有可能解析不成功。如果是本机ip，一定可以成功。

**答案**：当你使用一个字符串类型的IP地址来创建`asio::ip::address`对象时，如果IP地址格式正确，它应该能够成功解析，无论这个IP是否属于本机。解析失败通常是由于格式错误或不支持的地址类型。



#### 2. 创建端点endpoint对象需要address对象和端口

**问题**：创建端点endpoint对象需要address对象和端口。

**答案**：正确，`asio::ip::tcp::endpoint`对象的创建需要一个IP地址（`asio::ip::address`对象）和一个端口号。



#### 3. 关于创建和打开`socket`

**问题**：创建socket对象需要ios，打开socket需要protocol和error_code?为啥需要打开呢？打开的时候在什么情况下会出错呢？

**答案**：`asio::ip::tcp::socket`对象需要一个`asio::io_context`对象来处理I/O操作。打开`socket`指的是使其准备好进行网络通信。如果指定的协议不支持，或者系统资源不足（如端口号被占用），打开`socket`可能会出错。



#### 4. 关于`acceptor`

**问题**：accecptor也是一个接收器socket，也需要打开，并且流程跟socket一样？

**答案**：`asio::ip::tcp::acceptor`类似于一个特殊的`socket`，专门用于监听和接受来自客户端的连接。它也需要被打开和绑定到一个端点上，以便知道在哪个端口上监听。



#### 5. 绑定`acceptor`

**问题**：绑定accecptor，是先创建，然后在将他绑定到endpoint上么？然后listen，就可以接受ip：port的消息了？

**答案**：你需要先创建一个`acceptor`，然后将其绑定到一个`endpoint`上（一个IP地址和端口号的组合）。一旦绑定并调用了`listen()`，它就开始监听该端点上的入站连接。



#### 6. 客户端访问服务端

**问题**：客户端访问服务端，需要socket绑定endpoint（服务器的ip:port）。

**答案**：客户端的`socket`需要连接到服务器的`endpoint`（服务器的IP地址和端口号）。



#### 7. 关于`asio::ip::address_v6::any()`

**问题**：asio::ip::address_v6::any()，这种代码只会在服务器上创建endpoint上使用，而且基本不会出错？

**答案**：`asio::ip::address_v6::any()`在服务器上创建一个可以接受任何IPv6地址的端点，这意味着它可以从任何IPv6地址接收连接。这通常不会出错，除非有底层网络配置问题。



#### 8. 关于接收器端点的访问

**问题**：接收器接受服务器的端点，此时是不是只用（端点的ip，和端口号）访问？比如开放了公网ip，那只可以用“公网ip:端口”访问，如果用“127.0.0.1：3333”就不可以（如果服务器在本地）。

**答案**：如果服务器监听公网IP和端口，那么客户端必须使用这个公网IP和端口来连接。如果服务器监听的是本地地址（如127.0.0.1），那么只有本地客户端（在同一台机器上）可以使用这个地址和端口连接。



#### 9. 理解`asio::io_context`的重要性

**问题**：如果不理解asio::io_context，是不是可以先跳过？

**答案**：`asio::io_context`是Boost Asio库的核心，负责管理所有I/O服务。虽然一开始可能不容易理解，但它在Asio编程中非常重要，建议不要跳过。随着实践的增加，你会逐渐理解它的用途和重要性。

