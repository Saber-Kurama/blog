---
 title: Rust 2021 版入门
 date: 2022-01-16
 tag: rust
---
# rust

## Allocating Memory (分配内存)

* The various kinds of memory allocation, their performance characteristics, and their limitations (各种内存分配、它们的性能特征和局限性)
* How to specify in Rust which memory allocation to use for an object (如何在 Rust 中指定用于对象的内存分配)
* The difference between a reference and a Box (reference和 Box之间的区别)

### The Various Kinds of Allocation (各种分配)

要理解 Rust 语言，以及任何其他系统编程语言，如 C 语言，重要的是理解内存分配的各种概念，如静态分配、堆栈分配和堆分配。
本章完全致力于这些问题。特别是，我们将看到四种内存分配：
* In processor registers   在处理器寄存器中
* Static  静态分配
* In the stack  在栈中
* In the heap 在堆中

在 C 和 C++ 语言中，静态分配是全局变量和使用 static 关键字声明的变量的分配；堆栈分配用于所有非静态局部变量，也用于函数参数；而堆分配是通过调用 C 语言标准库的 malloc 函数或 C++ 语言的 new 运算符来使用的。

### Linear Addressing 线性寻址
在任何计算机硬件中，都有一种可读写的内存，也称为 RAM，它由一长串字节组成，可以通过它们的位置访问。内存的第一个字节的位置为零，而最后一个字节的位置等于已安装内存的大小减一。
简而言之，在我们这个时代有两种计算机：
* Those where a single process at a time can run, and where such a process directly uses the physical memory addresses. These are called real-memory systems . （那些一次可以运行一个进程，并且这样的进程直接使用物理内存地址的地方。这些被称为实内存系统。）
* Those with a multiprogramming operating system, which offers a virtual address space to each of the running processes. These are called virtual-memory systems （那些拥有多道程序操作系统的人，它为每个正在运行的进程提供了一个虚拟地址空间。这些被称为虚拟内存系统）

在第一类计算机中，现在仅用作控制器，可能没有操作系统（因此它们也被称为裸机系统），或者可能有一个操作系统驻留在内存的第一部分.在最后一种情况下，应用程序可以使用大于某个值的地址。



