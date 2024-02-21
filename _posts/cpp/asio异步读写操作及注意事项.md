---
title: asio异步读写操作及注意事项
date: 2023-11-12 22:22:10
tags: c++
categories: 
- 后端
- 网络编程
---





# 简介

本文介绍异步读写操作。


定义个session类，这个session类表示服务器处理客户端连接的管理类

其中有socket对象。

```cpp
class Session {
public:
    Session(std::shared_ptr<asio::ip::tcp::socket> socket);//构造函数
    void Connect(const asio::ip::tcp::endpoint& ep);//连接端点
private:
    std::shared_ptr<asio::ip::tcp::socket> _socket;//智能指针
};
```

session类定义了一个socket成员变量，负责处理对端的连接读写，封装了Connect函数

```cpp
void Session::Connect(const asio::ip::tcp::endpoint &ep) {
    _socket->connect(ep);
}
```





# 异步写操作

在写操作前，我们先封装一个MsgNode结构，用来管理要`发送和接收`的数据，该结构包含`数据域首地址`，`数据的总长度`，以及`已经处理的长度`(已读的长度或者已写的长度)



## async_write_some方式

> 一次性不一定发送完数据

### 定义MsgNode类用于封装数据

```cpp
//最大报文接收大小
const int RECVSIZE = 1024;
class  MsgNode {
public :
    //发送数据的构造函数
    MsgNode(const char* msg,  int total_len): _total_len(total_len), _cur_len(0){
        _msg = new char[total_len];
        memcpy(_msg, msg, total_len);
    }
    //接受数据的构造函数
    MsgNode(int total_len) :_total_len(total_len), _cur_len(0) {
        _msg = new char[total_len];
    }
    ~MsgNode(){
        delete[]_msg;
    }
    //消息首地址
    char* _msg;
    //总长度
    int _total_len;
    //当前长度
    int _cur_len;
};
```



### 为Session添加异步写操作

```cpp
class Session{
public:
    //回调函数
    void WriteCallBackErr(const boost::system::error_code & ec, std::size_t bytes_transferred,
    std::shared_ptr<MsgNode>);
    //写函数（传入需要写的数据）
    void WriteToSocketErr(const std::string& buf);
private:
    std::shared_ptr<MsgNode> _send_node;//定义需要发送的数据
};
```



### 实现`void WriteToSocketErr函数`

```cpp
void Session::WriteToSocketErr(const std::string& buf) {
    //构造发送数据
    _send_node = make_shared<MsgNode>(buf.c_str(), buf.length());
    //异步发送数据，因为异步所以不会一下发送完
    this->_socket->async_write_some(asio::buffer(_send_node->_msg, 
        _send_node->_total_len),
        std::bind(&Session::WriteCallBackErr,
            this, std::placeholders::_1, std::placeholders::_2, _send_node));
}
```

> async_wirte_some函数第一个参数是buffer，第二个参数是回调函数）



### 实现`WriteCallBackErr`函数

因为WriteCallBackErr函数为三个参数且为成员函数，而**async_write_some需要的回调函数为两个参数**，所以我们通过bind将三个参数转换为两个参数的普通函数。

```cpp
void Session::WriteCallBackErr(const boost::system::error_code& ec, 
    std::size_t bytes_transferred, std::shared_ptr<MsgNode> msg_node) {
    if (bytes_transferred + msg_node->_cur_len 
        < msg_node->_total_len) {
        _send_node->_cur_len += bytes_transferred;
        this->_socket->async_write_some(asio::buffer(_send_node->_msg+_send_node->_cur_len,
            _send_node->_total_len-_send_node->_cur_len),
            std::bind(&Session::WriteCallBackErr,
                this, std::placeholders::_1, std::placeholders::_2, _send_node));
    }
}
```





> 单凭这两个不可以投入使用，具体原因是



我们可以通过队列保证应用层的发送顺序。我们在Session中定义一个发送队列，然后重新定义正确的异步发送函数和回调处理



### 定义queue写入消息（重点）

```cpp
class Session{
public:
    void WriteCallBack(const boost::system::error_code& ec, std::size_t bytes_transferred);
    void WriteToSocket(const std::string &buf);
private:
    std::queue<std::shared_ptr<MsgNode>> _send_queue;//用来缓存要发送的消息节点
    std::shared_ptr<asio::ip::tcp::socket> _socket;
    bool _send_pending;//该变量为true表示一个节点还未发送完。
};
```



### **实现queue异步写入功能**

```cpp
void Session::WriteToSocket(const std::string& buf){
    //插入发送队列
    _send_queue.emplace(new MsgNode(buf.c_str(), buf.length()));
    //pending状态说明上一次有未发送完的数据
    if (_send_pending) {
        return;
    }
    //异步发送数据，因为异步所以不会一下发送完
    this->_socket->async_write_some(asio::buffer(buf), std::bind(&Session::WriteCallBack, this, std::placeholders::_1, std::placeholders::_2));
    _send_pending = true;//标志正在发送消息...
}

//回调函数
void Session::WriteCallBack(const boost::system::error_code & ec,  std::size_t bytes_transferred){
    if (ec.value() != 0) {
        std::cout << "Error , code is " << ec.value() << " . Message is " << ec.message();
        return;
    }
    //取出队首元素即当前未发送完数据
    auto & send_data = _send_queue.front();
    send_data->_cur_len += bytes_transferred;
    
    //数据未发送完， 则继续发送（相当于递归）
    if (send_data->_cur_len < send_data->_total_len) {
        this->_socket->async_write_some(asio::buffer(send_data->_msg + send_data->_cur_len, send_data->_total_len-send_data->_cur_len),
            std::bind(&Session::WriteCallBack,
            this, std::placeholders::_1, std::placeholders::_2));
        return;
    }
    
    //如果发送完，则pop出队首元素
    _send_queue.pop();
    //如果队列为空，则说明所有数据都发送完,将pending设置为false
    if (_send_queue.empty()) {
        _send_pending = false;
    }
    
    //如果队列不是空，则继续将队首元素发送
    if (!_send_queue.empty()) {
        auto& send_data = _send_queue.front();
        this->_socket->async_write_some(asio::buffer(send_data->_msg + send_data->_cur_len, send_data->_total_len - send_data->_cur_len),
            std::bind(&Session::WriteCallBack,
                this, std::placeholders::_1, std::placeholders::_2));
    }
    _send_pending=false;//消息发送完毕
}
```



## async_send方式

> 一次性发送完数据

其内部的实现原理就是帮我们不断的调用async_write_some直到完成发送，`async_send不能和async_write_some混合使用`，我们基于async_send封装另外一个发送函数

```cpp
//不能与async_write_some混合使用
void Session::WriteAllToSocket(const std::string& buf) {
    //插入发送队列
    _send_queue.emplace(new MsgNode(buf.c_str(), buf.length()));
    //pending状态说明上一次有未发送完的数据
    if (_send_pending) {
        return;
    }
    //异步发送数据，因为异步所以不会一下发送完
    this->_socket->async_send(asio::buffer(buf), 
        std::bind(&Session::WriteAllCallBack, this,
            std::placeholders::_1, std::placeholders::_2));
    _send_pending = true;
}
void Session::WriteAllCallBack(const boost::system::error_code& ec, std::size_t bytes_transferred){
    if (ec.value() != 0) {
        std::cout << "Error occured! Error code = "
            << ec.value()
            << ". Message: " << ec.message();
        return;
    }
    //如果发送完，此时一定发送完，则pop出队首元素
    _send_queue.pop();
    //如果队列为空，则说明所有数据都发送完,将pending设置为false
    if (_send_queue.empty()) {
        _send_pending = false;
    }
    //如果队列不是空，则继续将队首元素发送
    if (!_send_queue.empty()) {
        auto& send_data = _send_queue.front();
        this->_socket->async_send(asio::buffer(send_data->_msg + send_data->_cur_len, send_data->_total_len - send_data->_cur_len),
            std::bind(&Session::WriteAllCallBack,
                this, std::placeholders::_1, std::placeholders::_2));
    }
}
```





#  异步读操作





## async_read_some方式

> 触发的回调函数获取的读数据的长度可能会小于要求读取的总长度

```cpp
class Session {
public:
    void ReadFromSocket();
    void ReadCallBack(const boost::system::error_code& ec, std::size_t bytes_transferred);
private:
    std::shared_ptr<asio::ip::tcp::socket> _socket;
    std::shared_ptr<MsgNode> _recv_node;
    bool _recv_pending;
};
```

`recv_node用来缓存接收的数据，_recv_pending为true表示节点正在接收数据，还未接受完。`





**具体实现**

```cpp
//不考虑粘包情况， 先用固定的字节接收
void Session::ReadFromSocket() {
    if (_recv_pending) {
        return;
    }
    //可以调用构造函数直接构造，但不可用已经构造好的智能指针赋值
    /*auto _recv_nodez = std::make_unique<MsgNode>(RECVSIZE);
    _recv_node = _recv_nodez;*/
    
    _recv_node = std::make_shared<MsgNode>(RECVSIZE);
    _socket->async_read_some(asio::buffer(_recv_node->_msg, _recv_node->_total_len), std::bind(&Session::ReadCallBack, this,
        std::placeholders::_1, std::placeholders::_2));
    _recv_pending = true;
}
void Session::ReadCallBack(const boost::system::error_code& ec, std::size_t bytes_transferred){
    _recv_node->_cur_len += bytes_transferred;
    //没读完继续读
    if (_recv_node->_cur_len < _recv_node->_total_len) {
        _socket->async_read_some(asio::buffer(_recv_node->_msg+_recv_node->_cur_len,
            _recv_node->_total_len - _recv_node->_cur_len), std::bind(&Session::ReadCallBack, this,
            std::placeholders::_1, std::placeholders::_2));
        return;
    }
    //将数据投递到队列里交给逻辑线程处理，此处略去
    //如果读完了则将标记置为false
    _recv_pending = false;
    //指针置空
    _recv_node = nullptr;    
}
```



## async_receive方式



```cpp
void Session::ReadAllFromSocket(const std::string& buf) {
    if (_recv_pending) {
        return;
    }
    //可以调用构造函数直接构造，但不可用已经构造好的智能指针赋值
    /*auto _recv_nodez = std::make_unique<MsgNode>(RECVSIZE);
    _recv_node = _recv_nodez;*/
    _recv_node = std::make_shared<MsgNode>(RECVSIZE);
    _socket->async_receive(asio::buffer(_recv_node->_msg, _recv_node->_total_len), std::bind(&Session::ReadAllCallBack, this,
        std::placeholders::_1, std::placeholders::_2));
    _recv_pending = true;
}
void Session::ReadAllCallBack(const boost::system::error_code& ec, std::size_t bytes_transferred) {
    _recv_node->_cur_len += bytes_transferred;
    //将数据投递到队列里交给逻辑线程处理，此处略去
    //如果读完了则将标记置为false
    _recv_pending = false;
    //指针置空
    _recv_node = nullptr;
}
```





同样，async_read_some和async_receive不能混合使用，否则会出现逻辑问题。



# 总结



## 流程

（a）对于异步写操作，流程如下

1. 设置队列
2. 将需要写的放入队列里
3. 如果此时在写，则return
4. 否则开启写操作，绑定回调函数,然后标志”正在写“
5. 回调函数实现递归逻辑，按照顺序完成队列中所有的写操作



（b）对于异步读操作，流程如下

1. 如果此时在读，则return
2. 否则开启读操作，绑定回调函数，然后标志”正在读“
3. 回调函数实现递归逻辑，读完为止





## **注意**

对于写操作，尽量使用async_send
对于读操作，尽量使用async_read_some



## 重点

```cpp
//async_write_some

void Session::WriteToSocketErr(const std::string& buf) {
    // 构造发送数据
    _send_node = make_shared<MsgNode>(buf.c_str(), buf.length());
    // 异步发送数据
    this->_socket->async_write_some(asio::buffer(_send_node->_msg, _send_node->_total_len),
        std::bind(&Session::WriteCallBackErr,
            this, std::placeholders::_1, std::placeholders::_2, _send_node));
}

//async_send

void Session::WriteAllToSocket(const std::string& buf) {
    // 构造发送数据
    _send_queue.emplace(new MsgNode(buf.c_str(), buf.length()));
    // 异步发送数据
    this->_socket->async_send(asio::buffer(buf), //注意，这个一定是直接发完了，所以不需要指定长度******
        std::bind(&Session::WriteAllCallBack, this,
            std::placeholders::_1, std::placeholders::_2));
}

//async_read_some

void Session::ReadFromSocket() {
    _recv_node = std::make_shared<MsgNode>(RECVSIZE);
    // 异步读取数据
    _socket->async_read_some(asio::buffer(_recv_node->_msg, _recv_node->_total_len), 
        std::bind(&Session::ReadCallBack, this,
        std::placeholders::_1, std::placeholders::_2));
}

//async_receive

void Session::ReadAllFromSocket(const std::string& buf) {
    _recv_node = std::make_shared<MsgNode>(RECVSIZE);
    // 异步读取数据												必须指定大小，表示接受多少数据
    _socket->async_receive(asio::buffer(_recv_node->_msg, _recv_node->_total_len),
        std::bind(&Session::ReadAllCallBack, this,
        std::placeholders::_1, std::placeholders::_2));
}

```

