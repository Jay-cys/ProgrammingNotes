objdump命令是Linux下的反汇编目标文件或者可执行文件的命令，它以一种可阅读的格式让你更多地了解二进制文件可能带有的附加信息。

# 1 命令选项
```basic
--archive-headers 
-a 
显示档案库的成员信息,类似ls -l将lib*.a的信息列出。 

-b bfdname 
--target=bfdname 
指定目标码格式。这不是必须的，objdump能自动识别许多格式，比如： 

objdump -b oasys -m vax -h fu.o 
显示fu.o的头部摘要信息，明确指出该文件是Vax系统下用Oasys编译器生成的目标文件。objdump -i将给出这里可以指定的目标码格式列表。 

-C 
--demangle 
将底层的符号名解码成用户级名字，除了去掉所开头的下划线之外，还使得C++函数名以可理解的方式显示出来。 

--debugging 
-g 
显示调试信息。企图解析保存在文件中的调试信息并以C语言的语法显示出来。仅仅支持某些类型的调试信息。有些其他的格式被readelf -w支持。 

-e 
--debugging-tags 
类似-g选项，但是生成的信息是和ctags工具相兼容的格式。 

--disassemble 
-d 
从objfile中反汇编那些特定指令机器码的section。 

-D 
--disassemble-all 
与 -d 类似，但反汇编所有section. 

--prefix-addresses 
反汇编的时候，显示每一行的完整地址。这是一种比较老的反汇编格式。 

-EB 
-EL 
--endian={big|little} 
指定目标文件的小端。这个项将影响反汇编出来的指令。在反汇编的文件没描述小端信息的时候用。例如S-records. 

-f 
--file-headers 
显示objfile中每个文件的整体头部摘要信息。 

-h 
--section-headers 
--headers 
显示目标文件各个section的头部摘要信息。 

-H 
--help 
简短的帮助信息。 

-i 
--info 
显示对于 -b 或者 -m 选项可用的架构和目标格式列表。 

-j name
--section=name 
仅仅显示指定名称为name的section的信息 

-l
--line-numbers 
用文件名和行号标注相应的目标代码，仅仅和-d、-D或者-r一起使用使用-ld和使用-d的区别不是很大，在源码级调试的时候有用，要求编译时使用了-g之类的调试编译选项。 

-m machine 
--architecture=machine 
指定反汇编目标文件时使用的架构，当待反汇编文件本身没描述架构信息的时候(比如S-records)，这个选项很有用。可以用-i选项列出这里能够指定的架构. 

--reloc 
-r 
显示文件的重定位入口。如果和-d或者-D一起使用，重定位部分以反汇编后的格式显示出来。 

--dynamic-reloc 
-R 
显示文件的动态重定位入口，仅仅对于动态目标文件意义，比如某些共享库。 

-s 
--full-contents 
显示指定section的完整内容。默认所有的非空section都会被显示。 

-S 
--source 
尽可能反汇编出源代码，尤其当编译的时候指定了-g这种调试参数时，效果比较明显。隐含了-d参数。 

--show-raw-insn 
反汇编的时候，显示每条汇编指令对应的机器码，如不指定--prefix-addresses，这将是缺省选项。 

--no-show-raw-insn 
反汇编时，不显示汇编指令的机器码，如不指定--prefix-addresses，这将是缺省选项。 

--start-address=address 
从指定地址开始显示数据，该选项影响-d、-r和-s选项的输出。 

--stop-address=address 
显示数据直到指定地址为止，该项影响-d、-r和-s选项的输出。 

-t 
--syms 
显示文件的符号表入口。类似于nm -s提供的信息 

-T 
--dynamic-syms 
显示文件的动态符号表入口，仅仅对动态目标文件意义，比如某些共享库。它显示的信息类似于 nm -D|--dynamic 显示的信息。 

-V 
--version 
版本信息 

--all-headers 
-x 
显示所可用的头信息，包括符号表、重定位入口。-x 等价于-a -f -h -r -t 同时指定。 

-z 
--disassemble-zeroes 
一般反汇编输出将省略大块的零，该选项使得这些零块也被反汇编。 

@file 可以将选项集中到一个文件中，然后使用这个@file选项载入。
```

# 2 符号表常见字段

- .text：已编译程序的机器代码。
- .rodata：只读数据，比如printf语句中的格式串和开关（switch）语句的跳转表。
- .data：已初始化的全局C变量。局部C变量在运行时被保存在栈中，既不出现在.data中，也不出现在.bss节中。
- .bss：未初始化的全局C变量。在目标文件中这个节不占据实际的空间，它仅仅是一个占位符。目标文件格式区分初始化和未初始化变量是为了空间效率在：在目标文件中，未初始化变量不需要占据任何实际的磁盘空间。
- .symtab：一个符号表（symbol table），它存放在程序中被定义和引用的函数和全局变量的信息。一些程序员错误地认为必须通过-g选项来编译一个程序，得到符号表信息。实际上，每个可重定位目标文件在.symtab中都有一张符号表。然而，和编译器中的符号表不同，.symtab符号表不包含局部变量的表目。
- .rel.text：当链接噐把这个目标文件和其他文件结合时，.text节中的许多位置都需要修改。一般而言，任何调用外部函数或者引用全局变量的指令都需要修改。另一方面调用本地函数的指令则不需要修改。注意，可执行目标文件中并不需要重定位信息，因此通常省略，除非使用者显式地指示链接器包含这些信息。
- .rel.data：被模块定义或引用的任何全局变量的信息。一般而言，任何已初始化全局变量的初始值是全局变量或者外部定义函数的地址都需要被修改。
- .debug：一个调试符号表，其有些表目是程序中定义的局部变量和类型定义，有些表目是程序中定义和引用的全局变量，有些是原始的C源文件。只有以-g选项调用编译驱动程序时，才会得到这张表。
- .line：原始C源程序中的行号和.text节中机器指令之间的映射。只有以-g选项调用编译驱动程序时，才会得到这张表。
- .strtab：一个字符串表，其内容包括.symtab和.debug节中的符号表，以及节头部中的节名字。字符串表就是以null结尾的字符串序列。




# 3 使用示例

## 3.1 源代码
依然使用死锁的例子
```c
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
int cnt = 0; //计数

void resetCnt()
{
    pthread_mutex_lock(&mutex);
    cnt = 0;
    pthread_mutex_unlock(&mutex);
}

void* subThread(void*)
{
    //子线程
    while(1)
    {
        pthread_mutex_lock(&mutex);
        if (cnt > 2)
            resetCnt(); //这里会死锁，重复锁定mutex
        else
            cnt++;
        pthread_mutex_unlock(&mutex);

        printf("%d\n", cnt);
        sleep(1);
    }
}

int main()
{
    pthread_t tid;
    pthread_create(&tid, NULL, subThread, NULL);
    pthread_join(tid, NULL);

    return 0;
}
```

## 3.2 显示.text字段
```bash
barret@Barret-PC:~$ g++ -c -g a.cpp==============生成.o文件
barret@Barret-PC:~$ ls -l
-rw-r--r-- 1 barret barret   3160 Jun 19 21:03 a.o
-rwxr-xr-x 1 barret barret  22104 Jun 19 20:35 a.out
=============显示二进制文件中text字段的内容===============
barret@Barret-PC:~$ objdump --section=.text -s a.o

a.o:     file format elf64-x86-64

Contents of section .text:
 0000 f30f1efa 554889e5 488d3d00 000000e8  ....UH..H.=.....
 0010 00000000 c7050000 00000000 0000488d  ..............H.
 0020 3d000000 00e80000 0000905d c3f30f1e  =..........]....
 0030 fa554889 e54883ec 1048897d f8488d3d  .UH..H...H.}.H.=
 0040 00000000 e8000000 008b0500 00000083  ................
 0050 f8027e07 e8000000 00eb0f8b 05000000  ..~.............
 0060 0083c001 89050000 0000488d 3d000000  ..........H.=...
 0070 00e80000 00008b05 00000000 89c6488d  ..............H.
 0080 3d000000 00b80000 0000e800 000000bf  =...............
 0090 01000000 e8000000 00eba2f3 0f1efa55  ...............U
 00a0 4889e548 83ec1064 488b0425 28000000  H..H...dH..%(...
 00b0 488945f8 31c0488d 45f0b900 00000048  H.E.1.H.E......H
 00c0 8d150000 0000be00 00000048 89c7e800  ...........H....
 00d0 00000048 8b45f0be 00000000 4889c7e8  ...H.E......H...
 00e0 00000000 b8000000 00488b55 f8644833  .........H.U.dH3
 00f0 14252800 00007405 e8000000 00c9c3    .%(...t........
```

## 3.3 反编译源代码
**-S命令对于包含调试信息的目标文件，显示的效果比较好。如果编译时没有指定g++的-g选项，那么目标文件就不包含调试信息，那么显示效果就差多了**。
```bash
================反汇编.text字段，显示源代码===================== 
 barret@Barret-PC:~$ objdump -j .text -S a.o

a.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <_Z8resetCntv>:

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
int cnt = 0; //计数

void resetCnt()
{
   0:   f3 0f 1e fa             endbr64 
   4:   55                      push   %rbp
   5:   48 89 e5                mov    %rsp,%rbp
    pthread_mutex_lock(&mutex);
   8:   48 8d 3d 00 00 00 00    lea    0x0(%rip),%rdi        # f <_Z8resetCntv+0xf>
   f:   e8 00 00 00 00          callq  14 <_Z8resetCntv+0x14>
    cnt = 0;
  14:   c7 05 00 00 00 00 00    movl   $0x0,0x0(%rip)        # 1e <_Z8resetCntv+0x1e>
  1b:   00 00 00 
    pthread_mutex_unlock(&mutex);
  1e:   48 8d 3d 00 00 00 00    lea    0x0(%rip),%rdi        # 25 <_Z8resetCntv+0x25>
  25:   e8 00 00 00 00          callq  2a <_Z8resetCntv+0x2a>
}
  2a:   90                      nop
  2b:   5d                      pop    %rbp
  2c:   c3                      retq   

000000000000002d <_Z9subThreadPv>:

void* subThread(void*)
{
  2d:   f3 0f 1e fa             endbr64 
  31:   55                      push   %rbp
  32:   48 89 e5                mov    %rsp,%rbp
  35:   48 83 ec 10             sub    $0x10,%rsp
  39:   48 89 7d f8             mov    %rdi,-0x8(%rbp)
    //子线程
    while(1)
    {
        pthread_mutex_lock(&mutex);
  3d:   48 8d 3d 00 00 00 00    lea    0x0(%rip),%rdi        # 44 <_Z9subThreadPv+0x17>
  44:   e8 00 00 00 00          callq  49 <_Z9subThreadPv+0x1c>
        if (cnt > 2)
  49:   8b 05 00 00 00 00       mov    0x0(%rip),%eax        # 4f <_Z9subThreadPv+0x22>
  4f:   83 f8 02                cmp    $0x2,%eax
  52:   7e 07                   jle    5b <_Z9subThreadPv+0x2e>
            resetCnt(); //这里会死锁，重复锁定mutex
  54:   e8 00 00 00 00          callq  59 <_Z9subThreadPv+0x2c>
  59:   eb 0f                   jmp    6a <_Z9subThreadPv+0x3d>
        else
            cnt++;
  5b:   8b 05 00 00 00 00       mov    0x0(%rip),%eax        # 61 <_Z9subThreadPv+0x34>
  61:   83 c0 01                add    $0x1,%eax
  64:   89 05 00 00 00 00       mov    %eax,0x0(%rip)        # 6a <_Z9subThreadPv+0x3d>
        pthread_mutex_unlock(&mutex);
  6a:   48 8d 3d 00 00 00 00    lea    0x0(%rip),%rdi        # 71 <_Z9subThreadPv+0x44>
  71:   e8 00 00 00 00          callq  76 <_Z9subThreadPv+0x49>

        printf("%d\n", cnt);
  76:   8b 05 00 00 00 00       mov    0x0(%rip),%eax        # 7c <_Z9subThreadPv+0x4f>
  7c:   89 c6                   mov    %eax,%esi
  7e:   48 8d 3d 00 00 00 00    lea    0x0(%rip),%rdi        # 85 <_Z9subThreadPv+0x58>
  85:   b8 00 00 00 00          mov    $0x0,%eax
  8a:   e8 00 00 00 00          callq  8f <_Z9subThreadPv+0x62>
        sleep(1);
  8f:   bf 01 00 00 00          mov    $0x1,%edi
  94:   e8 00 00 00 00          callq  99 <_Z9subThreadPv+0x6c>
        pthread_mutex_lock(&mutex);
  99:   eb a2                   jmp    3d <_Z9subThreadPv+0x10>

000000000000009b <main>:
    }
}

int main()
{
  9b:   f3 0f 1e fa             endbr64 
  9f:   55                      push   %rbp
  a0:   48 89 e5                mov    %rsp,%rbp
  a3:   48 83 ec 10             sub    $0x10,%rsp
  a7:   64 48 8b 04 25 28 00    mov    %fs:0x28,%rax
  ae:   00 00 
  b0:   48 89 45 f8             mov    %rax,-0x8(%rbp)
  b4:   31 c0                   xor    %eax,%eax
    pthread_t tid;
    pthread_create(&tid, NULL, subThread, NULL);
  b6:   48 8d 45 f0             lea    -0x10(%rbp),%rax
  ba:   b9 00 00 00 00          mov    $0x0,%ecx
  bf:   48 8d 15 00 00 00 00    lea    0x0(%rip),%rdx        # c6 <main+0x2b>
  c6:   be 00 00 00 00          mov    $0x0,%esi
  cb:   48 89 c7                mov    %rax,%rdi
  ce:   e8 00 00 00 00          callq  d3 <main+0x38>
    pthread_join(tid, NULL);
  d3:   48 8b 45 f0             mov    -0x10(%rbp),%rax
  d7:   be 00 00 00 00          mov    $0x0,%esi
  dc:   48 89 c7                mov    %rax,%rdi
  df:   e8 00 00 00 00          callq  e4 <main+0x49>

    return 0;
  e4:   b8 00 00 00 00          mov    $0x0,%eax
  e9:   48 8b 55 f8             mov    -0x8(%rbp),%rdx
  ed:   64 48 33 14 25 28 00    xor    %fs:0x28,%rdx
  f4:   00 00 
  f6:   74 05                   je     fd <main+0x62>
  f8:   e8 00 00 00 00          callq  fd <main+0x62>
  fd:   c9                      leaveq 
  fe:   c3                      retq
```

## 3.4 显示符号表
```bash
barret@Barret-PC:~$ objdump -t a.o

a.o:     file format elf64-x86-64

SYMBOL TABLE:
0000000000000000 l    df *ABS*  0000000000000000 a.cpp
0000000000000000 l    d  .text  0000000000000000 .text
0000000000000000 l    d  .data  0000000000000000 .data
0000000000000000 l    d  .bss   0000000000000000 .bss
0000000000000000 l    d  .rodata        0000000000000000 .rodata
0000000000000000 l    d  .debug_info    0000000000000000 .debug_info
0000000000000000 l    d  .debug_abbrev  0000000000000000 .debug_abbrev
0000000000000000 l    d  .debug_aranges 0000000000000000 .debug_aranges
0000000000000000 l    d  .debug_line    0000000000000000 .debug_line
0000000000000000 l    d  .debug_str     0000000000000000 .debug_str
0000000000000000 l    d  .note.GNU-stack        0000000000000000 .note.GNU-stack
0000000000000000 l    d  .note.gnu.property     0000000000000000 .note.gnu.property
0000000000000000 l    d  .eh_frame      0000000000000000 .eh_frame
0000000000000000 l    d  .comment       0000000000000000 .comment
0000000000000000 g     O .bss   0000000000000028 mutex
0000000000000028 g     O .bss   0000000000000004 cnt
0000000000000000 g     F .text  000000000000002d _Z8resetCntv
0000000000000000         *UND*  0000000000000000 _GLOBAL_OFFSET_TABLE_
0000000000000000         *UND*  0000000000000000 pthread_mutex_lock
0000000000000000         *UND*  0000000000000000 pthread_mutex_unlock
000000000000002d g     F .text  000000000000006e _Z9subThreadPv
0000000000000000         *UND*  0000000000000000 printf
0000000000000000         *UND*  0000000000000000 sleep
000000000000009b g     F .text  0000000000000064 main
0000000000000000         *UND*  0000000000000000 pthread_create
0000000000000000         *UND*  0000000000000000 pthread_join
0000000000000000         *UND*  0000000000000000 __stack_chk_fail
=========加-C，更好的可读性
barret@Barret-PC:~$ objdump -t -C a.o

a.o:     file format elf64-x86-64

SYMBOL TABLE:
0000000000000000 l    df *ABS*  0000000000000000 a.cpp
0000000000000000 l    d  .text  0000000000000000 .text
0000000000000000 l    d  .data  0000000000000000 .data
0000000000000000 l    d  .bss   0000000000000000 .bss
0000000000000000 l    d  .rodata        0000000000000000 .rodata
0000000000000000 l    d  .debug_info    0000000000000000 .debug_info
0000000000000000 l    d  .debug_abbrev  0000000000000000 .debug_abbrev
0000000000000000 l    d  .debug_aranges 0000000000000000 .debug_aranges
0000000000000000 l    d  .debug_line    0000000000000000 .debug_line
0000000000000000 l    d  .debug_str     0000000000000000 .debug_str
0000000000000000 l    d  .note.GNU-stack        0000000000000000 .note.GNU-stack
0000000000000000 l    d  .note.gnu.property     0000000000000000 .note.gnu.property
0000000000000000 l    d  .eh_frame      0000000000000000 .eh_frame
0000000000000000 l    d  .comment       0000000000000000 .comment
0000000000000000 g     O .bss   0000000000000028 mutex
0000000000000028 g     O .bss   0000000000000004 cnt
0000000000000000 g     F .text  000000000000002d resetCnt()
0000000000000000         *UND*  0000000000000000 _GLOBAL_OFFSET_TABLE_
0000000000000000         *UND*  0000000000000000 pthread_mutex_lock
0000000000000000         *UND*  0000000000000000 pthread_mutex_unlock
000000000000002d g     F .text  000000000000006e subThread(void*)
0000000000000000         *UND*  0000000000000000 printf
0000000000000000         *UND*  0000000000000000 sleep
000000000000009b g     F .text  0000000000000064 main
0000000000000000         *UND*  0000000000000000 pthread_create
0000000000000000         *UND*  0000000000000000 pthread_join
0000000000000000         *UND*  0000000000000000 __stack_chk_fail
```

## 3.5 汇编代码，并与源文件对应
```bash
barret@Barret-PC:~$ objdump -d -l a.o

a.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <_Z8resetCntv>:
_Z8resetCntv():
/home/barret/a.cpp:9
   0:   f3 0f 1e fa             endbr64 
   4:   55                      push   %rbp
   5:   48 89 e5                mov    %rsp,%rbp
/home/barret/a.cpp:10
   8:   48 8d 3d 00 00 00 00    lea    0x0(%rip),%rdi        # f <_Z8resetCntv+0xf>
   f:   e8 00 00 00 00          callq  14 <_Z8resetCntv+0x14>
/home/barret/a.cpp:11
  14:   c7 05 00 00 00 00 00    movl   $0x0,0x0(%rip)        # 1e <_Z8resetCntv+0x1e>
  1b:   00 00 00 
/home/barret/a.cpp:12
  1e:   48 8d 3d 00 00 00 00    lea    0x0(%rip),%rdi        # 25 <_Z8resetCntv+0x25>
  25:   e8 00 00 00 00          callq  2a <_Z8resetCntv+0x2a>
/home/barret/a.cpp:13
  2a:   90                      nop
  2b:   5d                      pop    %rbp
  2c:   c3                      retq   

000000000000002d <_Z9subThreadPv>:
_Z9subThreadPv():
/home/barret/a.cpp:16
  2d:   f3 0f 1e fa             endbr64 
  31:   55                      push   %rbp
  32:   48 89 e5                mov    %rsp,%rbp
  35:   48 83 ec 10             sub    $0x10,%rsp
  39:   48 89 7d f8             mov    %rdi,-0x8(%rbp)
/home/barret/a.cpp:20
  3d:   48 8d 3d 00 00 00 00    lea    0x0(%rip),%rdi        # 44 <_Z9subThreadPv+0x17>
  44:   e8 00 00 00 00          callq  49 <_Z9subThreadPv+0x1c>
/home/barret/a.cpp:21
  49:   8b 05 00 00 00 00       mov    0x0(%rip),%eax        # 4f <_Z9subThreadPv+0x22>
  4f:   83 f8 02                cmp    $0x2,%eax
  52:   7e 07                   jle    5b <_Z9subThreadPv+0x2e>
/home/barret/a.cpp:22
  54:   e8 00 00 00 00          callq  59 <_Z9subThreadPv+0x2c>
  59:   eb 0f                   jmp    6a <_Z9subThreadPv+0x3d>
/home/barret/a.cpp:24
  5b:   8b 05 00 00 00 00       mov    0x0(%rip),%eax        # 61 <_Z9subThreadPv+0x34>
  61:   83 c0 01                add    $0x1,%eax
  64:   89 05 00 00 00 00       mov    %eax,0x0(%rip)        # 6a <_Z9subThreadPv+0x3d>
/home/barret/a.cpp:25
  6a:   48 8d 3d 00 00 00 00    lea    0x0(%rip),%rdi        # 71 <_Z9subThreadPv+0x44>
  71:   e8 00 00 00 00          callq  76 <_Z9subThreadPv+0x49>
/home/barret/a.cpp:27
  76:   8b 05 00 00 00 00       mov    0x0(%rip),%eax        # 7c <_Z9subThreadPv+0x4f>
  7c:   89 c6                   mov    %eax,%esi
  7e:   48 8d 3d 00 00 00 00    lea    0x0(%rip),%rdi        # 85 <_Z9subThreadPv+0x58>
  85:   b8 00 00 00 00          mov    $0x0,%eax
  8a:   e8 00 00 00 00          callq  8f <_Z9subThreadPv+0x62>
/home/barret/a.cpp:28
  8f:   bf 01 00 00 00          mov    $0x1,%edi
  94:   e8 00 00 00 00          callq  99 <_Z9subThreadPv+0x6c>
/home/barret/a.cpp:20
  99:   eb a2                   jmp    3d <_Z9subThreadPv+0x10>

000000000000009b <main>:
main():
/home/barret/a.cpp:33
  9b:   f3 0f 1e fa             endbr64 
  9f:   55                      push   %rbp
  a0:   48 89 e5                mov    %rsp,%rbp
  a3:   48 83 ec 10             sub    $0x10,%rsp
  a7:   64 48 8b 04 25 28 00    mov    %fs:0x28,%rax
  ae:   00 00 
  b0:   48 89 45 f8             mov    %rax,-0x8(%rbp)
  b4:   31 c0                   xor    %eax,%eax
/home/barret/a.cpp:35
  b6:   48 8d 45 f0             lea    -0x10(%rbp),%rax
  ba:   b9 00 00 00 00          mov    $0x0,%ecx
  bf:   48 8d 15 00 00 00 00    lea    0x0(%rip),%rdx        # c6 <main+0x2b>
  c6:   be 00 00 00 00          mov    $0x0,%esi
  cb:   48 89 c7                mov    %rax,%rdi
  ce:   e8 00 00 00 00          callq  d3 <main+0x38>
/home/barret/a.cpp:36
  d3:   48 8b 45 f0             mov    -0x10(%rbp),%rax
  d7:   be 00 00 00 00          mov    $0x0,%esi
  dc:   48 89 c7                mov    %rax,%rdi
  df:   e8 00 00 00 00          callq  e4 <main+0x49>
/home/barret/a.cpp:38
  e4:   b8 00 00 00 00          mov    $0x0,%eax
/home/barret/a.cpp:39
  e9:   48 8b 55 f8             mov    -0x8(%rbp),%rdx
  ed:   64 48 33 14 25 28 00    xor    %fs:0x28,%rdx
  f4:   00 00 
  f6:   74 05                   je     fd <main+0x62>
  f8:   e8 00 00 00 00          callq  fd <main+0x62>
  fd:   c9                      leaveq 
  fe:   c3                      retq
```


