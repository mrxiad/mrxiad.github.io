---
title: go语法
date: 2024-4-8 22:40:40
tags: go
categories: 
- go
- 字符串操作
---





# `stringBuilder`

```go
package main

import "fmt"
import "bytes"

func main() {
    str01 := "Hello,"
    str02 := "world!"

    // 声明字节缓冲
    var stringBuilder bytes.Buffer

    // 把字符串写入缓冲
    stringBuilder.WriteString(str01)
    stringBuilder.WriteString(str02)

    // 将缓冲以字符串形式输出
    fmt.Println(stringBuilder.String())
}

// 结果如下：
Hello,world!
```





# UTF-8字符串长度

```go
package main

import "fmt"
import "unicode/utf8"

func main() {
    str02 := "你好"
    fmt.Println(len(str02))       // 6
    fmt.Println(utf8.RuneCountInString(str02))   // 2
    fmt.Println(utf8.RuneCountInString("你好,world"))  // 8
}
```







# 大小写转换

```go
str := "Hello, Go!"
upper := strings.ToUpper(str)
lower := strings.ToLower(str)
fmt.Println(upper) // 输出 "HELLO, GO!"
fmt.Println(lower) // 输出 "hello, go!"
```





# 匹配查找计数



## 前后缀匹配

```go
str := "Hello, Go!"
hasPrefix := strings.HasPrefix(str, "Hello")
hasSuffix := strings.HasSuffix(str, "Go!")
fmt.Println(hasPrefix) // 输出 true
fmt.Println(hasSuffix) // 输出 true
```





## 包含判断和计数

- `strings.Contains()`：判断字符串是否包含指定的子串。
- `strings.Count()`：计算子串在字符串中出现的次数

```go
str := "apple, banana, orange, apple"
contains := strings.Contains(str, "banana")
count := strings.Count(str, "apple")
fmt.Println(contains) // 输出 true
fmt.Println(count)    // 输出 2
```





## 字符串索引和查找

- `strings.Index()`：查找子串第一次出现的索引。
- `strings.LastIndex()`：查找子串最后一次出现的索引。

```go
str := "apple, banana, orange, apple"
firstIndex := strings.Index(str, "banana")
lastIndex := strings.LastIndex(str, "apple")
fmt.Println(firstIndex) // 输出 7
fmt.Println(lastIndex)  // 输出 25
```



# 修剪和替换



## 修剪

- `strings.TrimSpace()`：去除字符串两端的空白字符。
- `strings.Trim()`：去除字符串指定字符集合。
- `TrimRight()`：去除字符串尾部的指定字符集合。
- `TrimLeft()`函数：去除字符串头部的指定字符集合。

```go
str := "   Hello, Go!   "


trimmed := strings.TrimSpace(str)
fmt.Println(trimmed)      // 输出 "Hello, Go!"


trimmedCustom := strings.Trim(str, " H")
fmt.Println(trimmedCustom) // 输出 "ello, Go!"


trimmedRight := strings.TrimRight(str, " ")
fmt.Println(trimmedRight) // 输出 "   Hello, Go!"


trimmedLeft := strings.TrimLeft(str, " ")
fmt.Println(trimmedLeft) // 输出 "Hello, Go!   "
```



- `TrimPrefix`函数：`TrimPrefix`函数用于去除字符串的指定前缀。
- `TrimSuffix`函数：`TrimSuffix`函数用于去除字符串的指定后缀。

```go
str := "Hello, Go!"
trimmedPrefix := strings.TrimPrefix(str, "Hello, ")
fmt.Println(trimmedPrefix) // 输出 "Go!"

str := "Hello, Go!"
trimmedSuffix := strings.TrimSuffix(str, ", Go!")
fmt.Println(trimmedSuffix) // 输出 "Hello"
```



## 替换

- `strings.Replace()`：将字符串中的某个子串替换为另一个子串。

```go
str := "Hello, World! Hello, Go!"
newStr := strings.Replace(str, "Hello", "Hi", -1)
fmt.Println(newStr) // 输出 "Hi, World! Hi, Go!"
```





# 拆分和连接

## 字符串拆分

- `strings.SplitN()`：按指定的分隔符拆分字符串，指定拆分的次数。
- `strings.SplitAfterN()`：按指定的分隔符拆分字符串，保留分隔符。

```go
str := "apple, banana, orange, apple"
splitN := strings.SplitN(str, ", ", 2)
splitAfterN := strings.SplitAfterN(str, ", ", 2)
fmt.Println(splitN)       // 输出 ["apple", "banana, orange, apple"]
fmt.Println(splitAfterN) // 输出 ["apple, ", "banana, orange, apple"]
```







## 字符串连接

- `strings.Join()`：将字符串数组连接成一个字符串，用指定的分隔符连接

```go
fruits := []string{"apple", "banana", "orange"}
str := strings.Join(fruits, ", ")
fmt.Println(str) // 输出 "apple, banana, orange"
```





# 重复和计数

## 重复

- `strings.Repeat()`：将字符串重复指定次数。

```go
str := "Go!"
repeated := strings.Repeat(str, 3)
fmt.Println(repeated) // 输出 "Go!Go!Go!"
```



## 字符串计数

- `strings.Count()`：计算子串在字符串中出现的次数

```go
str := "apple, banana, orange, apple"
count := strings.Count(str, "apple")
fmt.Println(count) // 输出 2
```

