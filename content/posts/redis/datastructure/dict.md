---
title: "Dict"
date: 2025-01-27T10:20:06+08:00
lastmod: 2025-01-27T10:20:06+08:00
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




reids中的字典可以说是redis中最重要的数据结构之一，redis作为高性能的kv存储工具，其键值对存储就是用了dict结构
```C
typedef struct redisDb {
    ...  
    dict *dict;                 /* The keyspace for this DB */
    dict *expires;              /* Timeout of keys with a timeout set */
    ...
```
上面代码中的redisDb就是redis存储数据的数据库结构，其中dict变量和expires变量均为dict结构，分别存储了redis中的键值对和键的过期时间，键值对被存储在dict数据结构中。
# 基本的数据结构
redis dict数据结构定义在dict.h中
```c
typedef struct dict {
    dictType *type;  // 定义了字典的操作方式：如何比较键的函数、如何释放键和值
    void *privdata;  // 字典操作的上下文信息，通过它可以用来访问外部数据
    dictht ht[2];    // 用于存储数据的哈希表，一个用来存储真正的数据，另一个用于逐步迁移数据，实现渐进式哈希
    long rehashidx;  // 如果为-1则当前没有在进行rehash，否则代表当前rehash执行的索引
    unsigned long iterators; // 当前字典正在运行的迭代器数量，如果当前字典有迭代器运行，redis会采取不一样的措施
} dict;
```
- type属性定义了该dict的各种重要属性有：哈希方程、键可以如何被拷贝的函数、值如何被拷贝的函数，键之间比较的函数，键、值的销毁函数
- privadata定义了传给type属性函数的额外信息
```c
typedef struct dictType {
    uint64_t (*hashFunction)(const void *key);
    void *(*keyDup)(void *privdata, const void *key);
    void *(*valDup)(void *privdata, const void *obj);
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);
    void (*keyDestructor)(void *privdata, void *key);
    void (*valDestructor)(void *privdata, void *obj);

} dictType;
```
 - dictht结构存储了哈希表这里发现会有两个dictht，存储数据不是只需要一个表就好了？难道redis将表一分为二？事实上，这里其中的一个存储了数据，另一个仅仅用于rehash的时候逐步迁移数据，实现渐进式哈希
 - rehashidx表示当前是否在进行rehash，如果为-1那么没有在进行，如果非-1，那么代表当前字典rehash正在进行的索引
 - iterator记录了当前正在运行的迭代器数量
```c
typedef struct dictht {
    dictEntry **table; // 指向哈希表的指针，存储数据的地方
    unsigned long size;// 哈希表大小
    unsigned long sizemask;  // 用于计算哈希槽位的指针，大小一般为size-1
    unsigned long used;    // 哈希表中已经使用的槽位数
} dictht;
```
table是一个二维数组，存储了真正的哈希表，这里的dictEntry是一个哈希表数据节点，存储了键和值
```c
typedef struct dictEntry {
    void *key;     // 哈希节点的键
    union {
        void *val; 
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;

} dictEntry;
```
这里的value可以是一个void\*指针，也可以是uint64、int64或者double。next指针指向了下一个dictEntry，dict中解决冲突的方式就是将相同的键的节点通过链表的方式连接起来。
# hash算法与链式地址法
哈希算法将一个键值对放入合适的位置，这里的哈希算法就是dictType的hashFunction，每次向字典中新添加键值对时，程序就会调用该函数，可以看到该函数的返回值是一个uint64，将计算得到的哈希和字典的sizemask进行与操作，就会得到新添加元素在字典中的下标索引。

哈希冲突是指两个或两个以上的键值对的哈希索引相同，在redis的dict中，使用链地址法解决冲突，他的思想就是将相同的哈希所以键值对放在一个链表中。为了提高解决冲突的速度，redis在解决冲突时将会直接将新添加的键值对放在链表的第一个位置，这样的时间复杂度将会是O(1)
# rehash
当字典中的键值对越来越多时，可以预见到冲突的键值对将会越来越对，这样会导致链地址法所用到的链表长度变得异常长，大大增加字典查询的时间复杂度，相反的如何字典很空，那么将会浪费大量空间。因此程序需要通过扩展或者收缩控制哈希表的大小，将他控制在一个合理的范围内。

字典通过rehash进行扩展或者收缩，rehash的过程就用到了dict数据结构中的ht\[1\]。步骤是：
1. 基于ht[0]的存储情况计算出一个合理的哈希表大小，将ht[1]空间扩展到这个合理的大小。
	 - 收缩：ht[1]大小将为ht[0].used
	 - 扩张：ht[1]大小将为ht[0].used*2
2. 遍历ht[0]的所有键值对，根据ht[1]的大小重新计算哈希值和索引，将键值对复制到ht[1]对应的位置
3. 释放ht[0]，将ht[1]设置为ht[0]，为ht[1]创建一个新的空哈希表
