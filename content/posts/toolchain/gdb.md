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
