---
title: asio延长session生命周期
date: 2023-11-13 17:25:15
tags: c++
categories: 网络编程
---







------

# 引言

asio异步应答服务器存在隐患，就是因为使用了delete删除session对象，而本章使用智能指针防止重复delete对象session，使用智能指针构造成一个伪闭包的状态延长session的生命周期。



# 智能指针管理Session

我们可以通过智能指针的方式管理Session类，将acceptor接收的链接保存在`Session类型`的`智能指针`里。由于智能指针会在引用计数为0时自动析构，所以为了防止其被自动回收，也方便Server管理Session。因为我们后期会做一些重连踢人等业务逻辑，我们在Server类中添加成员变量，该变量为一个map类型，key为Session的uid，value为该Session的智能指针,此时也会增加引用计数



