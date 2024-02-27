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

- 虚函数表是虚函数指针数组
- 一个类对象其内存分布的基本结构为**虚函数表地址+非静态成员变量**，类的成员函数**不占**用类对象的空间，他们分布在一片属于类的共有区域。
- 类的静态成员函数喝成员变量不占用类对象的空间，他们分配在静态区。
- **虚函数表的地址**存储在类对象的起始位置。所以我们利用这个原理，通过寻址的方式访问虚函数表里的函数
- 一个类得所有实例共享一个虚函数表



## 4.代码演示

```cpp
#include <iostream>
class A {
public:
    virtual void func1() { std::cout << "A::func1" << std::endl; }
    virtual void func2() { std::cout << "A::func2" << std::endl; }
    virtual ~A() {} // 虚析构函数，确保派生类的正确析构
};

int main() {
    A a;
    // 获取虚函数表的地址
    void** vptr = *reinterpret_cast<void***>(&a);

    typedef void (*Func1Ptr)();
    Func1Ptr func1 = reinterpret_cast<Func1Ptr>(vptr[0]);
    func1();

    typedef void (*Func2Ptr)();
    Func2Ptr func2 = reinterpret_cast<Func2Ptr>(vptr[1]);
    func2(); 
    return 0;
}
```



### 5.多继承

> 会含有多个虚函数表



## 对齐和补齐规则

对齐：类(结构体)对象每个成员分配内存的起始地址为其所占空间的整数倍。
补齐：类(结构体)对象所占用的总大小为其内部最大成员所占空间的整数倍。







## 引用折叠

引用折叠是C++中的一个规则，用于确定当模板实例化或类型别名声明中出现多重引用时，引用应该如何被处理。其规则如下：

- `T& &`、`T& &&`、`T&& &`都折叠成`T&`
- `T&& &&`折叠成`T&&`



## 通用引用

### 语法

通用引用的语法很简单，只需在类型后面加上`&&`，并且这个类型必须是模板参数的推导类型。例如：

```cpp
template<typename T>
void function(T&& arg) {
    // ...
}
```

在这个例子中，`T&&`类型的参数`arg`是一个通用引用。

### 工作原理

当一个左值被传递给`function`时，模板参数`T`被推导为左值引用类型。例如，如果你传递一个`int`类型的左值给`function`，`T`将被推导为`int&`，使得`T&&`实际上成为`int& &&`，根据引用折叠规则（Reference Collapsing Rules），它会变成`int&`。

相反，如果一个右值被传递给`function`，`T`将被推导为该右值的实际类型，所以如果你传递一个`int`类型的右值，`T`将被推导为`int`，使得`T&&`实际上是`int&&`。

> 通用引用避免重载



## 完美转发

直接转发会导致所有参数都被视为左值。为了保持参数的原始左值或右值属性，使用`std::forward`。

```cpp
#include <utility>
#include <iostream>

// 目标函数，接受一个右值引用参数
void function(int&& x)
{
    std::cout << "function called with right value\n";
}

// 目标函数，重载版本接受一个左值引用参数
void function(int& x)
{
    std::cout << "function called with left value\n";
}


// 函数模板，使用通用引用和完美转发
template<typename T>
void wrapper(T&& arg)
{
    // 完美转发arg到另一个函数
    function(std::forward<T>(arg));
}


int main()
{
    int x = 10;
    wrapper(x);  // 输出: function called with left value
    wrapper(20); // 输出: function called with right value
}
```





## 完美转发与可变参数

当我们结合使用可变参数模板和完美转发时，可以创建非常通用的函数包装器，这些包装器能将任意数量和类型的参数完美地转发给其他函数。

```cpp
template<typename F, typename... Args>
auto wrapper(F&& f, Args&&... args) -> decltype(f(std::forward<Args>(args)...)) {
    return f(std::forward<Args>(args)...);
}
```

在这个例子中：

- `F&& f` 使用通用引用接受任何类型的函数或可调用对象。
- `Args&&... args` 是一个参数包，使用通用引用接受任意数量和类型的参数。
- `std::forward<Args>(args)...` 完美转发这些参数给函数`f`。
- `decltype(f(std::forward<Args>(args)...))` 用于推断函数`f`调用的返回类型。

### 展开参数包

参数包的展开是通过递归函数调用或初始化列表的方式实现的。在完美转发的上下文中，通常不需要手动展开参数包，因为`std::forward<Args>(args)...`语法已经为我们完成了这项工作。

### 使用示例一

假设有一个函数`void print(int a, double b)`，我们想通过`wrapper`函数完美转发参数给它：

```cpp
void print(int a, double b) {
    std::cout << a << ", " << b << std::endl;
}

int main() {
    wrapper(print, 5, 3.14); // 完美转发5和3.14到print函数
}
```

> 注意：这里`return void`是合法的



### 使用示例二

```cpp
#include <string>
#include <utility> // For std::forward

class MyClass {
public:
    // 基础构造函数，执行主要的初始化逻辑
    MyClass(int x, const std::string& name) : value(x), name(name) {}

    // 模板构造函数，接受一个int和可变数量的其他参数
    // 使用完美转发将其他参数转发到std::string的构造函数
    template<typename... Args>
    MyClass(int x, Args&&... args)
        : MyClass(x, std::string(std::forward<Args>(args)...)) {//委托构造
        // 这里可以添加特定于此构造函数的逻辑
    }

private:
    int value;
    std::string name;
};
```





## 异常





## allocator类





## 强制类型转换

1. **`static_cast`**：

	- 用于非多态类型的转换。

	- 可以进行基本数据类型之间的转换，如将`float`转换为`int`。

	- 也用于类层次结构中基类和派生类之间的指针或引用的转换，但不进行运行时类型检查。

	- **场景示例**：将`double`类型转换为`int`，或将基类指针转换为派生类指针（向下转型）。

		```cpp
		double d = 9.5;
		int i = static_cast<int>(d); // 将double转换为int
		```

2. **`dynamic_cast`**：

	- 主要用于处理类层次中的向下转型，特别是在多态基类和派生类之间转换时。

	- 进行转换时会进行类型安全检查，如果转换失败则返回`nullptr`（对于指针）。

	- **场景示例**：在一个类继承体系中，将基类指针转换为派生类指针时检查安全性。

		```cpp
		Base* b = new Derived;
		Derived* d = dynamic_cast<Derived*>(b); // 安全转换，检查b是否真的指向Derived或其派生类的对象
		```

3. **`const_cast`**：

	- 用于修改类型的`const`或`volatile`属性，最常见的是去除`const`属性。

	- **场景示例**：当你想在一个只接受非`const`参数的函数中使用`const`数据时。

		```cpp
		const int ci = 10;
		int* modifiable = const_cast<int*>(&ci); // 去除const属性
		```

4. **`reinterpret_cast`**：

	- 用于进行低级别的重新解释转换，几乎可以在任意指针类型之间转换，也可用于指针与足够大的整型之间的转换。

	- 它不检查安全性，使用时需要特别小心。

	- **场景示例**：将指针类型转换为一个足够大的整数类型，或在不同的函数指针类型之间转换。

		```cpp
		int ci = 10;
		long p = reinterpret_cast<long>(&ci); // 将指针转换为整数类型
		```

### 使用场景总结：

- **`static_cast`**：适用于大多数类型转换，包括基础数据类型转换和非多态类型的转换。
- **`dynamic_cast`**：适用于多态类型的安全向下转型，能够在运行时检查转换的有效性。
- **`const_cast`**：适用于修改`const`或`volatile`属性的场景。
- **`reinterpret_cast`**：适用于低级转换，如指针类型之间的转换，需要开发者非常小心使用。