---
title: c++ 信号和异常
date: 2021-09-16
tags: [c, cpp, signal, exception]
---

# c++的信号和异常

信号和异常都是c++程序会遇到的，以前常把信号也当异常来看待，当其实两者是截然不同的东西

## 信号

信号不仅是对c++程序，而是对所有linux程序，是内核`在进程由内核态转用户态`时，检查信号槽并调用信号处理函数（用户定义或者默认）。  
有些信号可以由程序处理（常见退出信号SIGINT），而有些无法处理（常见退出信号SIGABT, 段错误信号SIGSEGV），而大多信号默认是退出进程。


## 异常

c++的异常是c所没有的，异常一般在出现`逻辑错误`和`运行时错误`时要抛出异常，如果没有异常处理函数，则coredump。


## 触发方式

触发信号用raise，而异常用trow 
```c
int raise(int sig);
```
