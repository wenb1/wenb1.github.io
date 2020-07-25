---
layout: post
title: MySQL InnoDB数据页结构(Page Structure)
categories: MySQL
description: MySQL InnoDB数据页结构
keywords: MySQL, Database, 数据库
---

`页`是`InnoDB`管理存储空间的基本单位，一般一个页的大小是`16KB`。`InnoDB`为了实现不同的目的而设计了许多页，比如存放表空间头部信息的页，存放`Insert Buffer`信息的页等等。存放我们表中记录的页是`索引页（index page）`，鉴于我们还没有了解过索引是个什么东西，而这些表中的记录就是我们日常口中所称的`数据`，所以目前还是叫这种存放记录的页为`数据页`吧。

# 1. 数据页结构的快速浏览

数据页代表的这块`16KB`大小的存储空间可以被划分为多个部分，不同部分有不同的功能，各个部分如图所示：

![mysql10](/images/posts/mysql/mysql_10.PNG)