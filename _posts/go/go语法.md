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





# defer 的执行顺序

defer执行顺序和调用顺序相反，类似于栈**后进先出**(LIFO)。

defer在return**之后**执行，但在函数退出之前，defer可以修改返回值。下面是一个例子：





# panic

> 先执行defer，在执行panic！！！

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

在 Go 语言中，反射的核心由 `reflect` 标准库提供，主要涉及两个类型：`Type` 和 `Value`。

- **`reflect.Type`** 表示 Go 语言中的类型。它提供了类型的名称、种类（如 `int`、`struct` 等）以及可以进行哪些操作的信息。
- **`reflect.Value`** 表示 Go 语言中的具体值。它可以用来获取或设置值、调用方法等。



## 主要方法

```go
reflect.TypeOf()
Elem()					//把指针类型转成普通类型


NumField()				//获取成员变量的个数
Field()					//获取具体成员变量详细信息
FieldByIndex()			//获取具体成员变量详细信息

NumMethod()				//获取成员方法的个数
Method() 				//获取具体成员方法详细信息


NumIn()					//方法的入参个数
In()					//方法的入参类型
NumOut()				//方法的出参个数
Out()					//方法的出参类型
```





## TypeOf

> 返回 i 的反射Type类型
>
> 注意Elem()将指针类型转化为普通类型

```go
package main

import (
	"fmt"
	"reflect"
)

func getType() {
	typeInt := reflect.TypeOf(1)
	fmt.Println(typeInt)          // 打印 int
	fmt.Println(typeInt.String()) // 打印 int
	fmt.Println(typeInt.Kind())   // 打印 int

	typeString := reflect.TypeOf("hello")
	fmt.Println(typeString)          // 打印 string
	fmt.Println(typeString.String()) // 打印 string
	fmt.Println(typeString.Kind())   // 打印 string
}

// 自定义一个数据类型
type User struct {
	UserName string `我是Tag`
	Age      int
}

func getType2() {
	var u1 User
	typeUser1 := reflect.TypeOf(u1)
	fmt.Println(typeUser1)               // 打印 main.User
	fmt.Println(typeUser1.String())      // 打印 main.User
	fmt.Println(typeUser1.Kind())        // 打印 struct
	fmt.Println(typeUser1.Field(0).Name) // 打印 UserName

	var u2 = new(User)
	typeUser2 := reflect.TypeOf(u2)
	fmt.Println(typeUser2)          // 打印 *main.User
	fmt.Println(typeUser2.String()) // 打印 *main.User
	fmt.Println(typeUser2.Kind())   // 打印 ptr

	var u3 = new(User)
	typeUser3 := reflect.TypeOf(u3).Elem() // Elem():把指针类型转成普通类型
	fmt.Println(typeUser3)                 // 打印 main.User
	fmt.Println(typeUser3.String())        // 打印 main.User
	fmt.Println(typeUser3.Kind())          // 打印 struct
}
func main() {
	getType()
	getType2()
}

```



> 获取成员变量详细信息

```go
type User struct {
	UserName string `我是Tag`
	Age      int
	student  Student
}
 
type Student struct {
	score float32
}
 
func getField() {
	typeUser := reflect.TypeOf(User{})
	// 获取成员变量详情
	numField := typeUser.NumField()
	for i := 0; i < numField; i++ {
		field := typeUser.Field(i)
		fmt.Println("成员变量详情：", field)
	}
 
	// 获取结构体中嵌套结构体student中的变量score
	subField := typeUser.FieldByIndex([]int{2, 0}) // []int{2,0} 2是student在User中的下标，0是score在Student中的下标
	fmt.Println(subField)
}
```







> 获取成员方法详细信息

```go
type User struct {
	UserName string `我是Tag`
	Age      int
}
 
func (User) Insert() int {
	return 1
}
 
// 获取成员方法，不是指针方法，如果需要获取指针方法，需要reflect.TypeOf(&User{})取地址才可
func getMethod() {
	typeUser := reflect.TypeOf(User{})
	numMethod := typeUser.NumMethod()
	for i := 0; i < numMethod; i++ {
		method := typeUser.Method(i)
		fmt.Println("成员方法详情：", method)
	}
}
```





> 获取方法入参出参详细信息

```go
func Add(a, b int) int {
	return a + b
}
 
// 获取普通方法详情
func getFunc() {
	typeFunc := reflect.TypeOf(Add)
	fmt.Println("方法类型：", typeFunc.Kind())
 
	numIn := typeFunc.NumIn()
	fmt.Println("方法输入参数个数：", numIn)
	numOut := typeFunc.NumOut()
	fmt.Println("方法输出参数个数：", numOut)
 
	for i := 0; i < numIn; i++ {
		fmt.Printf("第 %d 个输入参数类型是 %s \n", i, typeFunc.In(i))
	}
	for i := 0; i < numOut; i++ {
		fmt.Printf("第 %d 个输出参数类型是 %s \n", i, typeFunc.Out(i))
	}
}
```



## ValueOf

> reflect.ValueOf 的使用

```go
func getValue() {
	intValue := reflect.ValueOf(1)
	stringValue := reflect.ValueOf("hello")
	userValue := reflect.ValueOf(User{})
	fmt.Println(intValue)    // 打印 1
	fmt.Println(stringValue) // 打印 hello
	fmt.Println(userValue)   // 打印 { 0}
}
```



> Value 转 Type

```go
func getValue() {
	intValue := reflect.ValueOf(1)
    // Value 转 Type
	intType := intValue.Type()	
	fmt.Println(intType)
}
```



> 指针结构体互相转换

```go
func trans() {
	userValue := reflect.ValueOf(&User{})
	fmt.Println(userValue.Kind()) // 打印 ptr
	// 指针 转成 结构体
	userValuePtr := userValue.Elem()
	fmt.Println(userValuePtr.Kind()) // 打印 struct
	// 结构体 转成 指针
	userValue2 := userValuePtr.Addr()
	fmt.Println(userValue2.Kind()) // 打印 ptr
}
```



> 反射类型转普通类型

```go
func trans() {
	iValue := reflect.ValueOf(1)
	// 方式一：把 反射类型转成普通类型
	iValue.Int()
	// 方式二：把 反射类型转成普通类型
	iValue2 := iValue.Interface().(int)
	fmt.Println(iValue2)
 
	// 把 反射类型 转成 结构体类型
	userType := reflect.ValueOf(User{})
	user := userType.Interface().(User)
	fmt.Println(user.UserName)
}
```



> 通过反射修改基础类型的值

```go
func changeValue() {
	var i = 10
	iValue := reflect.ValueOf(&i)
	fmt.Println(iValue.Kind()) // 打印 ptr
	// 判断是否是可寻址值
	if iValue.CanAddr() {
		iValue.SetInt(20)
		fmt.Println("i = ", i)	// if 进不来，无打印
	}
 
	// 通过 elem 把指针类型解析成反射类型。注意：elem 只能被指针类型的反射所调用，所以在ValueOf的时候带&取址符号
	iValue2 := iValue.Elem()
	fmt.Println(iValue2.Kind()) // 打印 int
	// 判断是否是可寻址值
	if iValue2.CanAddr() {
		iValue2.SetInt(30)
		fmt.Println("i = ", i) // 打印 30
	}
}
```



> 通过反射修改结构体成员变量的值

```go
type User struct {
	UserName string `我是Tag`
	Age      int
    gender   int    // 首字母小写是未导出字段，反射不能修改未导出字段
}
 
func changeValue() {
	var user = User{
		UserName: "张三",
		Age:      18,
	}
	fmt.Println("修改前的值：", user.UserName, user.Age) // 打印 张三 18
 
	userValue := reflect.ValueOf(&user).Elem()
	if userValue.CanAddr() {
		userValue.FieldByName("UserName").SetString("李四")
		userValue.FieldByName("Age").SetInt(28)
		fmt.Println("修改后的值：", user.UserName, user.Age) // 打印 李四 28
	}
}
```



> 通过反射修改嵌套结构体成员变量的值

```go
type User struct {
	UserName string `我是Tag`
	Age      int
	gender   int
	Student  Student
}
 
type Student struct {
	Score float32
}
 
func changeValue() {
	var user = User{
		UserName: "张三",
		Student: Student{
			Score: 98,
		},
	}
 
	userValue := reflect.ValueOf(&user).Elem()
	if userValue.CanAddr() {
		studentValue := userValue.FieldByName("Student")
		fmt.Println(studentValue.Kind()) // 打印 struct
		// 修改 Score 的值
		studentValue.FieldByName("Score").SetFloat(59)
		fmt.Println(user.Student.Score) // 打印 59
	}
}
```



>  通过反射修改 slice 切片中的值

```go
type User struct {
	UserName string `我是Tag`
	Age      int
    gender   int    // 首字母小写是未导出字段，反射不能修改未导出字段
}
 
func changeValue() {
	var userSlice = make([]*User, 3, 5)
	userSlice[0] = &User{
		UserName: "张三",
		Age:      18,
	}
 
	fmt.Println("修改前的数据：", userSlice[0].UserName, userSlice[0].Age) // 打印 张三 18
 
	sliceValue := reflect.ValueOf(userSlice)
	// 判断是否是可寻址的
	fmt.Println(sliceValue.CanAddr()) // 打印 false
	if sliceValue.Len() > 0 {
		// 获取切片中第0个元素
		sliceValue0 := sliceValue.Index(0)
		// 判断是否是可寻址的
		fmt.Println(sliceValue0.CanAddr()) // 打印 true
		// 查看是否是指针类型，如果是指针类型，需要 elem 解析
		fmt.Println(sliceValue0.Kind()) // 打印 ptr 是指针类型
		userValue := sliceValue0.Elem()
		userValue.FieldByName("UserName").SetString("李四")
		userValue.FieldByName("Age").SetInt(28)
		fmt.Println("修改后的数据：", userSlice[0].UserName, userSlice[0].Age) // 打印 李四 28
	}
 
	// 直接修改整个 user 对象
	sliceValue.Index(1).Set(reflect.ValueOf(&User{
		UserName: "王五",
		Age:      16,
	}))
	fmt.Println(userSlice[1].UserName, userSlice[1].Age) // 打印 王五 16
}
```





>  通过反射修改 map 中的值

```go
func changeValue() {
	u1 := &User{
		UserName: "张三",
		Age:      18,
	}
	u2 := &User{
		UserName: "李四",
		Age:      20,
	}
	// 定义一个map对象
	userMap := make(map[int]*User, 5)
	userMap[0] = u1
	userMap[1] = u2
 
	// 反射value对象
	mapValue := reflect.ValueOf(userMap)
	// 把value类型转成type类型
	mapType := mapValue.Type()
	// 获取map中 key 的数据类型
	keyType := mapType.Key()
	fmt.Println("key的数据类型是：", keyType) // 打印 int
	// 获取map中 value 的数据类型。 注意这里的 elem 是 Type 的方法，作用是获取mapType的元素类型，不是解析指针
	valueType := mapType.Elem()
	fmt.Println("value的数据类型是：", valueType) //打印 *main.User
 
	// 修改 map中映射对应的值中的某一个成员变量的数据，比如只修改用户的年龄
	u1Value := mapValue.MapIndex(reflect.ValueOf(1))            // 修改map中key是1的数据
	fmt.Println("修改前的数据：", userMap[1].UserName, userMap[1].Age) // 打印  李四 20
	u1Value.Elem().FieldByName("Age").SetInt(28)
	fmt.Println("修改后的数据：", userMap[1].UserName, userMap[1].Age) // 打印  李四 28
 
	// 通过反射设置map的键值对
	k3 := 2
	u3 := &User{
		UserName: "王五",
		Age:      22,
	}
	mapValue.SetMapIndex(reflect.ValueOf(k3), reflect.ValueOf(u3))
	fmt.Println(userMap[k3].UserName, userMap[k3].Age) // 打印 王五 22
}
```





> 通过反射调用普通方法

```go
func callFunc() {
	valueFunc := reflect.ValueOf(Add)
	fmt.Println(valueFunc.Kind()) // 打印 func
	typeFunc := valueFunc.Type()
	numIn := typeFunc.NumIn() // 获取参数个数
 
	// 定义函数实参
	params := make([]reflect.Value, numIn)
	for i := 0; i < numIn; i++ {
		params[i] = reflect.ValueOf(1)
	}
	// 通过反射调用函数，返回切片Value，返回值中是函数返回结果
	callResult := valueFunc.Call(params)
	for i := 0; i < len(callResult); i++ {
		// 打印返回结果 // 实际开发中这里可以使用类型判断
		fmt.Println(callResult[i])
	}
}
```





> 通过反射调用结构体的成员方法

```go
type User struct {
	UserName string `我是Tag`
	Age      int
	gender   int
	Student  Student
}
 
func (User) Insert(userName string) int {
	return 1
}
 
func callFunc() {
	user := &User{}
	userValue := reflect.ValueOf(user)
	// 通过方法名称获取结构体中的方法
	insertMethod := userValue.MethodByName("Insert")
	// 调用方法，返回函数结果
	callResult := insertMethod.Call([]reflect.Value{reflect.ValueOf("张三")})
	for i := 0; i < len(callResult); i++ {
		// 打印返回结果
		fmt.Println(callResult[i])
	}
}
```



## 案例

### 1. 动态方法调用

```go
package main

import (
    "fmt"
    "reflect"
)

type Animal struct{}

func (a *Animal) Speak(sound string) string {
    return sound
}

func main() {
    a := Animal{}
    method := reflect.ValueOf(&a).MethodByName("Speak")
    args := []reflect.Value{reflect.ValueOf("Woof")}
    result := method.Call(args)
    fmt.Println(result[0].String())
}
```

### 2. 访问和修改变量的值

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    x := 42
    v := reflect.ValueOf(&x).Elem()
    fmt.Println("Before:", v.Int())
    v.SetInt(43)
    fmt.Println("After:", v.Int())
}
```

### 3. 类型检查和断言

```go
package main

import (
    "fmt"
    "reflect"
)

func typeCheck(values ...interface{}) {
    for _, value := range values {
        t := reflect.TypeOf(value)
        fmt.Printf("Value: %v, Type: %v\n", value, t)
    }
}

func main() {
    typeCheck(42, "hello", true, 3.14)
}
```

### 4. 结构体标签（Struct Tags）的处理

```go
package main

import (
    "fmt"
    "reflect"
)

type User struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}

func main() {
    u := User{}
    t := reflect.TypeOf(u)
    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        fmt.Printf("%s: %s\n", field.Name, field.Tag.Get("json"))
    }
}
```

### 5. 构建动态代理或中间件

此案例较为复杂，通常涉及到网络编程和较高级的反射技巧，难以在简短的代码中完全展示。不过，可以想象一个简化的场景，其中利用反射来动态调用指定的处理函数。

```go
package main

import (
    "fmt"
    "reflect"
    "time"
)

// Middleware 动态调用任意函数，并在调用前后执行附加操作
func Middleware(function interface{}, args ...interface{}) []reflect.Value {
    // 确保传入的是函数
    fn := reflect.ValueOf(function)
    if fn.Kind() != reflect.Func {
        panic("Middleware expects a function")
    }

    // 构建函数参数
    in := make([]reflect.Value, len(args))
    for i, arg := range args {
        in[i] = reflect.ValueOf(arg)
    }

    // 调用前的附加操作，例如日志记录
    fmt.Println("Before calling function:", fn.Type().Name(), "at", time.Now().Format(time.RFC3339))

    // 动态调用函数
    result := fn.Call(in)

    // 调用后的附加操作
    fmt.Println("After calling function:", fn.Type().Name(), "at", time.Now().Format(time.RFC3339))

    return result
}

// ExampleFunction 示例函数，用于被Middleware调用
func ExampleFunction(name string, age int) {
    fmt.Printf("Executing ExampleFunction with args: name=%s, age=%d\n", name, age)
}

func main() {
    // 使用中间件调用ExampleFunction
    Middleware(ExampleFunction, "John Doe", 30)
}
```



### 6. 实现通用函数和库

```go
package main

import (
    "fmt"
    "reflect"
)

func printSlice(s interface{}) {
    val := reflect.ValueOf(s)
    if val.Kind() != reflect.Slice {
        fmt.Println("Not a slice")
        return
    }
    for i := 0; i < val.Len(); i++ {
        fmt.Println(val.Index(i))
    }
}

func main() {
    printSlice([]int{1, 2, 3})
    printSlice([]string{"hello", "world"})
}
```

### 7. 类型安全的动态编程

反射的使用使得可以检查类型信息，并据此执行类型安全的操作，例如，动态地处理不同类型的数据，但同时确保在编译时不会违反Go的类型系统。

### 8. 自定义序列化

```go
package main

import (
    "fmt"
    "reflect"
    "strings"
)

func serializeStruct(s interface{}) string {
    val := reflect.ValueOf(s)
    t := val.Type()
    var parts []string
    for i := 0; i < val.NumField(); i++ {
        field := t.Field(i)
        value := val.Field(i)
        part := fmt.Sprintf("%s=%v", field.Name, value)
        parts = append(parts, part)
    }
    return strings.Join(parts, ", ")
}

type MyStruct struct {
    Name  string
    Value int
}

func main() {
    s := MyStruct{Name: "Test", Value: 42}
    fmt.Println(serializeStruct(s))
}
```







# runtime







