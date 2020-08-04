---
date: 2020-08-05 01:03:13
layout: post
title: "Assembly Language Series: 0x01"
subtitle:
description: In this article, you will get a brief introduction to binary exploitation by learning disassembly using gdb. 
image: /assets/img/uploads/assembly-01/cover.webp
optimized_image: /assets/img/uploads/assembly-01/cover-optimized.webp
category: exploitation
tags: binary-exploitation
author: yashsaxena
paginate: false
keywords: assembly, assembly language tutorial, disassembly hello world, infosec articles, assembly language series, disassembly using gdb, assembly language introduction, binary exploitaiton
---

Hello Everyone, I am starting a new series related to Assembly Language. In this article we are going to analyze a very small program using <b>gdb.</b> I have installed <a href="https://github.com/longld/peda">gdp-peda</a> which will help to understand the disassembly better.

So first of all I set the disassembly to intel flavor using the following command:
```r
$ set disassembly-flavor intel 
```
After this I used the command <b>disas main</b> to disassemble the main function.

<img src="/assets/img/uploads/assembly-01/0.webp">

Now let's set a breakpoint to the <b>main</b> function, this simply means that here we told the gdb to stop the execution whenever it hits the main function and after that we will analyze different registers. We have set the breakpoint, now let's run the program.

<img src="/assets/img/uploads/assembly-01/1.webp">

As you can see in the above image there are so many registers and each register has its own purpose. Let's understand them one by one.

**EBP**

We start with EBP i.e. Base pointer, EBP points to the higher memory address at the bottom of stack and as far as I have studied about this register, it generally stores the ESP, which means the top of the stack keeps changing (esp). So to perform some operations we can store esp in ebp using the <b>mov</b> instruction: 

```r
mov ebp, esp
```
**ESP**

This register points to the top of the stack and as the program adds data to the stack. It grows from higher memory address to lower memory address and as we know that different operations can be performed on the stack like push and pop, so accordingly the stack will grow and shrink.

**EIP**

This register points to the address of the next instruction to be executed. So in this particular example, EIP points to the address <b>0x565561b6</b> and what basically going to happen is when this instruction will be executed, the value <b>0x00</b> will be stored in the address <b>[ebp-0xc],</b> so let's first analyze this memory address before execution of instruction. 

```r
gdb-peda$ print $ebp-0xc $1 = (void *) 0xffffd1bc gdb-peda$ x/b $1 0xffffd1bc:	0x0d
```
As you can see that currently this memory address is holding some random value. Now by using the command <b>nexti,</b> we are going to execute the instruction.

<img src="/assets/img/uploads/assembly-01/2.webp">

As you can see in the above image that the EIP is now pointing to the memory address <b>0x565561bd.</b> Earlier we have examined the address <b>[ebp -0xc]</b> before the execution and now let's examine it after the execution (It should contain the value <b>0x00</b> instead of <b>0x0d,</b> please verify it by yourself.)

The next instruction to be executed looks something like:

```r
0x000011bd <+36>:	jmp    0x11d5 <main+60>
```
So now let us try to understand this instruction. Here we can see JMP is used, this means that now the program flow will be directed to the memory address <b>0x565561d5.</b> Now by using the command <b>nexti</b> we can execute this instruction and now program flow will be directed to the specified address .

<img src="/assets/img/uploads/assembly-01/3.webp">

Okay, so That's it for today, we will continue this program in the next article and will try to understand what this program is actually doing. I have shared the screenshot of the disassemble of main so you can also try yourself to understand this program. 

I am a beginner in this field so if you find any mistakes or have any query, then please let me know, you can contact me <a href="/contact">here,</a> or send an email to <a href="mailto:yashsaxena986@gmail.com">yashsaxena986@gmail.com.</a>

NOTE: The awesome artwork used in this article was created by <a href="https://dribbble.com/seandockery">Sean Dockery.</a>