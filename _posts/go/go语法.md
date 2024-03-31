---
title: go语法
date: 2024-3-24 22:40:40
tags: go
categories: 
- go
- 语法
---





# go类型

#### 引用类型

- **切片（Slice）**
- **映射（Map）**
- **通道（Channel）**
- **指针（Pointer）**
- **接口（Interface）**

这些类型的变量实际上存储的是对数据的引用，而不是数据本身。



#### 非引用类型

- **基本类型**：整型（Integers）、浮点型（Floats）、布尔型（Booleans）
- **复合类型**：数组（Arrays）、结构体（Structs）
- **字符串（String）**







# 空结构体

空结构体大小为0字节

> 节约内存，结合map，channel使用

```go
package main

import (
    "fmt"
    "unsafe"
)

type struct1 struct {
}

func main() {
    a := struct1{}
    fmt.Println(unsafe.Sizeof(a)) 	//0
    fmt.Printf("%p\n", &a)			//0x617460
}
```





# string



## 字符串拼接

直接使用”+“，会制造一个新的字符串

频繁操作string用`strings.Builder`,`strings.Join`



## 字符串指针

```go
package main

import (
	"fmt"
	"unsafe"
)

func main() {
	fmt.Println(unsafe.Sizeof("abc"))			//16
	fmt.Println(unsafe.Sizeof("abcdefg"))		//16
}

```









# 切片

切片共享底层，append函数，扩容的时候会换底层





# map

必须先初始化，才可以使用





# 闭包

闭包捕获的是外部变量的`引用`

如果不想被捕获,有两种方式：传参，或者复制一份(**对于引用类型，不适用**)



```go
package main

import "fmt"

func main() {
    // 非引用类型：整型
    var num int = 10
    modifyNum := func() {
        num:=num+5  
        fmt.Println(num) //输出:15
    }
    modifyNum()
    fmt.Println(num)                            // 输出:10，因为闭包内的修改不影响外部变量

    // 引用类型：切片
    elements := make([]int,3,5)
    fmt.Printf("elements: %p\n",&elements)      //输出:elements: 0xc00000c030

    elements[0],elements[1],elements[2]=1,2,3
    addElement := func(e int) {
        fmt.Printf("elements: %p\n",&elements)  //输出:elements: 0xc00000c030
        elements[0]=e
    }
    addElement(4)
    fmt.Println(elements)                       // 输出:[4 2 3]
    

    addElement2 := func(ele []int) {
        ele[0]=5
        fmt.Printf("ele: %p\n",&ele)            //输出:ele: 0xc00000c060
    }
    addElement2(elements)
    fmt.Println(elements)                       // 输出：[5 2 3]
    
}
```





# panic

`panic`不会叠加的方式存在，而是会被最近的`panic`覆盖，直到遇到`recover`为止。

```go
package main

import "fmt"

func panic1() {
	defer func() {
		if err := recover(); err != nil {
			fmt.Println("Recovered in panic1:", err)
		}
	}()
	defer func() {
		panic("defer panic in panic1")
	}()
	panic("panic in panic1")
}

func main() {
	panic1()
	fmt.Println("Main function continues after panic1.")
}
```



>  输出

```shell
Recovered in panic1: defer panic in panic1
Main function continues after panic1.
```





# new和make



- **`new(T)`** 返回的是一个指向类型 `T` 零值的**指针**。它适用于所有类型，包括切片、映射、通道等，但返回的总是一个**指向零值**的指针。对于切片，**不会分配底层数组**
- **`make(T, args...)`** 只能用于切片、映射和通道的内存分配和初始化。它返回的是一个初始化后的 `T` 类型的实例，而不是指针。对于切片，`make` 还会**分配底层数组**。





```go
package main

import "fmt"

func main() {
	//***********************************切片*************************************

	a := new([]int)
	//必须使用*解引用
	fmt.Println(*a == nil) //true
	*a = append(*a, 1)
	fmt.Println(a, *a) //&[1] [1]

	b := make([]int, 0)
	fmt.Println(b == nil) //false
	b = append(b, 1)
	fmt.Println(&b, b) //&[1] [1]

	//***********************************数组*************************************

	c := *new([10]int)
	fmt.Println(&c == nil) //false
	fmt.Println(c)         //[0 0 0 0 0 0 0 0 0 0]

	d := [10]int{}
	fmt.Println(&d == nil) //false
	fmt.Println(d)         //[0 0 0 0 0 0 0 0 0 0]
}

```





# 双引号和反引号区别

双引号字符串支持转义

反引号不支持转义





# 反射







# runtime







