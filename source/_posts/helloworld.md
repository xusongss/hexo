---
title: HelloWorld
date: 2016-06-11 00:12:06
comments: true
tags:
- linux

---
Hello World
```C
#include <stdio.h>
#include <stdlib.h>

static  __attribute__((constructor)) void before()
{

    printf("Hello");
}

static  __attribute__((destructor)) void after()
{
    printf(" World!\n");
}

int main(int args,char ** argv)
{

    return 0;
}
```

## \_\_attribute\_\_
[https://gcc.gnu.org/onlinedocs/gcc-4.7.1/gcc/Function-Attributes.html](https://gcc.gnu.org/onlinedocs/gcc-4.7.1/gcc/Function-Attributes.html)
