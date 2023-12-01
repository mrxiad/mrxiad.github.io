---
title: c++单例模式和CRTP
date: 2023-11-21 14:20:56
tags: c++
categories: 
- 工程
- 设计模式
---





# 本文介绍c++的单例模式以及注意事项





# 简单的单例模式

## 实现步骤



1. **私有化构造函数**：确保不能从类外部直接构造对象。
2. **静态成员函数**：提供一个全局访问点，用于获取这个唯一的实例。
3. **静态成员变量**：存储类的唯一实例。
4. **删除复制构造函数和赋值操作符**：防止复制和赋值实例。
5. **可选的私有析构函数**：如果类管理资源，明确声明析构函数有助于资源的正确释放。



## 实例

```cpp
#include <iostream>
using namespace std;

class A {
public:
    // 提供全局访问点
    static A* instance() {
        if (m_instance == nullptr) {
            m_instance = new A();
        }
        return m_instance;
    }

    void show() {
        cout << name << endl;
    }

    // 禁止外部删除单例对象
    ~A() = default;

private:
    static A* m_instance;
    string name;

    // 私有化构造函数
    A() : name("a") {}

    // 私有化拷贝和赋值
    A(const A& a){}
    A& operator=(const A&){}
};

// 初始化静态成员变量
A* A::m_instance=nullptr;

int main()
{
	A::instance()->show();
    return 0;
}
```





# 模板单例

```cpp
#include<iostream>
#include <string>
using namespace std;

template<class T>
class Singleton {
public:
    static T* instance() {
        if (m_instance == nullptr) {
            m_instance = new T();
        }
        return m_instance;
    }

protected:	//注意这句是protected
    Singleton() {}
    
    //注意参数
    Singleton(const Singleton<T>&) = delete;
    
    //注意返回值和参数
    Singleton<T>& operator=(const Singleton<T>&) = delete;
    ~Singleton() {}

    static T* m_instance;
};

template<class T>
T* Singleton<T>::m_instance = nullptr;

class A : public Singleton<A> {
public:
     // 让Singleton类模板访问A的私有成员
    friend class Singleton<A>;
    
    void display() {
        cout << "A's instance method called." << endl;
    }

    // 其他 A 类的成员函数和变量
private:
    // 私有化构造函数，确保外部不能直接创建A的实例
    A() {}
};

int main()
{
	A* a = A::instance();
	a->display();
}
```

> 注意
>
> - 构造函数必须是类名，不允许加模板
> - 其他函数需要加模板
> - 基类类型是Singleton\<T>,子类类型是T   ！！！！！！！！！！！！！！





# CRTP

## 思想

CRTP的全称是**C**uriously **R**ecurring **T**emplate **P**attern，即奇异递归模板模式。

**特点：**

1. 基类是一个模板类
2. 派生类继承该基类时，**将派生类自身作为模板参数传递给基类**



## 基本实现

```cpp
// 基类是模板类
template <typename T>
class Base
{
public:
    virtual ~Base() {}
 
    void func()
    {
        if (auto t = static_cast<T *>(this))
        {
            t->op();
        }
    }
};
 
// 派生类Derived继承自Base，并以自身作为模板参数传递给基类
class Derived : public Base<Derived>
{
public:
    void op()
    {
        std::cout << "Derived::op()" << std::endl;
    }
};
```



> 通过使用static_cast，将this指针转换为模板参数类型T的指针，然后调用类型T的方法,这种方法是安全的



## CRTP的特点

- **优点**：省去动态绑定、查询虚函数表带来的开销。通过CRTP，`基类`可以`获得`到`派生类`的类型，提供各种操作，比普通的继承更加灵活。但`CRTP`基类并不会单独使用，只是作为一个模板的功能。
- **缺点**：模板的通病，即影响代码的可读性。



## 用途

### 1. 静态分发（“静态多态”）

```cpp
template <typename T>
class Base
{
public:
    Base() {}
    virtual ~Base() {}
 
    void func()
    {
        if (auto t = static_cast<T *>(this))
        {
            t->op();
        }
    }
};
 
class Derived1 : public Base<Derived1>
{
public:
    Derived1() {}
    void op()
    {
        std::cout << "Derived1::op()" << std::endl;
    }
};
 
class Derived2 : public Base<Derived2>
{
public:
    Derived2() {}
    void op()
    {
        std::cout << "Derived2::op()" << std::endl;
    }
};
 
// 辅助函数：完成静态分发
template<typename DerivedClass>
void helperFunc(Base<DerivedClass>& d)//注意参数d的类型
{
    d.func();
}
 
int main() 
{
    Derived1 d1;
    Derived2 d2;
    helperFunc(d1);
    helperFunc(d2);
 
    return 0;
}
```

> Base<Derived1>和Base<Derived2>不算同一个类型

**注意：**

- 模板类或模板函数在`调用`时才会实例化。因此当`Base<Derived1>::func()`被调用时，`Base<Derived1>`已经知道`Derived1::op()`的存在





### 2. 计数器

```cpp
template<typename T>
class Counter
{
public:
    //静态变量
    static int count;
    Counter()
    {
        //注意类型
        ++Counter<T>::count;
    }
    ~Counter() 
    {
        //注意类型
        --Counter<T>::count;
    }
};
//初始化
template<typename T>
int Counter<T>::count = 0;
 
class DogCounter : public Counter<DogCounter>
{
public:
    int getCount()
    {
        return this->count;
    }
};
 
class CatCounter : public Counter<CatCounter>
{
public:
    int getCount()
    {
        return this->count;
    }
};
 
int main() 
{
    DogCounter d1;
    std::cout << "DogCount : " << d1.getCount() << std::endl;//1
    {
        DogCounter d2;
        std::cout << "DogCount : " << d1.getCount() << std::endl;//2
    }
    std::cout << "DogCount : " << d1.getCount() << std::endl;//1
 
    CatCounter c1, c2, c3, c4, c5[3];
    std::cout << "CatCount : " << c1.getCount() << std::endl;//7
 
    return 0;
}
```

输出如下：

```powershell
DogCount : 1
DogCount : 2
DogCount : 1
CatCount : 7
```





### 3.单例模式

见模板单例





# 总结



1. 单例模式下，`构造函数`**不**允许加模板，`其他函数`需要**加**模板
2. 单例模式下访问权限：子类需要基类的`友元`（用于基类访问子类构造函数），基类需要使用`protected`（用于子类访问基类的构造函数）
3. 子类需要`私有化构造函数`

