---
layout: post
title: "在Mac环境下跑汇编"
date: 2013-12-30 23:12:34 +0800
comments: true
categories: [Mac, assembly]
---
今天汇编作业做到第七章，就想在Mac下跑自己的asm程序，看到了一个很好的[教程](http://www.raywenderlich.com/37181/ios-assembly-tutorial)

虽然对自己没有用，但是对汇编又有了一次了解，一些嵌入式设备系统如ios之类的是主要基于arm体系结构的操作系统，而pc机上的大多是基于intel体系结构的操作系统。查到了几种编译汇编代码的方式：

1. 用nasm直接编译.asm文件：

     nasm -f macho hello.asm

    生成可执行文件：

     ld -o hello -e mystart hello.o

     运行

    ./hello

    检查退出

    echo $?

2. 用gcc在.c文件中嵌套asm代码：

    gcc -Wall -m32 -O3 -fasm-blocks convert.c -o convert

    运行

    ./convert

3. 本来想直接运行书里的代码，但是发现书里的代码都是用masm语法编程的，所谓masm就是windows平台下的汇编编译器（对应linux下是nasm），所以在自己的电脑上就坑爹的跑不起来了，但是查到还是可以用一些如DOSbox的工具编译的asm文件然后和c文件连起来的。下了DOSbox，还要配置实在有些麻烦就想着还是明天去实验室做大程的时候顺便写吧，懒得折腾了。  

4. 生成list文件

nasm -f elf 6-2.asm -l 6-2.lst 

顺带出来的.o里面有一堆机器码，比原来的程序长很多
.lst文件是.asm文件翻译成机器码的结果就是一对一的翻译

反汇编
ndisasm 6-2.o