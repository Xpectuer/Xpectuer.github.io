---
layout: post
title: "教你用汇编语言写一个Hello World"
tags: programming assembly
categories: programming
date: 2020-11-15
---
# 教你用汇编语言写一个Hello World

### 平台

OS: Ubuntu 18.04 STL

**Platform: i386**

Editor：**vim**

### Get Started

我们知道，高级语言的源码文件都有自己的格式，比如，C的后缀：.c ，java的后缀： .java ， C++的后缀：.cpp等等。

在汇编语言程序设计时，我们采用 .asm后缀的格式（代表assembly）来编辑源码。

```shell
vim hello_world.asm
```

### 编辑源码

在i386汇编中，不同的section有不同的作用，

它们在被打包成[elf文件](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format)时被**链接（linked）**

![elf](/assets/programming/image/279759ee3d6d55fb68df1ab16f224f4a20a4dd08.jpeg)

> .asm文件中， 表示注释采用 ```;``` 符号

我们来看一下要用到的代码块（section）：

```assembly
section .text:
```

.text区域存放**所有将会被执行的指令**

```assembly
section .data:
```

.data区域存放用户自定义的**变量**

#### 变量

接下来，我们在```.data```中定义需要输出的变量

```assembly
section .data:
		message: db "Hello World!" , 0xA
		message_length equ $-message
```

解释一下这段代码：我们定义了一个变量叫做 ```message```

```db``` 意思是 define byte，这相当于某些高级语言的类型声明，只不过这里更像是一个**函数调用（function call）**，因为**有参数紧跟其后** 。

我们 输入我们自定义的值：```"Hello World!"```

这相当于第一个参数。

输入第二个参数，这会是一个16进制数（hex），用来**代表回车**。

我们输入```0xA```，这在高级语言中相当于```\n``` 作为换行符（newline）



#### 程序结构

定义好变量后，该动手写主体逻辑了：

之前讲过，```.text```代码块规定了主体逻辑，

我们需要定义在**寄存器register**（register见计算机组织架构）的内容来完成这个程序

mov指令参数： register , value，就是告诉cpu：“把value值搬进对应的寄存器register中。”


![](/assets/programming/image/register1.jpg)



[寄存器图例来源](www.tutorialspoint.com/assembly_programming/assembly_registers.htm)

> 简单说明一下这里的系统调用，这些系统调用的hex值可以在manual上查到
>
> Linux 输入指令 ```locate unistd_32.h``` 找到这个文件
>
> 随后我们vim打开这个文件查看系统调用的编码
>
>  ``` #define __NR_write 4```
>
> 查到write的编码为4
>
> 其他调用的查询以此类推

关于这里调用的原理，可以看这篇：[x86 Assembly Guide](https://www.cs.virginia.edu/~evans/cs216/guides/x86.html)

我们完成如下的代码：

```assembly
section
.text:
    mov eax, 0x4        ;调用system call write()
    mov ebx, 1          ;定义标准输出stdout为返回值，1是stdout的file describer
    mov ecx, message    ;在缓冲区存放变量message
    mov edx, message_length     ;存放变量长度
    int 0x80            ;调用系统调用 write()
                        ;int 代表了interrupt系统中断，此时中断返回值为1


    mov eax, 0x1        ;调用system call exit()
    mov ebx, 0          ;返回一个0，这里可以是任意值
    int 0x80						;中断指令
```

### 定义入口

```assembly
global	_start
```

这里的入口相当于 C / C++ 语言的**main函数**，告诉CPU程序的入口。

我们把```_start```放到逻辑开始位置

```assembly
section .text:
_start									;定义程序入口
    mov eax, 0x4        ;调用system call write()
    mov ebx, 1          ;定义标准输出stdout为返回值，1是stdout的file describer
    mov ecx, message    ;在缓冲区存放变量message
    mov edx, message_length     ;存放变量长度
    int 0x80            ;调用系统调用 write()
                        ;int 代表了interrupt系统中断，此时中断返回值为1


    mov eax, 0x1        ;调用system call exit()
    mov ebx, 0          ;返回一个0，这里可以是任意值
    int 0x80						;中断指令
```

### 汇编、链接

vim输入```:wq``` 退出到shell命令行中，并保存所写的内容

命令行输入

```shell
nasm -f elf32 -o hello_world.o hello_world.asm # 汇编源码，-f 指定了目标文件的类型 -o 即output指定了输出的对象文件名
```

如果一切顺利，则没有任何输出

随后我们链接文件，生成可执行文件

```shell
ld -m elf_i386 -o hello_world hello_world.o
```

我们执行

```shell
./hello_world
```

如果输出

```
Hello World!
```

那就大功告成了！



## 致谢

John Hammond
