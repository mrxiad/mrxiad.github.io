---
title: asio socket同步读写
date: 2023-11-12 11:09:40
tags: c++
categories: 网络编程
---









# 同步写write_some

write_some可以每次向指定的空间写入固定的字节数，如果写缓冲区满了，就只写一部分，返回写入的字节数。

```cpp
void wirte_to_socket(asio::ip::tcp::socket& sock) {
	std::string buf = "Hello";
	std::size_t total_bytes_written = 0;
	while (total_bytes_written != buf.length()) {
		total_bytes_written += sock.write_some(
			asio::buffer(buf.c_str() + total_bytes_written, buf.length() - total_bytes_written)
		);
	}
}
```





# 同步写send



send函数会一次性将buffer中的内容发送给对端，如果有部分字节因为发送缓冲区满无法发送，则阻塞等待，直到发送缓冲区可用，则继续发送完成。

```cpp
int send_data_by_send() {
	std::string raw_ip_address = "127.0.0.1";
	unsigned short port_num = 3333;

	try {
		//创建端点
		asio::ip::tcp::endpoint
			ep(asio::ip::address::from_string(raw_ip_address),
				port_num);
		asio::io_service ios;
		//打开套接字
		asio::ip::tcp::socket sock(ios, ep.protocol());
		//连接套接字
		sock.connect(ep);
		std::string buf = "Hello World!";
		int send_length = sock.send(asio::buffer(buf.c_str(), buf.length()));
		if (send_length <= 0) {
			std::cout << "send failed" << std::endl;
			return 0;
		}
	}
	catch (boost::system::system_error& e) {
		std::cout << "Error occured! Error code = " << e.code()
			<< ". Message: " << e.what();
		return e.code().value();
	}
	return 0;
}
```





# 同步写write

类似send方法，asio还提供了一个write函数，可以一次性将所有数据发送给对端，如果发送缓冲区满了则阻塞，直到发送缓冲区可用，将数据发送完成。

```cpp
int send_data_by_wirte() {
	std::string raw_ip_address = "127.0.0.1";
	unsigned short port_num = 3333;
	try {
		//创建端点
		asio::ip::tcp::endpoint
			ep(asio::ip::address::from_string(raw_ip_address),
				port_num);
		asio::io_service ios;
		//打开套接字
		asio::ip::tcp::socket sock(ios, ep.protocol());
		//连接套接字
		sock.connect(ep);
		std::string buf = "Hello World!";
		//发送数据
		int send_length = asio::write(sock, asio::buffer(buf.c_str(), buf.length()));
		if (send_length <= 0) {
			std::cout << "send failed" << std::endl;
			return 0;
		}
	}
	catch (boost::system::system_error& e) {
		std::cout << "Error occured! Error code = " << e.code()
			<< ". Message: " << e.what();
		return e.code().value();
	}
	return 0;
}
```



# 同步读read_some

同步读和同步写类似，提供了读取指定字节数的接口read_some

```cpp
std::string read_from_socket(asio::ip::tcp::socket& sock) {
	const unsigned char MESSAGE_SIZE = 7;
	char buf[MESSAGE_SIZE];
	std::size_t total_bytes_read = 0;
    
    //如果没有读到7个字节，那么一直阻塞
	while (total_bytes_read != MESSAGE_SIZE) {
		total_bytes_read += sock.read_some(
			asio::buffer(buf + total_bytes_read,
				MESSAGE_SIZE - total_bytes_read));
	}
	return std::string(buf, total_bytes_read);
}
int read_data_by_read_some() {
	std::string raw_ip_address = "127.0.0.1";
	unsigned short port_num = 3333;
	try {
		//创建端点
		asio::ip::tcp::endpoint
			ep(asio::ip::address::from_string(raw_ip_address),
				port_num);
		asio::io_service ios;
		//打开套接字
		asio::ip::tcp::socket sock(ios, ep.protocol());
		//连接套接字
		sock.connect(ep);
		//读取数据
		read_from_socket(sock);
	}
	catch (boost::system::system_error& e) {
		std::cout << "Error occured! Error code = " << e.code()
			<< ". Message: " << e.what();
		return e.code().value();
	}
	return 0;
}
```





# 同步读receive

可以一次性同步读取对方发送的数据

```cpp
int read_data_by_receive() {
	std::string raw_ip_address = "127.0.0.1";
	unsigned short port_num = 3333;
	try {
		//创建端点
		asio::ip::tcp::endpoint
			ep(asio::ip::address::from_string(raw_ip_address),
				port_num);
		asio::io_service ios;
		//打开套接字
		asio::ip::tcp::socket sock(ios, ep.protocol());
		//连接套接字
		sock.connect(ep);
		const unsigned char BUFF_SIZE = 7;
		char buffer_receive[BUFF_SIZE];
		int receive_length = sock.receive(asio::buffer(buffer_receive, BUFF_SIZE));
		if (receive_length <= 0) {
			std::cout << "receive failed" << std::endl;
		}
	}
	catch (boost::system::system_error& e) {
		std::cout << "Error occured! Error code = " << e.code()
			<< ". Message: " << e.what();
		return e.code().value();
	}
	return 0;
}
```





# 同步读read

可以一次性同步接收对方发送的数据

```cpp
int read_data_by_read() {
	std::string raw_ip_address = "127.0.0.1";
	unsigned short port_num = 3333;
	try {
		//创建端点
		asio::ip::tcp::endpoint
			ep(asio::ip::address::from_string(raw_ip_address),
				port_num);
		asio::io_service ios;
		//打开套接字
		asio::ip::tcp::socket sock(ios, ep.protocol());
		//连接套接字
		sock.connect(ep);
		const unsigned char BUFF_SIZE = 7;
		char buffer_receive[BUFF_SIZE];
		int receive_length = asio::read(sock, asio::buffer(buffer_receive, BUFF_SIZE));
		if (receive_length <= 0) {
			std::cout << "receive failed" << std::endl;
		}
	}
	catch (boost::system::system_error& e) {
		std::cout << "Error occured! Error code = " << e.code()
			<< ". Message: " << e.what();
		return e.code().value();
	}
	return 0;
}
```



# 读取直到指定字符

```cpp
std::string  read_data_by_until(asio::ip::tcp::socket& sock) {
    asio::streambuf buf;
    // Synchronously read data from the socket until
    // '\n' symbol is encountered.  
    asio::read_until(sock, buf, '\n');
    std::string message;
    // Because buffer 'buf' may contain some other data
    // after '\n' symbol, we have to parse the buffer and
    // extract only symbols before the delimiter. 
    std::istream input_stream(&buf);
    std::getline(input_stream, message);
    return message;
 }
```

