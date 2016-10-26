---
layout: post
title:  "Windows shellcode"
categories: 翻译　文章
tags:  shellcode
---

* content
{:toc}

> 这篇文章包含了shellcode的开发技术和一些具体问题的概述。理解了这些概念，你将可以写出属于自己的shellcode。此外，还可以通过修改现有exp的shellcode来执行一些自定义功能。

### 0x00 简介

假设我们有一个exp，可以利用IE浏览器或者Flash播放器打开计算器calc.exe，但是这并没有什么用，我们真正想要的是执行一些远程命令或者实现其他有用的功能。在这种情况下，可能会想到使用Shell Storm database或者通过Metasploit下的msfvenom生成，然而，我们必须先了解编写shellcode的基本原理，这样才可以有效的利用这些工具写出自己的exp。

先介绍一下shellcode，这是从wiki摘录来的定义：
In computer security, a shellcode is a small piece of code used as the payload in the exploitation of a software vulnerability. It is called "shellcode" because it typically starts a command shell from which the attacker can control the compromised machine, but any piece of code that performs a similar task can be called shellcode...Shellcode is commonly written in machine code.

翻译过来大概就是：在计算机安全中，shellcode是在软件漏洞利用中用作有效攻击载荷的一段很小的代码，被称为“sell code”是因为攻击者通常启动一个命令就可以控制入侵的机器，任何执行相似任务的那一小段机器代码都可以被称为shellcode。
既然shellcode是一小段可以作为exp有效攻击载荷的机器代码，那么，“机器代码”又是什么呢？

下面我们以C语言代码为例说明：

```c
#include <stdio.h>
int main()
{
    printf("Hello, World!\n");
    return 0;
}

```

将这段代码翻译成汇编代码，如下：

```C
_main PROC
    push ebp
    mov ebp, esp
    push OFFSET HelloWorld ; "Hello, World!\n"
    call _printf
    add esp, 4
    xor eax, eax
    pop ebp
    ret 0
_main ENDP

```

> 这里要注意的是主程序和调用printf函数。

![图１](http://ofn62lbqx.bkt.clouddn.com/1.png)


这段代码就是由机器代码编写的，可以在调试器中看到高亮显示了
因此，“55 8B EC 68 00 B0 33 01...”这串代码就是上面那段C语言代码的机器代码。

### 0x01 shellcode在exp中的使用

先来举一个例子，这是一个基于堆栈的缓冲区溢出漏洞的简单exp：

```c
void exploit(char *data)
{
    char buffer[20];      // 定义缓冲区大小；
    strcpy(buffer, data); // 利用strcpy函数来复制数据；
}

```

下面是漏洞利用的主要思路：

* 向应用程序发送一段长度大于20字节的字符串，其中包含shellcode；
* 由于静态分配的缓冲区边界被覆盖，堆栈被损坏，shellcode将会替代堆栈；
* 发送的字符串利用自定义的内存地址覆盖堆栈上的一段重要数据（eg.保存的EIP或函数指针）；
* 应用程序从堆栈跳转到shellcode，开始执行机器代码的指令。

如果能过成功利用该漏洞运行shellcode，那么我们应该利用该漏洞做点有用的事情，而不仅仅是让程序崩溃。shellcode能够打开一个shell，下载和执行文件，重启计算机，开启RDP或者执行其他的命令。

### 0x02 shellcode的特异性

shellcode和其他机器代码是不同的，我们在写shellcode的时候要考虑到其他的因素，比如：
* 我们不能用字符串的绝对地址或者绝对偏移；
* 我们不知道要调用的函数地址；
* 我们需要避开一些特定的字符，例如空字符。

接下来简短的讨论一下上述几个问题：

#### 0x021 字符串的直接偏移量

在C/C++代码中，我们可以定义一个全局变量，将字符串赋值“Hello，world”；在“Hello,world”这个例子中，还可以直接将该字符串作为一个函数的参数。编译器将该字符串放置在文件的一个特定部分中：

![图２](http://ofn62lbqx.bkt.clouddn.com/2.png)


由于我们需要的是浮动地址代码，希望有字符串作为shellcode的一部分，所以必须将字符串存储在堆栈上。

#### 0x022 函数地址

C/C++代码调用函数很容易，指定#include<>通过名称使用特定的头和调用函数。在后台，编译器和链接器需要找到函数地址，然后我们通过函数名称调用所需函数。

![图３](http://ofn62lbqx.bkt.clouddn.com/3.png)

 在shell code中不能这样做。我们并不知道，包含我们所需函数的DLL是否被加载到内存中，也不知道所需函数的地址。Dll由于ASLR（地址空间随机化）不会每次都在同一地址加载，同时，Dll会随着windows的更新而改变，因此，我们不能依赖特定的偏移量。
 
   我们必须把Dll加载到内存中，直接从shellcode中找到我们所需的函数。幸运的是，Windows API提供了两个有用的函数：LoadLibrary和GetProcAddress函数，可以通过它来找到我们所需函数的地址。

#### 0x023 避开空字节 

   空字节的值为0x00,在C/C++语言中，空字节通常表示一个字符串的结束符，因此，shellcode中这些字符的存在可能会干扰到目标程序的功能，导致shellcode 无法正确的复制到内存中。
   
这种情况一般不是强制性的，常见的有类似使用strcpy()函数导致的缓冲区溢出，这个函数是逐个字节的复制字符串，当它遇到空字节的时候，就会停止复制，所以，当shellcode里面包含空字节，strcpy()函数就会在遇到空字节时停止，那么shellcode就不完整，无法正常运行。

 ![图４](http://ofn62lbqx.bkt.clouddn.com/4.png)
<p><img src="/img/2016/windows/4.png"></p>

上面这个图片中的两个指令的功能是相同的，但是第一个包含一个空字符，第二个没有，空字节在已编译的代码中是很常见的，也不难避免。

但是，也有一些比较特殊的情况shellcode必须避免使用一些字符，比如\r或者\n，甚至有的时候只能使用字母数字字符。

### 0x03 Linux shellcode vs Windows shellcode

为Linux编写一个基础的shellcode更简单。这是因为在Linux上，可以使用系统调用（系统“函数”），例如我们用0x80中断（以“函数调用”的方式）写入，执行或发送。可以在这里找到系统调用的[列表](http://syscalls.kernelgrok.com)

例如，Linux上的“Hello，world”shellcode需要执行以下步骤：

1.指定系统调用号（如“write”）

2.指定syscall参数（如stdout，“Hello，world”，length）

3.中断0x80以执行系统调用

这将导致调用：write（stdout，“Hello，world”，length）

在Windows上这个过程要复杂一些，创建一个可靠的shellcode还有更多必要的步骤：

1. 获取kernel32.dll基地址

2. 找到GetProcAddress函数的地址

3. 使用GetProcAddress找到LoadLibrary函数的地址

4. 使用LoadLibrary加载DLL（如user32.dll）

5. 使用GetProcAddress查找函数的地址（如MessageBox）

6. 指定功能参数
    		
7. 调用该函数

### 0x04总结

这是一系列关于初学者如何编写Windows shellcode的文章的第一部分,为大家呈现什么是shellcode以及Windows shellcode和Linux shellcode之间差异和限制的介绍。

第二部分将简要介绍汇编语言，PE（便携式可执行文件）文件和PEB（过程环境块）的格式。稍后将会看到这将如何帮助我们编写自定义的shellcode。

