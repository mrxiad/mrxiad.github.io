---
title: CMake配置+库概念(待完成)
date: 2024-01-03 22:22:30
tags: 技术
categories:
- 工程
- Cmake
---





本文介绍CMake的常用命令以及库的相关概念

项目地址：[cmake教程](https://github.com/mrxiad/Cmake_project_template)



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



## 本文项目结构

```makefile
MyProject/
|-- CMakeLists.txt          # 主CMake文件
|-- external/               # 外部依赖目录
|   |-- CMakeLists.txt
|   `-- libExternal         # 模拟的外部库
|       |-- include/        # 外部库的头文件
|       |   `-- libExternal.h
|       `-- src/            # 外部库的源文件
|           `-- libExternal.cpp
|-- app/                    # 应用程序目录
|   |-- CMakeLists.txt
|   `-- src/
|       `-- main.cpp
|-- lib/                    # 库文件目录
|   |-- libA/               # 静态库A
|   |   |-- CMakeLists.txt
|   |   `-- src/
|   |       `-- libA.cpp
|   `-- libB/               # 动态库B
|       |-- CMakeLists.txt
|       `-- src/
|           `-- libB.cpp
|-- include/                # 公共头文件目录
|   |-- libA/
|   |   `-- libA.h
|   `-- libB/
|       `-- libB.h
|-- tests/                  # 测试目录
|   |-- CMakeLists.txt
|   `-- testA/
|       |-- CMakeLists.txt
|       `-- src/
|           `-- testA.cpp
|-- docs/                   # 文档目录
|-- README.md
`-- .gitignore

```



## 需要明确一些点

1. `include”head.h”`仅仅是将文件内容复制，所以一个项目可以**完全没有**头文件

2. 如果指定库文件（或者可执行文件）的头文件**搜索路径**，则首先在库文件搜索路径下找，然后在当前cpp文件目录下，然后在编译器的默认搜索路径中寻找

3. CMake是一个跨平台的自动化构建系统，主要用于生成Makefile，所以一般cmake不会报错，在make的时候会报错

4. **无法解析的外部命令**

	- 函数仅在头文件中声明，而没有在任何编译单元（.cpp文件）中实现。
	- 相应的库文件没有被正确链接到您的项目中。
	- 如果涉及动态库，可能是动态库没有被正确安装或找不到

5.  **编译和链接的关系**

	每个源文件（.cpp）可以独立编译，生成对象文件。链接阶段将这些对象文件组合成最终的可执行文件或库文件。如果只有函数声明没有实现，编译时不会报错，但链接时会报错，因为链接器找不到这些函数的定义。并且链接器会自动处理依赖关系，不需要指定对象文件的链接顺序。

6. **动态库链接的代码执行**

	当您的程序链接了动态库，运行时，操作系统会加载这些库并解析所需的符号。如果您删除了动态库，程序可能无法运行，因为它找不到必要的函数实现。动态库的代码不是写入可执行文件中，而是在运行时动态加载的。

7. **库文件和头文件**

	库文件必须包含函数的定义，但**不一定**要包含头文件。头文件通常包含函数声明，使得其他源文件能够知道这些函数的存在。即使不包含头文件，只要函数在库内部定义，链接时就不会报错。

8. **动态库搜索**，

	可执行文件本身不包含库代码，而是包含对库函数的引用。在运行时，操作系统的动态链接器负责找到并加载这些动态库。这通常是通过以下方式实现的：

	1. **链接时信息**: 当可执行文件被创建时，链接器会在可执行文件中存储有关它依赖的动态库的信息（如库的名称和版本）。
	2. **运行时搜索**: 在程序启动时，动态链接器会根据这些信息查找并加载必要的动态库。库的搜索可以基于多个因素，包括操作系统的库搜索路径、环境变量（如 `LD_LIBRARY_PATH` 在Linux上）等。





# cmake知识点

## 一些常量

1. ### 项目和源代码相关

	```cmake
	# 获取包含顶层CMakeLists.txt的目录的路径
	set(TOP_LEVEL_SOURCE_DIR ${CMAKE_SOURCE_DIR})
	
	# 获取当前处理的CMakeLists.txt的目录的路径
	set(CURRENT_LIST_DIR ${CMAKE_CURRENT_SOURCE_DIR})
	
	# 获取最近通过project()命令定义的项目的源目录(根cmake目录)
	set(LAST_PROJECT_SOURCE_DIR ${PROJECT_SOURCE_DIR})
	
	# 获取包含当前正在处理的列表文件的目录
	set(CURRENT_LIST_FILE_DIR ${CMAKE_CURRENT_LIST_DIR})
	```

2. ### 构建和输出相关

	```cmake
	# 获取顶层构建目录的路径(build)
	set(TOP_LEVEL_BINARY_DIR ${CMAKE_BINARY_DIR})
	
	# 获取当前处理的CMakeLists.txt对应的构建目录的路径
	set(CURRENT_BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR})
	
	# 设置可执行文件的输出目录
	set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
	
	# 设置库文件（动态）的输出目录
	set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
	
	# 设置静态库的输出目录
	set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/archive)
	```

3. ### 系统和平台相关

	```cmake
	# 获取系统名称，例如Linux、Windows
	set(SYSTEM_NAME ${CMAKE_SYSTEM_NAME})
	
	# 获取系统处理器，例如x86_64、AMD64
	set(SYSTEM_PROCESSOR ${CMAKE_SYSTEM_PROCESSOR})
	```

4. ### C++ 和编译器相关

	```cmake
	# 设置C++标准
	set(CMAKE_CXX_STANDARD 17)
	set(CMAKE_CXX_STANDARD_REQUIRED ON)
	
	# 设置C++编译器的标志
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")
	
	# 获取编译器的完整路径
	set(COMPILER_PATH ${CMAKE_CXX_COMPILER})
	```

> 注意：
> CMAKE_BINARY_DIR是执行cmake命令的目录，外部构建的时候是build目录
> PROJECT_SOURCE_DIR是项目的根CMakeLists.txt所在目录
> CMAKE_CURRENT_SOURCE_DIR是当前CMakeLists.txt所在目录

### 区分预定义变量和自定义变量

- **预定义变量**：这些是CMake系统提供的变量，通常以 `CMAKE_` 开头。它们用于获取关于构建环境和项目配置的信息。
- **自定义变量**：您可以通过 `set()` 命令创建自己的变量。您可以使用这些变量来存储自己的数据，或者修改和存储预定义变量的值。





## 构建camke工程的步骤

1. 首先要在每个目录下创建CMakeLists.txt文件

2. 嵌套的目录需要使用`add_subdirectory`调用

3. 动态库链接静态库需要添加**-fPIC编译选项**

	```cma
	# 添加-fPIC编译选项,libA是静态库名
	set_property(TARGET libA PROPERTY POSITION_INDEPENDENT_CODE ON)
	```



## 常用命令大全



1. ### 设置全局c++标准

	```cmake
	# 设置全局C++标准
	set(CMAKE_CXX_STANDARD 17)
	set(CMAKE_CXX_STANDARD_REQUIRED ON)
	```

2. ### 指定文件生成目录

	```cmake
	# 设置可执行文件和库文件的输出目录
	set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/bin)
	set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/libs)
	```

3. ### 添加子目录

	```cmake
	# 添加子目录,括号内是子目录名称
	add_subdirectory(external)
	add_subdirectory(lib)
	add_subdirectory(app)
	add_subdirectory(tests)
	```

4. ### 生成库文件

	```cmake
	add_library(libname SHARED src/mylib.cpp)  #生成动态库文件libname
	add_library(libname STATIC src/mylib.cpp)  #生成静态库文件libname
	```

5. ### 为目标添加头文件搜索路径

	```cmake
	target_include_directories(libA PUBLIC ${PROJECT_SOURCE_DIR}/include/libA)
	```

6. ### 为目标链接库

	```cmake
	target_link_libraries(myApp PRIVATE libA libB libExternal)
	#myqpp是可执行文件（也可以是库文件）
	#libA，libB，libExternal是库文件
	```





# 添加外部库实战

## boost库

1. 执行命令	

	```bash
	sudo apt-get update
	sudo apt-get install libboost-all-dev
	```

2. 在CMake中找到Boost库

	```cmake
	cmake_minimum_required(VERSION 3.15)
	project(MyComplexProject VERSION 1.0)
	
	# 设置全局C++标准
	set(CMAKE_CXX_STANDARD 17)
	set(CMAKE_CXX_STANDARD_REQUIRED ON)
	
	#设置可执行文件和库的输出目录
	set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/bin)
	set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/libs)#动态库
	set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/archive)#静态库
	
	# 找到Boost库
	find_package(Boost 1.65 REQUIRED COMPONENTS system filesystem)
	
	# 如果找到了Boost，包括Boost的头文件目录
	if(Boost_FOUND)
	    message(STATUS "找到了Boost库 ${Boost_VERSION_MAJOR}.${Boost_VERSION_MINOR}.${Boost_VERSION_PATCH} in ${Boost_INCLUDE_DIRS}")
	    include_directories(${Boost_INCLUDE_DIRS})
	endif()
	
	# 添加子目录
	add_subdirectory(external)
	add_subdirectory(lib)
	add_subdirectory(app)
	add_subdirectory(tests)
	```

3.  链接Boost库

	```cmake
	add_executable(myApp src/main.cpp)
	# 链接Boost库
	if(Boost_FOUND)
	    message("链接成功：Boost_INCLUDE_DIRS: ${Boost_INCLUDE_DIRS}")
	    target_link_libraries(myApp PRIVATE ${Boost_LIBRARIES})
	endif()
	target_link_libraries(myApp PRIVATE libA libB libExternal)
	target_include_directories(myApp PRIVATE 
	    ${PROJECT_SOURCE_DIR}/include/libA 
	    ${PROJECT_SOURCE_DIR}/include/libB
	)
	```

	





## opencv库









## Json库



