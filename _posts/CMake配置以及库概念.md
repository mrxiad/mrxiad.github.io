---
title: CMake配置+库概念
date: 2023-11-21 14:20:56
tags: 技术
categories: CMake
---





本文介绍CMake的常用命令以及库的相关概念



# 安装Cmake

在ubuntu18.04下安装cmake

```bash
sudo apt install cmake
```

安装后查看版本

```bash
cmake -version
```





# CMake编写



[【C++】Cmake使用教程（看这一篇就够了）_隐居的遮天恶鬼的博客-CSDN博客](https://blog.csdn.net/weixin_43717839/article/details/128032486?ops_request_misc=%7B%22request%5Fid%22%3A%22170054364016800222888865%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=170054364016800222888865&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-128032486-null-null.142^v96^pc_search_result_base3&utm_term=cmake&spm=1018.2226.3001.4187)

[【C++】静态库和动态库文件的生成和使用_c++ 生成静态库和动态库命令-CSDN博客](https://blog.csdn.net/weixin_43717839/article/details/127991174?spm=1001.2014.3001.5502)



## 标准项目结构

```makefile
YourProject/
│
├── CMakeLists.txt       # 顶层 CMake 配置文件
│
├── src/                 # 源代码目录
│   ├── CMakeLists.txt   # 源代码目录的 CMake 配置文件
│   ├── main.cpp         # 主源文件
│   └── ...              # 其他源文件和头文件
│
├── include/             # 头文件目录
│   └── ...              # 公共头文件
│
├── libs/                # 第三方库或子项目
│   ├── lib1/            # 第三方库1
│   └── lib2/            # 第三方库2
│
├── tests/               # 测试代码
│   ├── CMakeLists.txt   # 测试代码的 CMake 配置文件
│   └── ...              # 测试源文件
│
├── doc/                 # 文档
│   └── ...
│
├── build/               # 构建目录（通常在 .gitignore 中排除）
│
└── bin/                 # 可执行文件目录（如果需要）

```





## 基础配置

```cmake
#指定最低版本号
cmake_minimum_required (VERSION 2.8)

#指定项目名称
project (demo)

#生成可执行文件
add_executable(main main.c)
```



## （1）单目录源文件

### 方法1

```cmake
aux_source_directory(dir var)
```

dir是你的`源文件目录`，var是一个`变量名`，用于表示dir目录中的源文件`列表`



 **示例**

```cmake
cmake_minimum_required (VERSION 2.8)

project (demo)

#将当前目录下的文件赋值给SRC_LIST
aux_source_directory(. SRC_LIST)

#注意变量名的使用方法
add_executable(main ${SRC_LIST})
```



### 方法2

**示例**

```cmake
cmake_minimum_required (VERSION 2.8)

project (demo)

# 设置多个源文件给SRC_LIST列表
set( SRC_LIST
	 ./main.c
	 ./testFunc1.c
	 ./testFunc.c)

add_executable(main ${SRC_LIST})
```





## （2）多目录（源文件+头文件）

**示例**

```cmake
# 指定CMake的最小版本要求
cmake_minimum_required (VERSION 2.8)

# 设置项目名称
project (YourProjectName)

# 添加头文件搜索路径
include_directories (path/to/headers1 path/to/headers2)

# 自动收集第一个源代码（目录）中的源文件
aux_source_directory (path/to/source1 SRC_LIST1)

# 自动收集第二个源代码（目录）中的源文件
aux_source_directory (path/to/source2 SRC_LIST2)

# 指定生成的可执行文件和用于构建它的源文件
add_executable (Executable_name main.c ${SRC_LIST1} ${SRC_LIST2})
```



## (3)添加子目录

使用命令 `add_subdirectory (src)`

```cmake
cmake_minimum_required (VERSION 2.8)

project (demo)

add_subdirectory (src)  #执行到这句后，去处理src的CMakeLists.txt文件
```

这里指定src目录下存放了源文件，当执行cmake时，就会进入src目录下去找src目录下的CMakeLists.txt，所以在src目录下也建立一个CMakeLists.txt，内容如下:

```cmake
aux_source_directory (. SRC_LIST)

include_directories (../include)


add_executable (main ${SRC_LIST})

#设置输出的exe路径
set (EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
```



## （4）动态库和静态库的编译控制（待完成）





## （5）条件编译（待完成）









## 总结（待完成2）

### 用到的指令（待完成）

```cmake
# 指定CMake的最小版本要求
cmake_minimum_required (VERSION 2.8)

# 设置项目名称
project (YourProjectName)

# 添加头文件搜索路径
include_directories (path/to/headers1 path/to/headers2)

# 自动收集第一个源代码（目录）中的源文件
aux_source_directory (path/to/source1 SRC_LIST1)

# 自动收集第二个源代码（目录）中的源文件
aux_source_directory (path/to/source2 SRC_LIST2)

# 指定可执行文件输出目录
set (EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)

# 指定库文件输出路径
set (LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)

# 添加子目录（假设子目录包含CMakeLists.txt和额外的源代码）
add_subdirectory (path/to/subdirectory1)
add_subdirectory (path/to/subdirectory2)

# 指定生成的可执行文件和用于构建它的源文件
add_executable (Executable_name main.c ${SRC_LIST1} ${SRC_LIST2})
```





### 易错点

1. **执行顺序**：
	- CMake 的命令执行是有顺序的。例如，`set_target_properties` 应该在相关的目标被定义**后**使用。
2. **源文件列表**：
	- 在使用 `aux_source_directory` 时，确保该命令收集的源文件是您实际需要编译的。该命令不会递归收集子目录中的文件。
3. **库和可执行文件的输出路径**：
	- `set(EXECUTABLE_OUTPUT_PATH ...)` 或 `set(LIBRARY_OUTPUT_PATH ...)` 应在相关的 `add_executable` 或 `add_library` **之前**设置。(注意第一条是之后)
4. **头文件路径**：
	- 使用 `include_directories` 时，确保路径正确，并且是您需要的头文件所在的路径。
5. **子目录处理**：
	- 当使用 `add_subdirectory` 时，确保每个子目录都有自己的 `CMakeLists.txt` 文件。
6. **库的链接**：
	- 当链接库时，确保正确指定了库的类型（静态或动态）和路径





当然可以，我会将有关如何使用库的信息加入您的笔记，并重新排版。以下是更新后的笔记，整理成Markdown格式：

------

# 库的概念和使用

## 1. 库的基本概念

库是一组预先编译好的方法的集合，用于提供开箱即用的变量、函数或类。它是计算机上的一类文件，主要分为静态库和动态库。

### 静态库与动态库

- **静态库**：程序在编译时会将静态库的内容复制到可执行文件中，导致每个使用静态库的程序都有一份库的副本，可能会导致空间浪费。
- **动态库**：程序执行时链接到动态库，所有使用同一动态库的程序都会共享这一份库文件，节省内存空间。

### 文件扩展名

- Windows系统：
	- 静态库：`.lib`
	- 动态库：`.dll` (Dynamic-Link Libraries)
- Linux系统：
	- 静态库：`.a`
	- 动态库：`.so` (Shared Object)

## 2. 创建库文件

### 创建静态库

1. **编译多个源文件**：首先，将每个 `.c` 文件编译成 `.o` 文件。

	```bash
	gcc -c file1.c
	gcc -c file2.c
	gcc -c file3.c
	```

	这将生成 `file1.o`、`file2.o` 和 `file3.o` 等文件。

2. **合并 `.o` 文件到静态库**：使用 `ar` 工具将所有的 `.o` 文件合并成一个静态库（`.a` 文件）。

	```bash
	ar -crv libmylibrary.a file1.o file2.o file3.o
	```

	`libmylibrary.a` 就是生成的静态库文件。

	------

	

### 创建动态库

1. **编译多个源文件**：和创建静态库一样，先编译每个 `.c` 文件。

	```bash
	gcc -c file1.c
	gcc -c file2.c
	gcc -c file3.c
	```

2. **合并 `.o` 文件到动态库**：使用 `gcc` 与 `-shared` 选项将所有 `.o` 文件合并成一个动态库（`.so` 文件）。

	```bash
	gcc -shared -o libmylibrary.so file1.o file2.o file3.o
	```

	`libmylibrary.so` 就是生成的动态库文件。

	------

	

## 3. 使用库

### 链接静态库

在编译程序时，将静态库链接到您的可执行文件中：

```bash
gcc -o myprogram myprogram.c -L[path_to_lib] -lmylibrary
```

### 链接动态库

链接动态库的过程类似，但您可能需要确保运行时动态库的路径被正确设置：

```basj
gcc -o myprogram myprogram.c -L[path_to_lib] -lmylibrary
```

## 4. 链接优先级

当静态库和动态库同名时，在Linux系统中，gcc命令会优先链接到动态库。默认情况下，会链接到系统的`/lib`或`/usr/lib`目录中的动态库。

## 5. Linux系统库的存储位置

Linux系统中，库文件一般存放在以下位置：

- `/lib`
- `/usr/lib`

在使用库时，确保包含了所有必要的头文件，并且库的路径被正确指定。静态库将在编译时被整合到您的可执行文件中，而动态库则在程序运行时被加载。