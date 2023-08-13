---
title: "Cmake"
date: 2023-08-11T21:43:12+08:00
lastmod: 2023-08-11T21:43:12+08:00
author: ["chx9"]
keywords: 
- 
categories: # 没有分类界面可以不填写
- 
tags: # 标签
- 
description: ""
weight:
slug: ""
draft: false # 是否为草稿
comments: true # 本页面是否显示评论
reward: false # 打赏
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "" #图片路径例如：posts/tech/123/123.png
    zoom: # 图片大小，例如填写 50% 表示原图像的一半大小
    caption: "" #图片底部描述
    alt: ""
    relative: false
---



# CMake介绍
CMake是一个跨平台的构建工具，它能够帮助开发者管理大型项目，处理各种依赖关系，实现多平台、多编译器下的统一构建。使用CMake，开发者可以摆脱繁琐的手动配置环境，专注于核心的代码实现，从而大大提高开发效率。

# 第一个CMake文件
在一个C++项目中，我们通常需要编写一个名为CMakeLists.txt的文件来指导CMake如何构建项目。在这个文件中，我们可以定义项目的源文件、头文件、依赖关系等信息。

让我们一起来创建一个简单的CMakeLists.txt文件。
```c++
# CMake版本要求
cmake_minimum_required(VERSION 3.10)

# 项目名称和版本
project(HelloWorld VERSION 1.0.0)

# 添加一个可执行文件
add_executable(${PROJECT_NAME} main.cpp)
```

以下是每行代码的简介：

- `cmake_minimum_required(VERSION 3.10)`: 这一行定义了要构建此项目所需的最低CMake版本。如果使用的CMake版本低于此版本，将会提示错误。
- `project(HelloWorld VERSION 1.0.0)`: 这一行定义了项目的名称（此处为`HelloWorld`）和版本号（此处为`1.0.0`）。`${PROJECT_NAME}`变量将被设置为"HelloWorld"，在后续的命令中可以使用。
- `add_executable(${PROJECT_NAME} main.cpp)`: 这一行添加了一个可执行文件。第一个参数是生成的可执行文件的名称，这里我们用之前定义的项目名变量`${PROJECT_NAME}`作为输出的可执行文件名；后面的参数是构成这个可执行文件的源文件，这里只有一个`main.cpp`。



# CMake中的变量定义

在CMake中，我们可以通过 `set` 命令来定义一个变量。以下是一些基本示例：

```cmake
set(VAR_NAME "Hello, CMake!")
```

## 定义一个列表变量

```cmake
set(SOURCE_FILES main.cpp util.cpp)
```

这条命令定义了一个名为`SOURCE_FILES`的变量，它包含两个元素：`main.cpp`和`util.cpp`。列表变量在处理源文件或者库文件时非常有用。

## 使用变量

在CMake中，我们可以使用 `${}` 来引用一个变量。例如：

```cmake
set(SOURCE_FILES main.cpp util.cpp)
add_executable(${PROJECT_NAME} ${SOURCE_FILES})
```

这里，`${SOURCE_FILES}` 将会被替换为 `main.cpp util.cpp`。

## 修改变量

我们也可以修改已经存在的变量。例如下面的命令将向`SOURCE_FILES`列表添加新的元素：

```cmake
set(SOURCE_FILES main.cpp util.cpp)
list(APPEND SOURCE_FILES other.cpp) # Now SOURCE_FILES contains main.cpp, util.cpp and other.cpp
```

以上就是在CMake中如何定义和使用变量的基本方法。正确地使用变量可以让你的CMakeLists.txt更加清晰和灵活。

## 指定C++版本

在CMake中，你可以使用`set`命令和`CMAKE_CXX_STANDARD`变量来指定C++的版本。以下是一个示例：

```cmake
# 设置C++标准为C++14
set(CMAKE_CXX_STANDARD 14)

# 我们也可以要求编译器必须支持这一标准：
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 如果你不希望编译器使用比指定版本更新的标准，你可以设置下面的变量为ON：
set(CMAKE_CXX_EXTENSIONS OFF)
```

其中，

- `CMAKE_CXX_STANDARD` 变量用于设定项目中所使用的C++标准。常见的值有11、14、17等，对应C++11、C++14、C++17标准。
- `CMAKE_CXX_STANDARD_REQUIRED` 变量决定是否强制要求编译器支持设定的C++标准。如果设置为 `ON`，那么当编译器不支持设定的标准时，CMake将报错。
- `CMAKE_CXX_EXTENSIONS` 变量用于控制编译器是否允许使用比设定的C++标准更新的语言扩展。如果设置为 `OFF`，编译器将只使用标准C++，而不包含任何特定编译器的扩展。

注意以上的这些变量设置必须放在项目声明 (`project()`) 之后，否则可能不会生效。

这种方式适用于CMake版本3.1及以上，如果你使用的CMake版本较低，可能需要另找其他方式设定C++版本。

## 指定输出路径

在CMake中，你可以设置 `CMAKE_RUNTIME_OUTPUT_DIRECTORY` 变量来指定可执行文件的输出路径，设置 `CMAKE_LIBRARY_OUTPUT_DIRECTORY` 和 `CMAKE_ARCHIVE_OUTPUT_DIRECTORY` 变量来指定库文件的输出路径。以下是一个示例：

```cmake
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
```

这段代码将会把所有的可执行文件输出到构建目录下的`bin`文件夹中，所有的库文件（包括动态库和静态库）输出到构建目录下的`lib`文件夹中。

其中，`${CMAKE_BINARY_DIR}`变量表示的是构建目录（即你调用`cmake`命令的那个目录）。如果你希望使用其他目录作为输出路径，只需要替换掉对应的目录即可。

注意，这些设置通常放在顶层的 `CMakeLists.txt` 文件中，以确保它们对整个项目生效。

# 搜索文件

## file

## 搜索指定子目录下的所有 `.cpp` 文件

```cmake
file(GLOB SOURCES "src/*.cpp")
```

这个命令将会搜索当前源目录下的 `src` 子目录中所有的 `.cpp` 文件，并将结果存储到 `SOURCES` 变量中。

## 在指定子目录中递归搜索 `.cpp` 文件

```cmake
file(GLOB_RECURSE SOURCES "src/*.cpp")
```

`GLOB_RECURSE` 可以递归地搜索目录下的文件，上面的例子中搜索了 `include` 目录下的所有 `.h` 文件。

```cmake
file(GLOB_RECURSE HEADER_FILES "${CMAKE_SOURCE_DIR}/include/*.h")
```

##  `aux_source_directory` 搜索源文件

`aux_source_directory` 命令是一个更专注于构建源文件的工具。它会在指定目录中搜索所有源文件，并将它们添加到一个变量中，然后可以将该变量用于构建目标。

```cmake
aux_source_directory(${CMAKE_SOURCE_DIR}/src SOURCE_FILES)
add_executable(MyApp ${SOURCE_FILES})
```

上面的例子中，`aux_source_directory` 命令会将 `src` 目录中的所有源文件添加到 `SOURCE_FILES` 变量中，然后可以将该变量用于构建目标。

# cmake指定头文件目录

在 CMake 中，要指定头文件目录，你可以使用 `include_directories()` 命令或 `target_include_directories()` 命令。这些命令允许你向 CMake 表示哪些目录应该被添加到头文件搜索路径中。

下面是两种常用的方式：

1. **使用 `include_directories()` 命令**：

   这个命令会将指定的目录添加到整个项目的头文件搜索路径中。在项目的 CMakeLists.txt 文件中使用它。

   ```cmake
   include_directories(${CMAKE_SOURCE_DIR}/include)
   include_directories(${CMAKE_SOURCE_DIR}/other_include)
   ```

   或者，你可以将多个目录合并成一个列表：

   ```
   set(INCLUDE_DIRS
       ${CMAKE_SOURCE_DIR}/include
       ${CMAKE_SOURCE_DIR}/other_include
   )
   include_directories(${INCLUDE_DIRS})
   ```

2. **使用 `target_include_directories()` 命令**：

   这个命令会将指定的目录添加到特定目标（例如可执行文件或库）的头文件搜索路径中。这是一种更推荐的方式，因为它将目录限定在特定目标，避免了全局的头文件搜索路径混乱。

   ```cmake
   add_executable(MyApp ${SOURCES})
   target_include_directories(MyApp PRIVATE ${CMAKE_SOURCE_DIR}/include)
   ```

   如果你在多个目标中使用同样的头文件路径，也可以将路径设置为变量：

   ```cmake
   set(INCLUDE_DIRS ${CMAKE_SOURCE_DIR}/include)
   add_executable(MyApp ${SOURCES})
   target_include_directories(MyApp PRIVATE ${INCLUDE_DIRS})
   ```

# CMake制作库

**制作静态库**：

用`add_library` ，

- 静态库使用STATIC，默认是STATIC可以忽略

- 动态库使用SHARED

```Cmake
add_library(MyStaticLib STATIC
    source_file1.cpp
    source_file2.cpp
)
```



在 CMake 中制作静态库（也称为静态链接库）通常涉及创建一个包含编译后目标代码的归档文件。

# CMake使用库


当使用CMake构建C++项目时，连接（链接）库是将代码与外部库整合在一起的关键部分。以下是四个关于连接库的CMake命令，以及它们的应用场景和区别：

1. **`link_libraries`**：
   - 作用：指定全局性的连接库，适用于整个项目。
   - 用法：`link_libraries(library_name1 library_name2 ...)`
   - 示例：`link_libraries(SDL2 SDL2_image)`
   - 适用场景：在项目中多个目标都需要链接同样的库时，使用这个命令。
2. **`link_directories`**：
   - 作用：定义链接库时的搜索路径。
   - 用法：`link_directories(directory_path)`
   - 示例：`link_directories(/path/to/libs)`
   - 适用场景：当你使用的库不在默认搜索路径中，需要指定额外的库搜索路径时，使用这个命令。
3. **`target_link_libraries`**：
   - 作用：将指定目标与一个或多个库链接。
   - 用法：`target_link_libraries(target_name library_name1 library_name2 ...)`
   - 示例：`target_link_libraries(my_app SDL2)`
   - 适用场景：为特定目标（可执行文件或库）指定它所需的连接库。
4. **`target_link_directories`**：
   - 作用：为特定目标定义连接库的搜索路径。
   - 用法：`target_link_directories(target_name PRIVATE|PUBLIC|INTERFACE directory_path)`
   - 示例：`target_link_directories(my_app PRIVATE /path/to/libs)`
   - 适用场景：在特定目标上为连接库的搜索路径提供定制化配置，以确保链接器能够找到所需的库文件。

## target_link_directories的权限

`target_link_directories`命令用于为特定目标指定连接库时的搜索路径，具有三种权限：`PRIVATE`、`PUBLIC`和`INTERFACE`。

1. `PRIVATE`权限仅适用于目标本身，不传递给依赖目标。
2. `PUBLIC`权限适用于目标和依赖目标。
3. `INTERFACE`权限适用于依赖目标，不影响目标本身。

# 字符串操作

## set命令

在CMake中，你可以使用`set`和`list`等命令来进行字符串操作和列表操作。以下是一些常见的操作及其示例：

1. **字符串操作**：

   - 设置变量：使用`set`命令来创建或设置变量。

     ```cmake
     set(my_string "Hello, CMake!")
     ```

   - 字符串拼接：使用`${}`来访问变量的值，进行字符串拼接。

     ```cmake
     set(greeting "Hello")
     set(name "John")
     set(message "${greeting}, ${name}!")
     ```

   - 子字符串截取：使用`string(SUBSTRING ...)`来获取子字符串。

     ```cmake
     set(full_string "CMake is awesome!")
     string(SUBSTRING ${full_string} 0 5 sub_string)
     ```

2. **列表操作**：

   - 创建列表：使用`set`命令创建一个包含多个元素的列表。

     ```cmake
     set(my_list_item1 "apple")
     set(my_list_item2 "banana")
     set(fruit_list ${my_list_item1} ${my_list_item2})
     ```

   - 访问列表元素：使用`${}`来访问列表中的元素。

     ```cmake
     set(fruits "apple" "banana" "cherry")
     message("First fruit: ${fruits}")
     ```

   - 列表追加：使用`list(APPEND ...)`将元素添加到列表。

     ```cmake
     set(colors "red" "green")
     list(APPEND colors "blue")
     ```

   - 列表长度：使用`list(LENGTH ...)`获取列表的长度。

     ```cmake
     set(numbers 1 2 3)
     list(LENGTH numbers length)
     ```

   - 列表循环：使用`foreach`循环遍历列表元素。

     ```cmake
     set(shapes "circle" "square" "triangle")
     foreach(shape ${shapes})
         message("Shape: ${shape}")
     endforeach()
     ```

这些示例演示了CMake中的基本字符串操作和列表操作。你可以根据具体需求使用这些命令来进行字符串和列表的处理。

## list命令

在 CMake 中，`list` 命令是用于操作列表（数组）的命令。它允许你对列表进行添加、删除、查询等操作。CMake 中的列表类似于编程中的数组，可以包含一系列的元素。`list` 命令提供了多个子命令，用于操作列表的不同方面。

以下是一些常用的 `list` 命令及其子命令：

1. **`list(APPEND list_name element1 element2 ...)`**：

   将一个或多个元素追加到列表的末尾。

   ```cmake
   list(APPEND MY_LIST "item1" "item2")
   ```

2. **`list(INSERT list_name index element)`**：

   在列表的指定位置插入一个元素。

   ```cmake
   list(INSERT MY_LIST 2 "new_item")
   ```

3. **`list(REMOVE_ITEM list_name element)`**：

   从列表中移除指定的元素。

   ```cmake
   list(REMOVE_ITEM MY_LIST "item_to_remove")
   ```

4. **`list(REMOVE_AT list_name index)`**：

   从列表中移除指定索引位置的元素。

   ```cmake
   list(REMOVE_AT MY_LIST 1)
   ```

5. **`list(GET list_name index variable_name)`**：

   获取列表中指定索引位置的元素，并将其存储在一个变量中。

   ```cmake
   list(GET MY_LIST 0 FIRST_ITEM)
   ```

6. **`list(LENGTH list_name length_variable)`**：

   获取列表的长度（元素个数），并将其存储在一个变量中。

   ```cmake
   list(LENGTH MY_LIST LIST_LENGTH)
   ```

7. **`list(POP_FRONT list_name variable_name)`**：

   弹出列表的第一个元素，并将其存储在一个变量中。

   ```cmake
   list(POP_FRONT MY_LIST FIRST_ITEM)
   ```

8. **`list(REVERSE list_name)`**：

   反转列表中的元素顺序。

   ```cmake
   list(REVERSE MY_LIST)
   ```

这些命令可以用来操作列表，帮助你在 CMake 中更灵活地管理数据。列表在构建过程中经常用于存储文件、路径、变量等一系列相关的数据。

# 定义宏

在CMake中，你可以使用`add_definitions()`或者更推荐的方法，使用`target_compile_definitions()`命令来定义宏。下面是两种定义宏的方式：

1. **使用`add_definitions()`命令**：

   这种方式在全局范围内定义宏，会应用于整个项目。但在CMake 3.12及以后的版本，推荐使用`target_compile_definitions()`来更精确地在目标级别定义宏。

   ```cmake
   add_definitions(-DMY_MACRO=1)
   ```

2. **使用`target_compile_definitions()`命令**：

   这种方式在特定目标上定义宏，更加灵活，能够在目标级别控制宏的作用范围。

   ```cmake
   add_executable(my_app main.cpp)
   target_compile_definitions(my_app PRIVATE MY_MACRO=1)
   ```

   在这个例子中，宏`MY_MACRO`会应用于`my_app`目标，但不会影响其他目标。

无论你选择哪种方式，定义宏都是为了在编译过程中向源代码添加预定义的符号，用于条件编译、功能开关等。

