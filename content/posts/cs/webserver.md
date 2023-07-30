---
title: "Webserver"
date: 2023-07-29T11:06:43+08:00
lastmod: 2023-07-29T11:06:43+08:00
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





# 1、采用IO复用技术Epoll和线程池实现多线程的Reactor高并发模型

## 什么是IO复用

IO多路复用是现代操作系统中常用的一种网络编程技术。它允许一个**进程同时监听多个文件描述符**，从而**可以同时处理多个网络连接，而无需为每个连接使用一个单独的线程。**

常用的IO多路复用的三种方式是select、poll和epoll。它们的实现方式不同，但都可以实现IO多路复用的功能。它们的区别如下：

### select

select是最古老、最常见的IO多路复用方式之一。它使用一个文件描述符集合来监听多个文件描述符，当其中任何一个文件描述符变为可读或可写时，select函数就会返回。

select的一个缺点是它使用的是线性扫描算法，因此它的性能随着文件描述符数量的增加而逐渐降低。

### poll

poll是select的一种改进，它使用一个pollfd结构体数组来监听多个文件描述符。与select不同的是，pollfd结构体数组是动态的，因此它可以监听更多的文件描述符。

poll的一个缺点是它仍然使用线性扫描算法，因此它的性能也会受到文件描述符数量的限制。

### epoll

epoll是Linux特有的一种IO多路复用方式。它使用一个epollfd结构体来管理多个文件描述符，并使用事件通知机制来通知应用程序哪些文件描述符已准备好进行读写操作。

epoll的优点是它使用了基于事件驱动的方式来通知应用程序，因此它的性能不会随着文件描述符数量的增加而下降。

Poll机制相较于Select机制中，解决了文件描述符数量上限为1024的缺陷。但另外两点缺陷依然存在：

每次调用poll，都需要把fd集合从用户态拷贝到内核态，fd越多开销则越大；
每次调用poll，都需要在内核遍历传递进来的所有fd，这个开销在fd很多时也很大

epoll是Linux下的一种I/O事件通知机制，可以用于高效地处理大量的socket连接。它提供了三个主要的函数：

1. `epoll_create()`：创建一个epoll实例，返回一个文件描述符。该函数的参数size指定epoll实例中最大的文件描述符数目，一般设置为需要监听的socket数。
2. `epoll_ctl()`：向epoll实例中添加或删除一个文件描述符，或者修改一个文件描述符的监听事件。该函数的参数epfd是epoll实例的文件描述符，op指定操作类型，fd是需要添加/删除/修改的文件描述符，event指定要监听的事件类型。
3. `epoll_wait()`：等待文件描述符上的事件发生，当有事件发生时，该函数将返回一个结构体数组，每个结构体描述一个事件。该函数的参数epfd是epoll实例的文件描述符，events是一个存放事件的数组，maxevents指定最多返回的事件数目，timeout指定等待的超时时间。如果timeout为-1，表示一直等待，直到有事件发生；如果timeout为0，表示立即返回，不等待。

单reactor多线程高并发模型有什么缺陷，在什么场景下会触发，解决办法是什么？

单reactor多线程高并发模型的缺陷在于：

1. 竞争条件：多个线程同时访问共享资源，容易出现竞争条件，导致数据不一致或者程序崩溃。
2. 上下文切换：线程切换需要消耗时间，当线程数过多时，会导致过多的上下文切换，降低程序的性能。
3. 瓶颈：单reactor模型只有一个事件循环，无法充分利用多核CPU的优势，无法处理更多的并发连接。

当并发连接数量增加时，单reactor多线程模型容易出现线程切换开销大、竞争条件等问题，导致性能下降，甚至出现程序崩溃等情况。

解决办法可以是使用多reactor模型，将事件分配到多个事件循环中处理，降低单个事件循环的负载，提高并发处理能力。同时，可以使用线程池等技术，将并发连接分配到多个工作线程中处理，避免竞争条件和上下文切换问题。此外，可以使用异步IO等技术，避免阻塞IO操作带来的性能问题。

[https://www.cnblogs.com/-wenli/p/13343397.html](https://www.cnblogs.com/-wenli/p/13343397.html)

## epoll 触发模式

epoll支持三种触发模式：

1. EPOLL_ET（边沿触发）：只有当文件描述符状态发生变化时，epoll才会触发相应的事件。该模式适用于需要及时响应文件描述符状态变化的场景，但相对来说需要更加精细的管理。
2. EPOLL_LT（水平触发）：只要文件描述符处于就绪状态，epoll就会持续不断地触发事件。该模式适用于需要频繁读写文件描述符的场景，但相对来说会产生更多的CPU开销。

# epoll 底层是如何实现的

一颗红黑树，一张准备就绪句柄链表，少量的内核 cache，就帮我们解决了大并发下的 socket 处理问题。

执行 epoll_create()时，创建了红黑树和就绪链表；

执行 epoll_ctl()时，如果增加 socket 句柄，则检查在红黑树中是否存在，存在立即返回，不存在则添加到树干上，然后向内核注册回调函数，用于当中断事件来临时向准备就绪链表中插入数据；

执行 epoll_wait()时立刻返回准备就绪链表里的数据即可。
![image-1680750833355](/upload/2023/04/image-1680750833355.png)
红黑树维护事件表，事件驱动机制下的回调函数加入就绪链表，拷贝只拷贝就绪链表

当某一进程调用 epoll_create 方法时，Linux 内核会创建一个 eventpoll 结构体，这个结构体中有两个成员与 epoll 的使用方式密切相关，如下所示：
```c++
truct eventpoll {
　　...
　　/*红黑树的根节点，这棵树中存储着所有添加到epoll中的事件，
　　也就是这个epoll监控的事件*/
　　struct rb_root rbr;
　　/*双向链表rdllist保存着将要通过epoll_wait返回给用户的、满足条件的事件*/
　　struct list_head rdllist;
　　...
};
```
我们在调用 epoll_create 时，内核除了帮我们在 epoll 文件系统里建了个 file 结点，在内核 cache 里建了个红黑树用于存储以后 epoll_ctl 传来的 socket 外，还会再建立一个 rdllist 双向链表，用于存储准备就绪的事件，当 epoll_wait 调用时，仅仅观察这个 rdllist 双向链表里有没有数据即可。有数据就返回，没有数据就 sleep，等到 timeout 时间到后即使链表没数据也返回。所以，epoll_wait 非常高效。

所有添加到 epoll 中的事件都会与设备(如网卡)驱动程序建立回调关系，也就是说相应事件的发生时会调用这里的回调方法。这个回调方法在内核中叫做 ep_poll_callback，它会把这样的事件放到上面的 rdllist 双向链表中。

在 epoll 中对于每一个事件都会建立一个 epitem 结构体，如下所示：

```c++
struct epitem {
　　...
　　//红黑树节点
　　struct rb_node rbn;
　　//双向链表节点
　　struct list_head rdllink;
　　//事件句柄等信息
　　struct epoll_filefd ffd;
　　//指向其所属的eventepoll对象
　　struct eventpoll *ep;
　　//期待的事件类型
　　struct epoll_event event;
　　...
}; // 这里包含每一个事件对应着的信息。
```
# 2、利用状态机和正则解析HTTP报文请求，实现静态资源的处理

状态机一共有四个状态：REQUEST_LINE， HEADERS， BODY， FINISH，分别对应了http请求的三个内容，FINISH是结束，程序通过while循环，将状态从REQUEST_LINE→Headers→Body→Finish转换。

## http请求和响应的格式

请求格式：

```cpp
<Method> <Request-URI> <HTTP-Version>
<Headers>
<Body>
```

响应格式

```coq

<HTTP-Version> <Status-Code> <Reason-Phrase>
<Headers>
<Body>
```

# 3、利用标准库容器封装char，实现自动增长的缓冲区

**缓冲区功能：**如果是固定的缓冲区，在存储小数据时会照成极大资源的浪费。自动增长的缓存区会根据每个客户端的请求响应数据大小，自适应的调整缓冲区大小。

**缓冲区场景：**

1. 从内核缓冲区中分散读取数据至缓冲区（in）
2. HTTP数据解析的时候，将缓冲区的数据读出（out）
3. HTTP生成响应数据的时候，会写入缓冲区（in）
4. 将缓冲区中的数据写入内核缓冲区（out）

**如何扩容：**

从内核缓冲区读数据采用的是**分散读（readv）**，会定义一个长度为2的结构体iovec数组（0存放缓冲区可写部分，1存放一个临时char数组）。在读的时候会首先将WritableBytes空间占满，当还有数据需要写入的时候就需要扩容。扩容分为两种情况，因为在HTTP类解析缓冲区数据的时候，readPos_会不断增加，那么PrependableBytes就会不断增加，这块区域是空闲的，如果说WritableBytes和PrependableBytes的空间大小满组的话，那就只需要移动数据。那如果不满足的话，就要resize缓冲区的大小。

**分散写：**

写的内容也是被放在一个2个大小iovec数组（0存放响应头，1存放html文件）

![image-1682217477732](/upload/2023/04/image-1682217477732.png)

- 代码详解
    
    ```cpp
    class Buffer {
    public:
      Buffer(int initBuffSize = 1024);
      ~Buffer() = default;
    
      // 有多少字节的数据可以写
      size_t WritableBytes() const;
      // 有多少字节的数据可以读
      size_t ReadableBytes() const;
      // buffer头部有多少字节的数据空出来
      size_t PrependableBytes() const;
    
      // 返回第一个可读字节的char地址
      const char *Peek() const;
      // 确保WritableBytes小于len，如果不是，就调用MakeSpace_动态扩容
      void EnsureWriteable(size_t len);
      // 写入结束，更新writePos_
      void HasWritten(size_t len);
    
      // 写出，根据长度调整readPos_
      void Retrieve(size_t len);
      // 写出，根据指针计算写出的长度，并调用Retrieve
      void RetrieveUntil(const char *end);
    
      // 重置缓冲区，在写入内核区缓存后来调用
      void RetrieveAll();
      std::string RetrieveAllToStr();
    
      // 定位buffer内数据段末尾，在了http解析函数中使用
      const char *BeginWriteConst() const;
      // 定位buffer内数据段首部
      char *BeginWrite();
    
      // 动态扩容
      void Append(const std::string &str);
      // 核心是这个，其他重载函数内部调用了这个
      void Append(const char *str, size_t len);
      void Append(const void *data, size_t len);
      void Append(const Buffer &buff);
    
      // 采用分散读，分散读的好处是减少系统调用次数，提高IO效率
      ssize_t ReadFd(int fd, int *Errno);
      // 这个函数没有用，WriteFd写在了HttpConn::write中
      ssize_t WriteFd(int fd, int *Errno);
    
    private:
      // 返回buffer头部指针
      char *BeginPtr_();
      const char *BeginPtr_() const;
      // 若PrependableBytes和WritableBytes大于len，整体数据往前移，若小于，那就要resize
      void MakeSpace_(size_t len);
    
      // 用动态数组封装的char缓冲区
      std::vector<char> buffer_;
      // 如果没有实现log的话，就不需要原子，因为每个线程独自占有各自的buffer
      std::atomic<std::size_t> readPos_;
      std::atomic<std::size_t> writePos_;
    };
    ```
    
    Buffer类中的成员变量包括一个动态数组封装的char缓冲区buffer_，以及两个std::atomic<std::size_t>类型的变量readPos_和writePos_，分别表示当前缓冲区的读取位置和写入位置。
    
    类中提供的主要接口包括：
    
    - WritableBytes()、ReadableBytes()、PrependableBytes()：分别用于获取缓冲区中可以写入的字节数、可以读取的字节数和缓冲区头部留出来的字节数。
    - Peek()：返回缓冲区中第一个可读字节的char地址。
    - EnsureWriteable()：确保缓冲区中可以写入的字节数大于等于len，如果不够则进行动态扩容。
    - HasWritten()：写入结束后更新writePos_。
    - Retrieve()、RetrieveUntil()：用于将已读取的数据从缓冲区中移除。
    - RetrieveAll()、RetrieveAllToStr()：用于将缓冲区中所有数据移除并返回。
    - BeginWriteConst()、BeginWrite()：返回缓冲区中可写数据的起始地址。
    - Append()：用于向缓冲区中增加额外的数据。
    - ReadFd()：用于采用分散读方式从文件描述符fd中读取数据并写入缓冲区。
    
    除此之外，类中还有一些私有函数，用于实现动态扩容、返回buffer头部指针等功能。
    

非阻塞模式下read返回值 < 0表示没有数据，= 0表示连接断开，> 0表示接收到数据。

# 4、基于小根堆实现定时器，关闭超时的非活动链接

**功能：**为每一个客户端设置一个定时器，用户关闭超时的非活动链接。

**实现：**定义了一个定时器结构体，包括id（服务端为每个客户端分配的文件描述符），过期时间，回调函数以及重载了一个比较操作符。实现了一个HeapTimer类，用于存储管理定时器。此外还设计了Tick和GetNextTick函数。

Trik函数用于更新计时器堆的状态，通过while循环来不断pop出超时的定时器节点。

GetNextTrick用于返回堆顶元素的到期时间戳与当前时间戳的差，如果堆为空，则返回-1。同时，结合这个返回到期时间戳的函数，可以减少epoll_wait系统调用次数，来提高效率。epoll_wait有一个参数叫timeout，设置为0，会直接返回，如果是-1就一直阻塞，如果大于0，表示要最多等到timeout时间才返回。那么我们假设timeout这段时间内没有新的事件发生，就没有必要一直轮询。

# 5、利用RAII机制实现数据库连接池，减少数据库建立和关闭的消耗


# epoll的零拷贝技术
在使用epoll进行网络编程时，零拷贝技术可以提高数据传输的效率，减少CPU的使用率和数据复制的开销。

具体来说，epoll的零拷贝技术可以通过以下两种方式实现：

1. sendfile系统调用

sendfile系统调用可以将文件数据直接从内核缓冲区发送到网络中，而无需将数据从内核缓冲区复制到用户空间缓冲区。在使用epoll进行网络编程时，可以通过sendfile系统调用将文件数据直接发送到网络中，从而避免了数据复制的开销。这种方式称为epoll的“零拷贝”技术。

2. mmap系统调用

mmap系统调用可以将文件映射到进程的地址空间中，从而使得应用程序可以直接访问文件数据，而无需进行读写操作。在使用epoll进行网络编程时，可以通过mmap系统调用将文件映射到进程的地址空间中，然后使用send系统调用将数据直接从内核缓冲区发送到网络中，从而避免了数据复制的开销。

总的来说，epoll的零拷贝技术可以将数据传输的效率提高到极致，减少CPU的使用率和数据复制的开销。这在高并发的网络编程中非常有用，可以显著提高系统的性能和稳定性。
# 为什么要用线程池
使用线程池的主要原因有:

- 避免频繁创建和销毁线程。在使用超时或存在较长的Waiting时间的应用中,如果每次都需要创建一个新的线程将造成较大的开销。线程池可以重用过期的空闲线程,从而避免线程创建的开销。

- 提高线程的可用性。在线程池中,可以有效地管理所有重复使用的线程,从而可以更好地利用这些可用的工作线程。

- 控制并发数量。通过调整线程池中包含的线程数量,可以控制同一时间内允许运行的最大线程数目。这对于避免过载控制非常有用。

- 提高资源利用率。 multiplex 可以重用同一个线程来处理多个任务,使资源得以更好地利用,比如共享内存的分配和设备等。

- 管理任务的进度。线程池允许通过设置最大线程数来限制任务的并行进行数,以使任务进展得以控制和管理。

-  避免死锁和饥饿，线程池可以通过控制线程的调度和资源分配来避免这些问题的发生。

需要使用线程池的主要场景有:

CPU 密集型任务。运行于循环或占用较长时钟周期的任务最适合使用线程池。

I/O 密集型任务。涉及大量 I/O 等待的任务,比如数据库查询,网络请求等也非常合适。

服务端应用。提供服务的应用,需要处理大量客户端请求,使用线程池可以管理多个请求使用同一个线程。

GUI 应用。从事件驱动线程处理 GUI 事件的应用也应该使用线程池,以避免频繁创建和销毁 UI 线程。

总结来说,在以下情况下,使用线程池会更为合适:

任务较少,执行时间较长的任务。

存在较长时间的I/O等待的任务。

需要控制并发数的场景。如避免过载。

需要重用空闲的工作线程的情况。如处理密集的但较短期的请求等。

# 线程池的核心参数
创建一个线程池需要考虑的关键信息有:

核心线程数(corePoolSize):线程池中始终存在的基本大小的工作线程数目。这个值通常很小,默认值是 1。

最大线程数(maximumPoolSize):线程池能容纳的最大线程数。即使处理请求需要更多的工作线程,这个值也不会超过这个上限值。

允许线程阻塞的时间(keepAliveTime):当空闲的工作线程数超过核心大小时,允许适当的扩充。当没有新的任务到达指定的时间后,线程池会进行判断,如果当前空闲的工作线程数超过核心大小,且空闲时间超过keepAliveTime,那么多余的线程会被销毁。单位是毫秒。默认值是 60 秒。

阻塞队列(BlockingQueue):用来存储任务的阻塞队列。通常选择LinkedBlockingQueue或者ArrayBlockingQueue。他们同时还支持超时特性,可以指定阻塞队列不满时的超时时间。

续约策略(补充策略):用于在任务捶捶式增长时决定是否增加或减少工作线程数的策略。常用策略有:

FIFO FIFO 队列策略,按照任务添加到阻塞队列的顺序依次处理。公平性最强。
LIFO 后入先出队列策略。
PriorityQueue 优先级队列,根据任务优先级依次处理。
LinkedBlockingQueue 链式阻塞队列,默认使用 FIFO 策略。
ArrayBlockingQueue 数组阻塞队列,同样使用 FIFO 策略。
