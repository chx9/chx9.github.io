---
title: "gdb"
date: 2023-08-11T21:48:16+08:00
lastmod: 2023-08-11T21:48:16+08:00
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
    image: "" #图片路径例如:posts/tech/123/#png
    zoom: # 图片大小，例如填写 50% 表示原图像的一半大小
    caption: "" #图片底部描述
    alt: ""
    relative: false
---
# start命令
在GDB中，start 命令用于启动程序并停在程序的入口点处，等待你进行调试。它相当于设置一个断点在程序的 main 函数上并运行程序。这使你可以在程序开始执行之前检查变量、设置断点以及做其他调试操作。

以下是 start 命令的用法，其中 [args] 是传递给程序的命令行参数（
```
start [args] 
```
# gdb启动的时候指定参数
一种方法是在 gdb 启动程序的同时直接指定参数。格式如下
```bash
gdb --args your_program arg1 arg2 arg3...
```
另一种方法是在gdb中设置
1. 首先启动 gdb 并加载你的程序。例如，如果你的程序名为 `myprogram`，则运行：

   ```
   gdb myprogram
   ```

2. 在 gdb 提示符下，使用 `set args` 命令设置参数。例如，如果你想将 `arg1` 和 `arg2` 作为参数传递给你的程序，你应该运行：

   ```
   (gdb) set args arg1 arg2
   ```

3. 现在，当你使用 `run` 命令开始执行程序时，它将使用你指定的参数。你可以使用 `show args` 命令查看当前设置的参数：

   ```
   (gdb) show args
   Argument list to give program being debugged when it is started is "arg1 arg2".
   ```

# **run (r)**

运行程序

   ```
   r(un)
   ```

# **break (b)**
设置断点

   ```gdb
   b(reak) main
   ```

# **next (n)**
执行下一行代码

   ```
   n(ext)
   ```

# **list (l)**
显示源代码

   ```
   l(ist)
   ```

# **up**
进入上一级函数的上下文

   ```
   up
   ```

# **down**
进入下一级函数的上下文

   ```
   down
   ```

# **display/undisplay**
每次步进时显示或取消显示变量的值

   ```
   display variable_name
   undisplay variable_name
   ```

# **backtrace (bt)**
显示函数调用堆栈

   ```
   bt(acktrace)
   ```

# **step**
单步进入函数

   ```
   s(tep)
   ```

# **continue**
继续执行直到下一个断点

    ```
    c(ontinue)
    ```

# **finish**
执行完当前函数并停在调用该函数的位置

    ```
    finish
    ```

# **watch**
监视变量的值是否发生变化

    ```
    watch variable_name
    ```

# **info breakpoints**
显示当前设置的断点信息

    ```
    info breakpoints
    ```

# **delete**
删除指定断点

    ```
    delete breakpoint_number
    ```

# **reverse-next (rn)**
反向执行到上一行

    ```
    reverse-next
    ```

# **set var**
设置变量的值

    ```
    set var x = new_value
    ```

# **print (p)**
打印变量的值

   ```
   arduinoCopy code
   print variable_name
   ```

# **info locals**
显示当前函数的局部变量

   ```
   info locals
   ```

# **info args**
显示当前函数的参数

   ```
   info args
   ```

# **set breakpoint condition**
设置断点的触发条件

   ```
   break main if condition
   ```

# **disable/enable breakpoints**
禁用/启用断点

   ```
   disable breakpoint_number
   enable breakpoint_number
   ```

# **ignore breakpoint**
设置忽略断点的次数

   ```
   ignore breakpoint_number count
   ```

# **tbreak (tb)**
在单次执行后自动移除断点

   ```
   tbreak function_name
   ```

# **info frame**
显示当前栈帧的信息

   ```
   info frame
   ```

# **info registers**
显示寄存器的内容

   ```
   info registers
   ```

# **set $register = value**
设置寄存器的值

    ```
    set $rax = 42
    ```

# **x**
以不同的格式显示内存内容

    ```
    x/4xw memory_address
    ```

# **info threads**
显示所有线程的信息

    ```
    info threads
    ```

# **thread**
切换到指定线程的上下文

    ```
    thread thread_number
    ```

# **set pagination off/on**
关闭/打开分页显示

    ```
    set pagination off
    ```

# **attach**
附加到正在运行的进程

    ```
    attach process_id
    ```

# **detach**
从正在调试的进程中分离

    ```
    detach
    ```

# **info functions**
显示可用函数的列表

    ```
    info functions
    ```

# 发生段错误如何调试

段错误（Segmentation fault），通常也被称为 Sigsegv，是一种特定的错误条件，表示一个程序试图访问的内存区域并未被分配给它，或者只读。这通常是由于编程错误引起的，比如在 C/C++ 中，非法指针解引用、数组越界、堆栈溢出等情况可能触发段错误。

解决段错误的关键步骤如下：

**1. 定位错误**

你可以使用各种工具来帮助定位引发段错误的代码行，如 GDB、Valgrind 等。

对于 GDB，你可以这样使用：

```bash
gdb your-program
run
```

当段错误发生时，使用 `backtrace` 或 `bt` 命令查看调用堆栈。

**2. 检查可能的原因**

- 非法指针解引用：你是否有指针没有初始化就使用，或者指针已经释放后又使用？
- 数组越界：你是否访问了数组的边界之外？
- 堆栈溢出：你是否在栈上申请了大量的内存？

**3. 修复问题**

找到问题后，修复相应的代码。例如，对于空指针解引用，你需要检查指针是否为空，然后再进行操作。如果是数组越界，确定你的索引或计数器在正确范围内。

**4. 测试**

修复后，重新编译并运行你的程序，确认问题是否已经解决。

以上步骤并不保证能解决所有的段错误，但是至少能帮你找到出错的地方，并且提供一些可能的解决方案。碰到棘手的问题时，不要忘记寻求同事或者网络社区的帮助。

# gdb如何调试多线程程序

要使用 GDB 调试这个多线程程序，需要以下步骤:

1. 先编译你的程序并包含调试信息。你可以用 `-g` 参数来做到这一点：

```
g++ -g -std=c++11 multithread.cpp -o multithread -pthread
```

1. 然后，启动 GDB 并加载你的程序：

```
gdb ./multithread
```

1. 在 `main` 函数或者 `worker` 函数开始处设置断点。比如我们在 `worker` 函数设置断点，执行如下命令：

```
(gdb) break worker
```

1. 开始运行你的程序：

```
(gdb) run
```

1. 当程序停在断点时，你可以查看所有的线程：

```
(gdb) info threads
```

1. 这会列出所有的线程及其状态。每个线程都有一个唯一的 ID，你可以用这个 ID 来切换不同的线程，例如，切换到线程 2：

```
(gdb) thread 2
```

1. 你可以使用 `next`, `step`, `continue` 等命令在选定的线程中进行单步调试，或查看堆栈信息：

```
bt
```