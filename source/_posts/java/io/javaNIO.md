---
title: java中NIO
date: 2020-01-17 16:55:34
tags: [java]
---

Java NIO（New IO） 是从Java 1.4版本开始引入的一个新的IO API，可以替代标准的Java IO API。NIO与原来的IO有同样的作用和目的，但是使用的方式完全不同， NIO支持面向缓冲区的、基于通道的IO操作。 NIO将以更加高效的方式进行文件的读写操作。


1.java.nio.Buffer

IO是以流的方式处理数据，而NIO是以块的方式处理数据

2.java.nio.channels Interface Channel

Channel是一个对象，可以通过它读取和写入数据。可以把它看做IO中的流。但是它和流相比还有一些不同

Channel是双向的，既可以读又可以写，而流是单向的

Channel可以进行异步的读写

对Channel的读写必须通过buffer对象 ( 应用程序不能直接对 Channel 进行读写操作，而必须通过 Buffer 来进行，即 Channel 是通过 Buffer 来读写数据的。)
