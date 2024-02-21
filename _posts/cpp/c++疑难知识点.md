---
title: c++恶心知识点
date: 2024-2-21 15:19:12
tags: 技术
categories:
- c++
---



# 本文介绍c++常用知识点



## 内存溢出和内存泄漏，数组越界

- **内存溢出（Memory Overflow）**：这通常发生在试图分配的内存超过了系统可用内存的情况。例如，在一个有限的内存空间中创建太多对象或数据结构。
- **内存泄漏（Memory Leak）**：这发生在分配的内存没有得到适当释放的时候。如果程序反复分配内存而不释放，最终可能耗尽系统资源。
- **数组越界（Array Index Out of Bound）**：当试图访问数组之外的索引时发生。这是常见的编程错误，可能导致程序崩溃或不可预期的行为





## 多线程和多进程的关系

- **多线程（Multithreading）**：在一个进程内部，可以创建多个线程，这些线程共享相同的内存空间和资源，但可以并行执行不同的任务。多线程适用于轻量级的并行任务，尤其是在需要共享大量数据时。
- **多进程（Multiprocessing）**：每个进程有自己独立的内存空间和资源。多进程可以在不同的CPU核心上并行执行，适用于需要隔离和安全性的重量级任务。进程间通信比线程间通信更复杂和开销更大。



## 指针字节

- 在 32 位系统上，指针的大小通常是 4 字节（32 位）。
- 在 64 位系统上，指针的大小通常是 8 字节（64 位）。、



去除x的第一个字节的数字（可以判断大端序小端序）

```cpp
#include<bits/stdc++.h>
using namespace std;

int main()
{
    int x=1;
    cout<<(int)*(char*)&x;		//x占4字节，char*取出第一字节，转化为int类型输出
    return 0;
}
```

遍历x的每个字节并打印输出

```cpp
#include<bits/stdc++.h>
using namespace std;

int main()
{
    int x=1;
    for(int i=0;i<4;i++){
        cout<<(int)*((char*)&x+i);
    }
    return 0;
}
```

**注意：**

- 任何类型的指针都是4字节

- char\*指向的内存为1字节，int\*指向的内存为4字节

**`char*` 和 `int*` 指针的增加区别**:

- 当您对 `char*` 类型的指针执行 `++` 操作时，指针将向前移动 1 字节，因为 `char` 的大小是 1 字节。
- 相比之下，对 `int*` 类型的指针执行 `++` 操作时，指针将向前移动 `sizeof(int)` 字节，通常是 4 字节（这取决于编译器和系统架构）。
- 这反映了指针的算术操作是基于它们指向的数据类型的大小。





## 类里面尽量用指针

```cpp
#include<bits/stdc++.h>
using namespace std;
class A{
    int x;
    A* a; //不允许写成A a(不允许使用不完整的类型)
};
int main()
{
    A a;
    return 0;
}
```





## C++ 库目录和头文件目录

- **头文件目录**：这是存放 C++ 头文件（通常以 `.h` 或 `.hpp` 结尾）的目录。这些头文件包含了类定义、模板和函数声明等。
- **库目录**：这是存放编译后的库文件的目录。对于静态库，通常是 `.a` 或 `.lib` 文件；对于动态库（共享库），是 `.so`（在 Linux 上）或 `.dll`（在 Windows 上）文件。



## 在 Linux 下指定动态链接库的目录:

- **运行时指定动态链接库的目录**:
	- 如果动态链接库不在标准的库目录（如 `/lib` 或 `/usr/lib`）中，可以通过设置环境变量 `LD_LIBRARY_PATH` 来指定额外的库搜索路径。
	- `LD_LIBRARY_PATH` 环境变量包含一系列目录，运行时链接器（dynamic linker）将在这些目录中搜索动态链接库。
	- 如果 `LD_LIBRARY_PATH` 没有正确设置，当程序尝试加载一个不在标准路径中的库时，会报错，如 “cannot open shared object file” 或类似的消息。
- **编译时指定动态链接库的目录**:
	- 在编译阶段，使用 `g++` 或其他编译器时，确实需要指定动态链接库的位置，这样编译器能够正确链接这些库。
	- 通过 `-L` 选项指定库所在的目录，例如 `g++ -L/path/to/library`。
	- 同时，使用 `-l` 选项指定要链接的库的名称，例如 `g++ -lmylibrary`，这里 `mylibrary` 是库的名称，对应于 `libmylibrary.so` 文件。
- **注意事项**:
	- 需要注意的是，`LD_LIBRARY_PATH` 是运行时的设置，而 `-L` 和 `-l` 是编译时的设置。
	- 修改 `LD_LIBRARY_PATH` 对已经编译好的程序有效，但不会影响未来的编译过程。
	- 同样，编译时使用的 `-L` 和 `-l` 选项不会影响已经编译好的可执行文件的运行。
- **最佳实践**:
	- 尽管可以使用 `LD_LIBRARY_PATH` 来解决库的查找问题，但最佳实践是将库安装到标准路径，或者在程序的链接阶段正确指定库的位置。
	- 对于开发和部署，保持一致的库路径有助于避免运行时错误。



## make_shared隐患

```cpp
#include<bits/stdc++.h>
using namespace std;
class A{
public:
    A(){
        p=new int[10]();
    }
    ~A(){
        //A类内部new出来的需要手动delete
        delete[]p;
    }
public:
    int* p;
};
int main()
{
    shared_ptr<A> p1(new A);
    return 0;
}
```





## 嵌套类

```cpp
#include<bits/stdc++.h>
using namespace std;
class A{
public:
    class B{
        public:
            void print(){
                cout<<"Hello World"<<endl;
            }
        private:
    };
    B b;
public:
    B getB(){
        return b;
    }
};
int main()
{
    A a;
    A::B b=a.getB();
    b.print();
}
```

> 注意作用域



## c++的按位取反操作

```cpp
#include<bits/stdc++.h>
using namespace std;

int main()
{
    for(int i=-1e8;i<=1e8;i++){
        if(-i!=(~i+1)){
            cout<<"NO"<<endl;
            return 0;
        }
    }
    cout<<"YES";
}
```

> 最终输出yes





## 函数指针

### 基本函数指针语法

假设我们有一个简单的函数，如下所示：

```cpp
void myFunction(int a) {
    std::cout << "Value: " << a << std::endl;
}
```

为了声明一个指向这个函数的指针，你可以使用以下语法：

```cpp
void (*functionPtr)(int);
```

这里，`functionPtr`是一个指针，指向一个接受`int`参数并返回`void`的函数。你可以这样赋值给它：

```cpp
functionPtr = myFunction;
```

然后，你可以通过解引用指针来调用函数，如下：

```cpp
(*functionPtr)(10); 
// 或者 functionPtr(10); 两者在这里是等价的
```

### 使用typedef简化函数指针

使用`typedef`可以使函数指针的声明更简单、更易读。继续使用上面的例子，我们可以这样定义一个类型别名：

```cpp
typedef void (*FunctionPointerType)(int);
```

然后使用这个别名来声明指针：

```cpp
FunctionPointerType myPtr = myFunction;
myPtr(10); // 调用函数
```

### 函数指针指向类成员函数

当你想要指针指向类的成员函数时，语法会稍有不同，因为你需要处理`this`指针。这里是一个例子：

```cpp
class MyClass {
public:
    void memberFunction(int a) {
        std::cout << "Class Value: " << a << std::endl;
    }
};
```

为了声明一个指向这个成员函数的指针，你需要这样做：

```cpp
void (MyClass::*ptr)(int) = &MyClass::memberFunction;
```

使用`typedef`简化：

```cpp
typedef void (MyClass::*MyClassFunctionPointer)(int);
MyClassFunctionPointer ptr = &MyClass::memberFunction;
```

调用这个成员函数指针时，你需要一个类的实例，并且使用`->*`或`.*`运算符，如下：

```cpp
MyClass obj;
(obj.*ptr)(10); // 如果ptr是一个成员函数指针

MyClass *objPtr = &obj;
(objPtr->*ptr)(10); // 如果ptr是一个成员函数指针并且你有一个指向对象的指针
```







## 虚函数表

### 1.类的静态函数和构造函数不可以定义为虚函数



### 2.将析构函数定义为虚函数的作用

> 类的构造函数不能定义为虚函数，析构函数可以定义为虚函数，这样当我们delete一个指向子类对象的基类指针时可以达到调用子类析构函数的作用，从而动态释放内存。



### 3.要点

- 一个类对象其内存分布的基本结构为**虚函数表地址+非静态成员变量**，类的成员函数**不占**用类对象的空间，他们分布在一片属于类的共有区域。
- 类的静态成员函数喝成员变量不占用类对象的空间，他们分配在静态区。
- 虚函数表的地址存储在类对象的起始位置。所以我们利用这个原理，通过寻址的方式访问虚函数表里的函数



## 4.代码演示

```cpp
#include<bits/stdc++.h>

using namespace std;


class Baseclass
{

public:
    Baseclass() : a(1024) {}
    virtual void f() { cout << "Base::f" << endl; }
    virtual void g() { cout << "Base::g" << endl; }
    virtual void h() { cout << "Base::h" << endl; }
    int a;
};

// 0 1 2 3   4 5 6 7(虚函数表空间)    8 9 10 11 12 13 14 15(存储的是a)

class DeriveClass : public Baseclass
{
public:
    virtual void f() { cout << "Derive::f" << endl; }
    virtual void g2() { cout << "Derive::g2" << endl; }
    virtual void h3() { cout << "Derive::h3" << endl; }
};

void uesVitualTable()
{
    typedef void (*Func)(void);
    Baseclass b;
    b.a=1024;
    cout<<"sizeof(b):"<<sizeof(b)<<endl;

    int *p=(int*)&b;    //取出虚函数表的地址
    cout<<"虚函数表地址的地址："<<p<<endl;
    cout<<"b的地址:"<<&b<<endl;

    cout<<"b.a的地址:"<<p+2<<endl;//这一步涉及到内存对齐
    cout<<"b.a的值:"<<*(p+2)<<endl;

    cout<<"虚函数表的地址:"<<(int*)(*p)<<endl;

    cout << "int大小:" << sizeof(int) << endl;
    cout << "p大小:" << sizeof(p) << " int*大小" << sizeof(int *) << endl;

    Func pFunc = (Func)(*(int*)(*p));
    pFunc();

    pFunc = (Func)(*p+1);
    pFunc();

    pFunc = (Func)(*p+2);
    pFunc();
}

int main()
{
    uesVitualTable();
}

/*


*/
```



### 5.多继承

> 会含有多个虚函数表



## 对齐和补齐规则

对齐：类(结构体)对象每个成员分配内存的起始地址为其所占空间的整数倍。
补齐：类(结构体)对象所占用的总大小为其内部最大成员所占空间的整数倍。