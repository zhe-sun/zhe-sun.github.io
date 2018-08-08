---
layout: post
title: Linux Direct I/O
description: ""
comments: true
reading_time: true
modified: 2014-12-03
tags: [Linux Kernel]
image:
  feature: abstract-2.jpg

---

## Direct I/O In Linux ##

### Introduction ###

To the traditional operating system, common I/O operations will normally be cached in the kernel call `Buffer I/O`. In this paper, the file access mechanism we will describe doesn't pass through the cache of Linux kernel. Data transfer directly to disk from the address space of application. So it calls `Direct I/O` offered by Linux.

### Motivation of Direct I/O ###

### What is the Buffer I/O #####

Buffer I/O is also known as the standard I/O which as the default I/O operation offered by most file systems. In the cache mechanism of Linux OS will cache the I/O data in the `page cache`, that is to say, data will be copied to the buffer in the kernel space, and then, copied to the application space in the user space from kernel space.
**`Buffer I/O` have some advantages below:**

* `Buffer I/O` use the kernel space buffer, separate the application space and the physical devices to some extent.
* `Buffer I/O` can improve the R/W performance by reducing the time of R/W the disks.

When applications attempt to read the data on the disk, if the data have already stored in the `page cache`, the data can be immediately returned to the applications without R/W the physical disk. Of course, if the data don't exist in the `page cache` before the applications occur reading, that need to read the data from disk to the `page cache`. 

To the write operation, application will also write the data to the `page cache` first, beside, whether the data is written to the disk immediately depend on the write operation mechanism adopted by application: if user use the `synchronous write`, the data will be written to the disk immediately, the applications will wait until the data translating over; If it's the `deferred write` mechanism, application will not wait, and return when data is written into the `page cache` immediately. In the case of `deferred write`, OS will flush the data in `page cache` to the disk regularly. Different from `asynchronous write`, `deferred write` don't notify the application when the data translation is over. `Asynchronous write` will return to applications when the data translation is over. So, `deferred write` can case a risk of data loss, the `asynchronous write` can't.

### Disadvantage Of Buffer I/O ###

In the `Buffer I/O`, the DMA can translate data from disk to `page cache` directly, or write back the data from `page cache` to disk directly. But DMA can not translate from applications address space to disk, in this case, while in transit data need to make multiple copies between applications address space and the `page cache`. The overhead of the data copy operation between the CPU and memory is very large.

For some special applications, avoiding the kernel buffer of operation system and directly translating from application address space to disk will get better performance than using the kernel buffer. The `self-caching application` is one of the special applications.

### Self-caching Applications ###

To some application, it would have its own data cache mechanism. For example, it will cache the data in its application address space. Such applications do not need to use the kernel buffer and they are called `self-caching applications`. Database Management System is a representative of this type of applications. Self-caching applications use the logical expression of data rather than physical expression. When the memory of system is low, self-caching application will make the logical cache of data to be swapped out, rather than the the actual data in the disk. The self-caching application know the semantics of the data want to work with, so it can use a more efficient cache replacement algorithm. Self-caching application may be shared a block of memory between multiple hosts, thus self-caching applications need to offer a mechanism which can make the data cached in the application address space invalid to ensure consistency of the data cached in the application address space.

To the self-caching applications, `buffer I/O` is not a good chance obviously. The `Direct I/O` in Linux is good for self-caching applications, the data translate from applications address space to disk directly which makes self-caching applications can omit the complicated structure of the system and implement programs to R/W your own custom data management to reduce the impact of system-level management for application to access data.

## Direct I/O In Linux 2.6 ##

### Several File Access Modes in Linux 2.6 ###

All of the I/O operations access the file by R/W. Here is the list of Linux 2.6 supports file access methods.

#### The Standard Way To Access The File ####

In Linux, this way to access files is implemented with two system call: `read()` `write()`.


