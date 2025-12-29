---
draft: false
comments: true
date: 2025-12-29
tags: [c， libdl]
---

# live c program
程序运行时编译和执行c代码。公司有个基础库可以修改和打印进程的状态，这个库是基于GNU的bfd的，代码比较复杂。在新项目中，发现dlsym（libdl）可以实现类似功能，并且代码非常简洁。  
在HN上看到一个很强大的示例也是用的libdl https://nullprogram.com/blog/2014/12/23/, 在运行时改代码，实际也是修改了状态。

## dlsym
dlopen 是打开动态库的，dlsym用来查询动态库或者可执行文件的符号，第一个参数表示查询范围。如果设置成动态库则表示在动态库找。如果设置成RTLD_DEFAULT，则表示在可执行文件和依赖里找，包括以RTLD_GLOBAL方式dlopen的动态库。
```
NAME
       dlsym, dlvsym - obtain address of a symbol in a shared object or executable
LIBRARY
       Dynamic linking library (libdl, -ldl)
SYNOPSIS
       #include <dlfcn.h>
       void *dlsym(void *restrict handle, const char *restrict symbol);
```

## demo

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <dlfcn.h>
int a;
int foo(int b){
        a = b;
}
int main(){
        char *code = NULL;
        size_t n = 0;
        getline(&code, &n, stdin);
        code[n] = 0;
        char* lstart = code;
        char* rstart = strchr(code, '=');
        *rstart = 0;
        rstart ++;
        void* var = dlsym(RTLD_DEFAULT, lstart);
        if(var){
                *(int*)var = atoi(rstart);
        }else{
                printf("invalid variable");
        }
        printf("a = %d\n", a);
        free(code);
        return 0;
}
```
通过判断用户输入的'='还是'()'来区分是函数还是变量。 函数可声明成
```c
void (*fun)(int) = dlsym(RTLD_DEFAULT, lstart);
fun();
```

## 符号类型检查，函数和变量
dlsym放回到是void*，指向一个地址，不知道是函数还是变量。如果getline读入了赋值语句（=）而实际是函数，那可能导致程序退出。如果要检查地址类型，需要用`dlinfo`。

## 函数的返回类型区别
在调用函数时，最好是忽略返回值。因为如果申明函数要返回一个值，而函数实际是void类型的，则在执行时会导致未定义的错误。相反如果函数有返回值，但是声明成void函数，则不会有任何问题。

## 类似的还有 [LibTCC](https://gist.github.com/energygreek/26393a80f3c54227c57fc33454649b28)
