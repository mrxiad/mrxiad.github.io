---
title: asio健壮的异步服务器
date: 2023-11-13 17:25:15
tags: c++
categories: 网络编程
---







------

# 引言

asio异步应答服务器存在隐患，就是因为使用了delete删除session对象，而本章使用智能指针防止重复delete对象session，使用智能指针构造成一个伪闭包的状态延长session的生命周期。



# 智能指针管理Session

我们可以通过智能指针的方式管理Session类，将`acceptor`接收的链接保存在`Session类型`的`智能指针`里。由于智能指针会在引用计数为0时自动析构，所以为了防止其被自动回收，也方便Server管理Session。因为我们后期会做一些重连踢人等业务逻辑，我们在Server类中添加成员变量，该变量为一个map类型，`key为Session的uid`，`value为该Session的智能指针`,此时也会增加引用计数



# session类



## session类定义

- 首先定义一个消息节点，表示接受或者发送的消息
- 定义session类，继承std::enable_shared_from_this\<CSession>
- session存储socket，server，消息队列，uuid
- 回调函数中多一个参数，shared_ptr\<CSession> _self_shared，传入\_self\_shared保证引用计数+1，从而防止隐患

​	

```cpp
#pragma once
#include <boost/asio.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <queue>
#include <mutex>
#include <memory>
#include <iostream>
using namespace std;
#define MAX_LENGTH  1024
using boost::asio::ip::tcp;
class CServer;


//定义消息节点
class MsgNode
{
	friend class CSession;
public:
	MsgNode(char* msg, int max_len) {
		_data = new char[max_len];
		memcpy(_data, msg, max_len);
	}

	~MsgNode() {
		delete[] _data;
	}

private:
	int _cur_len;//当前发送（接受）消息长度
	int _max_len;//最大消息长度
	char* _data;//消息内容
};

//定义会话类，继承自enable_shared_from_this，用于在异步操作中获取this指针
class CSession :public std::enable_shared_from_this<CSession>
{
public:
    //构造函数只需要io_context，不需要ip、端口，因为是服务器方，accecptor才会有ip和端口
	CSession(boost::asio::io_context& io_context, CServer* server);
    
	~CSession() {
		std::cout << "Ssession destruct" << endl;
	}
    
	tcp::socket& GetSocket();
    
	std::string& GetUuid();
    
	void Start();//开启监听状态
  
	void Send(char* msg, int max_length);//开启写状态
private:
    //读回调，注意最后一个参数
	void HandleRead(const boost::system::error_code& error, size_t  bytes_transferred, shared_ptr<CSession> _self_shared);
    
    //写回调，不需要指定bytes_transferred，注意最后一个参数
	void HandleWrite(const boost::system::error_code& error, shared_ptr<CSession> _self_shared);
	
	
	//会话的socket
	tcp::socket _socket;
	//每个会话都有一个唯一的uuid
	std::string _uuid;

	//接收数据的缓冲区
	char _data[MAX_LENGTH];

	//对应的服务器
	CServer* _server;

	//发送队列
	std::queue<shared_ptr<MsgNode> > _send_que;

	//发送锁，防止多线程同时发送
	std::mutex _send_lock;
};
```



## session实现

```cpp
#include "CSession.h"
#include "CServer.h"
#include <iostream>

//构造函数，初始化socket对象，初始化server，同时初始化随机数uuid，用于标识每个连接
CSession::CSession(boost::asio::io_context& io_context, CServer* server) :
	_socket(io_context), _server(server) {
	boost::uuids::uuid  a_uuid = boost::uuids::random_generator()();
	_uuid = boost::uuids::to_string(a_uuid);
}

tcp::socket& CSession::GetSocket() {
	return _socket;
}

std::string& CSession::GetUuid() {
	return _uuid;
}



//开始监听读取
void CSession::Start() {
    //注意绑定的是shared_from_this()！！
	memset(_data, 0, MAX_LENGTH);
	_socket.async_read_some(boost::asio::buffer(_data, MAX_LENGTH), std::bind(&CSession::HandleRead, this,
		std::placeholders::_1, std::placeholders::_2, shared_from_this()));
}

//读回调函数
void CSession::HandleRead(const boost::system::error_code& error, size_t  bytes_transferred, shared_ptr<CSession> _self_shared) {
	if (!error) {
		cout << "read data is " << _data << endl;
		
		//将读取到的数据发送给客户端
		Send(_data, bytes_transferred);
		memset(_data, 0, MAX_LENGTH);

		//继续监听读取
		_socket.async_read_some(boost::asio::buffer(_data, MAX_LENGTH), std::bind(&CSession::HandleRead, this,
			std::placeholders::_1, std::placeholders::_2, _self_shared));
	}
	else {
		std::cout << "handle read failed, error is " << error.what() << endl;

		//如果读取失败，清除session
		_server->ClearSession(_uuid);
	}
}

//填入消息队列发送数据
void CSession::Send(char* msg, int max_length) {
	bool pending = false;//是否有待发送的数据
	std::lock_guard<std::mutex> lock(_send_lock);
	if (_send_que.size() > 0) {//如果有待发送的数据，将数据放入队列，等待发送
		pending = true;
	}
	_send_que.push(make_shared<MsgNode>(msg, max_length));
	if (pending) {//如果有待发送的数据，直接返回
		return;
	}
	//如果没有待发送的数据，直接发送
	boost::asio::async_write(_socket, boost::asio::buffer(msg, max_length),
		std::bind(&CSession::HandleWrite, this, std::placeholders::_1, shared_from_this()));
}

//写回调函数
void CSession::HandleWrite(const boost::system::error_code& error, shared_ptr<CSession> _self_shared) {
	if (!error) {
		std::lock_guard<std::mutex> lock(_send_lock);
		//由于async_write一次发送数据完整，所以可以直接pop
		_send_que.pop();
		if (!_send_que.empty()) {//如果还有待发送的数据，继续发送
			auto& msgnode = _send_que.front();
			boost::asio::async_write(_socket, boost::asio::buffer(msgnode->_data, msgnode->_max_len),
				std::bind(&CSession::HandleWrite, this, std::placeholders::_1, _self_shared));
		}
	}
	else {
		std::cout << "handle write failed, error is " << error.what() << endl;
		//如果写入失败，清除session
		_server->ClearSession(_uuid);
	}
}
```

- start中开始异步读，读回调有send函数（异步写），然后接着开始异步读，继续监听
- send函数开始异步写（如果此时没有在发送消息），写回调中开始异步写，直到全部写完

> 注意，此方式中:
>
> 异步读->读回调（这个里面调用send，开启异步写）->异步读
>
> 异步写->写回调->异步写



如果回调函数中不增加引用计数，就会存在风险，假设异步读**之后**，客户端关闭连接，此时读回调`调用`send开启异步写，然后读回调`开启`异步读，会Clearsesson（uuid）两次，这样可能会导致：shared_ptr引用计数变为0，最后一次调用的时候，访问不存在的内存



# Cserver类

## Cserver类定义



```cpp
#pragma once
#include <boost/asio.hpp>
#include "CSession.h"
#include <memory.h>
#include <map>
using namespace std;
using boost::asio::ip::tcp;
class CServer
{
public:
    //accecptor需要端口
	CServer(boost::asio::io_context& io_context, short port);
    
    //删除map中的session
	void ClearSession(std::string);
private:
    
    //接受回调，第一个参数是智能指针
	void HandleAccept(shared_ptr<CSession>, const boost::system::error_code & error);
   
	void StartAccept(); //开启接受服务器（接受状态）
    
	boost::asio::io_context &_io_context;
	short _port;
	tcp::acceptor _acceptor;
	std::map<std::string, shared_ptr<CSession>> _sessions;
};
```





## Cserver实现

```cpp
#include "CServer.h"
#include <iostream>
CServer::CServer(boost::asio::io_context& io_context, short port):_io_context(io_context), _port(port),
_acceptor(io_context, tcp::endpoint(tcp::v4(),port))
{
	StartAccept();
}

//接受回调函数
void CServer::HandleAccept(shared_ptr<CSession> new_session, const boost::system::error_code& error){
	if (!error) {
		new_session->Start();//开启监听
		_sessions.insert(make_pair(new_session->GetUuid(), new_session));//插入map中
	}
	else {
		cout << "session accept failed, error is " << error.what() << endl;
	}

    //继续开启服务器
	StartAccept();
}


//开启接受状态，注意，session必须是智能指针！！
void CServer::StartAccept() {
	shared_ptr<CSession> new_session = make_shared<CSession>(_io_context, this);
    
    //接受到新的连接，此时new_session->GetSocket()是被绑定到一个会话了，然后回调，注意传参是shared_ptr
	_acceptor.async_accept(new_session->GetSocket(), std::bind(&CServer::HandleAccept, this, new_session, placeholders::_1));
    
    //此函数结束后，new_session销毁，引用计数-1，但是回调函数中还在使用，所以没释放内存
}

//map中删除这个session，引用计数-1
void CServer::ClearSession(std::string uuid) {
	_sessions.erase(uuid);
}
```







# 易错点

## shared_ptr的初始化问题

不能用两个智能指针管理同一块内存，如下用法是错误的：

```cpp
void CSession::Start(){
    memset(_data, 0, MAX_LENGTH);
    _socket.async_read_some(boost::asio::buffer(_data, MAX_LENGTH), std::bind(&CSession::HandleRead, this, 
        std::placeholders::_1, std::placeholders::_2, shared_ptr<CSession>(this)));
}
```

shared_ptr\<CSession>(this)生成的新智能指针和this之前绑定的智能指针并不共享引用计数，所以要通过shared_from_this()函数返回智能指针，该智能指针和其他管理这块内存的智能指针共享引用计数：

```cpp
void CSession::Start(){
    memset(_data, 0, MAX_LENGTH);
    _socket.async_read_some(boost::asio::buffer(_data, MAX_LENGTH), std::bind(&CSession::HandleRead, this, 
        std::placeholders::_1, std::placeholders::_2, shared_from_this()));
}
```

同理，send函数中第一次发送也要绑定shared_from_this()

```cpp
//填入消息队列发送数据
void CSession::Send(char* msg, int max_length) {
	bool pending = false;//是否有待发送的数据
	std::lock_guard<std::mutex> lock(_send_lock);
	if (_send_que.size() > 0) {//如果有待发送的数据，将数据放入队列，等待发送
		pending = true;
	}
	_send_que.push(make_shared<MsgNode>(msg, max_length));
	if (pending) {//如果有待发送的数据，直接返回
		return;
	}
	//如果没有待发送的数据，直接发送
	boost::asio::async_write(_socket, boost::asio::buffer(msg, max_length),
		std::bind(&CSession::HandleWrite, this, std::placeholders::_1, shared_from_this()));
}
```









## shared_from_this()

`shared_from_this()` 方法来获取当前对象的 `shared_ptr` 实例。这样可以确保你得到的 `shared_ptr` 与最初用于创建当前对象的 `shared_ptr` 共享相同的控制块。



> 注意，使用shared_from_this(),必须保证这个类对象是用make_shared方式创建的！！！



# 流程梳理

- 服务器 (`Server`) 初始化一个新的会话 (`Session`)，并为该会话分配一个网络套接字 (`Socket`)。
- 接受`session`后，回调函数中开启监听(`async_read_some`),然后回调函数中继续开启接受状态（`StartAccept()`)
- 监听到消息，`读`回调中调用`send`（开启写），然后`读`回调中继续开启监听（`async_read_some`）
- send开启写(`async_write`),`写`回调中继续`写`，直到写完
- sesson有问题，直接删除即可，整个异步过程的所有函数都有此对象的引用，直到所有函数执行完毕，引用计数才会清0