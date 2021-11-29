### inline在c++的两个作用
曾经我很疑惑为什么定义在头文件且类外的成员函数为什么要加inline，现在找到了答案。
参考：https://stackoverflow.com/questions/9734175/why-are-class-member-functions-inlined?noredirect=1&lq=1

>It tells the compiler that the function code can be expanded where the function is called, instead of effectively being called.
It tells the compiler that the function definition can be repeated.
Point 1. is "archaic" in the sense that the compiler can in fact do what it likes in order to optimize code. It will always "inline" machine code if it can and find convenient to do and it will never do that if it cannot.
Point 2. is the actual meaning of the term: if you define (specify the body) a function in the header, since a header can be included in more sources, you must tell the compiler to inform the linker about the definition duplicates, so that they can be merged.

在类外定义的inline指示并不是真要函数inline，而是告诉链接器忽略其他编译单元的这个函数定义而不会产生多重定义的错误。

做一个小实验：

```c++
//a.h
#pragma once

class Sample
{
public:
    void func();
};

inline void Sample::func()
{
    int a = 0;
    int b = a + 1;
}
```

```c++
//b.cc
#include "a.h"

void funcb()
{
    Sample s;
    int a = 0xaaaaaaaa;
    s.func();
    int b = 0xaaaaaaaa;
}
```
g++ -c b.cc 后objdum查看汇编文件：
```
0000000000000000 <_Z5funcbv>:
   0:   f3 0f 1e fa             endbr64 
   4:   55                      push   %rbp
   5:   48 89 e5                mov    %rsp,%rbp
   8:   48 83 ec 20             sub    $0x20,%rsp
   c:   64 48 8b 04 25 28 00    mov    %fs:0x28,%rax
  13:   00 00 
  15:   48 89 45 f8             mov    %rax,-0x8(%rbp)
  19:   31 c0                   xor    %eax,%eax
  1b:   c7 45 f0 aa aa aa aa    movl   $0xaaaaaaaa,-0x10(%rbp)
  22:   48 8d 45 ef             lea    -0x11(%rbp),%rax
  26:   48 89 c7                mov    %rax,%rdi
**29:   e8 00 00 00 00          call   2e <_Z5funcbv+0x2e>**
  2e:   c7 45 f4 aa aa aa aa    movl   $0xaaaaaaaa,-0xc(%rbp)
  35:   90                      nop
  36:   48 8b 45 f8             mov    -0x8(%rbp),%rax
  3a:   64 48 2b 04 25 28 00    sub    %fs:0x28,%rax
  41:   00 00 
  43:   74 05                   je     4a <_Z5funcbv+0x4a>
  45:   e8 00 00 00 00          call   4a <_Z5funcbv+0x4a>
  4a:   c9                      leave  
  4b:   c3                      ret    

Disassembly of section .text._ZN6Sample4funcEv:

**0000000000000000 <_ZN6Sample4funcEv>:**
   0:   f3 0f 1e fa             endbr64 
   4:   55                      push   %rbp
   5:   48 89 e5                mov    %rsp,%rbp
   8:   48 89 7d e8             mov    %rdi,-0x18(%rbp)
   c:   c7 45 f8 00 00 00 00    movl   $0x0,-0x8(%rbp)
  13:   8b 45 f8                mov    -0x8(%rbp),%eax
  16:   83 c0 01                add    $0x1,%eax
  19:   89 45 fc                mov    %eax,-0x4(%rbp)
  1c:   90                      nop
  1d:   5d                      pop    %rbp
  1e:   c3                      ret
```
可以发现其并没有内联。



