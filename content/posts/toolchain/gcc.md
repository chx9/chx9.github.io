---
title: "gcc"
date: 2023-08-15T21:04:34+08:00
lastmod: 2023-08-15T21:04:34+08:00
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

# gcc 优化的几个等级

1. gcc 中指定优化级别的参数有：-O0、-O1、-O2、-O3、-Og、-Os、-Ofast。

2. 在编译时，如果没有指定上面的任何优化参数，则默认为 -O0，即没有优化。

3. 参数 -O1、-O2、-O3 中，随着数字变大，代码的优化程度也越高，不过这在某种意义上来说，也是以牺牲程序的可调试性为代价的。

4. 参数 -Og 是在 -O1 的基础上，去掉了那些影响调试的优化，所以如果最终是为了调试程序，可以使用这个参数。不过光有这个参数也是不行的，这个参数只是告诉编译器，编译后的代码不要影响调试，但调试信息的生成还是靠 -g 参数的。

5. 参数 -Os 是在 -O2 的基础上，去掉了那些会导致最终可执行程序增大的优化，如果想要更小的可执行程序，可选择这个参数。

6. 参数 -Ofast 是在 -O3 的基础上，添加了一些非常规优化，这些优化是通过打破一些国际标准（比如一些数学函数的实现标准）来实现的，所以一般不推荐使用该参数

# gcc 的参数

`gcc` 是 GNU Compiler Collection 的缩写，它是一个用于编译 C/C++以及其他语言的强大编译器。以下是一些常用的 `gcc` 命令行参数：

1. `-o outputfile`: 指定输出文件的名称。例如，`gcc -o myprog myprog.c` 将会编译 `myprog.c` 并生成可执行文件 `myprog`。
2. `-c`: 只编译但不链接，通常生成 `.o` 对象文件。例如，`gcc -c myprog.c` 会生成 `myprog.o`。
3. `-g`: 在编译的时候包含调试信息，这样在使用 gdb 或其他调试器时可以显示更多有用的信息。
4. `-Wall`: 开启所有警告信息。这个参数可以帮助你发现代码中的潜在问题。
5. `-I dir`: 添加头文件搜索路径。如果你的头文件不在标准的系统路径下，你可以用 `-I` 参数告诉 `gcc` 到哪里找它们。
6. `-L dir`: 添加库文件搜索路径。如果你的库文件不在标准的系统路径下，你可以用 `-L` 参数告诉 `gcc` 到哪里找它们。
7. `-l libname`: 链接到名为 `libname` 的库。例如，`-lm` 将链接到数学库 `libm`。
8. `-std=`: 用于指定 C 或 C++的版本，例如 `-std=c99` 或 `-std=c++11`。
9. `-D name`: 定义预处理宏。例如，`gcc -D DEBUG ...` 相当于在源码前面添加了 `#define DEBUG`。

以上只是 `gcc` 最常用的部分参数，实际上 `gcc` 的参数非常丰富，可以根据具体需求选择合适的参数。
