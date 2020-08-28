---
date: 2020-08-27 01:55:11
layout: post
title: "Assembly Language Series: 0x02"
subtitle:
description: This article is the second part of the Assembly Language Series. In this we will complete the disassembly of our Hello World program.
image: /assets/img/uploads/assembly-02/robot.jpg
optimized_image: /assets/img/uploads/assembly-02/robot-optimized.jpg
category: exploitation
tags: binary-exploitation
author: yashsaxena
paginate: false
keywords: assembly, assembly language tutorial, disassembly hello world, infosec articles, assembly language series, disassembly using gdb, assembly language introduction, binary exploitaiton
---

Today I will be continuing the topic which I published in the last article of the Assembly Language Series. First let's recall what I have done till now. In the last article, I have discussed different types of useful registers like <b>EBP, ESP, EIP</b> and we tried to understand some of the instructions using gdb and according to the last article this is the scenario we have right now. If you have not read that article yet, I recommend reading it first <a href="https://infosecarticles.com/assembly-language-series-0x01/">here.</a>  

<img src="/assets/img/uploads/assembly-02/00.webp">

By using the command <b>nexti</b> I am going to execute the instruction pointed by <b>EIP.</b>

```r
[ebp-0xc ] => 0x00

EIP => 0x565561d5
```

Okay, So let's try to understand this instruction first:

```r
EIP: 0x565561d5 (<main+60>:   cmp    DWORD PTR [ebp-0xc],0x9)
```

Here <i><b>cmp</b></i> is used, this means that <i>gdb</i> is going to compare two values present at these two addresses. The 
value present at <b>[ebp-0xc]</b> is <b><i>0x9,</i></b> and from the previous <a href="https://infosecarticles.com/assembly-language-series-0x01/">article</a> we know that value at <b>[ebp-0xc]</b> is <b><i>0x00.</i></b> So it will compare the value <b><i>0x00<i></i> and <b><i>0x9.</i></b>

<img src="/assets/img/uploads/assembly-02/01.webp">

These two values was compared and now the <b>EIP</b> points to the next instruction:

```r
EIP: 0x565561d9 (<main+64>:   jle    0x565561bf <main+38>)
```

Here <b>jle</b> means, If the value present at <b>[ebp-0xc]</b> is less than equal to <b><i>0x9</i></b> then jump to 
the address <b>0x565561bf,</b> and we already know that condition is true. Therefore the program flow will be redirected to the specified address, let's check it by using the command <b>nexti.</b>

<img src="/assets/img/uploads/assembly-02/02.webp">

Umm this means we are going in the right direction and after that, some instructions don't look to be useful for us. Still let me explain to you what is <b>lea</b> here.

<b>lea</b> stands for load effective address and there is a small difference between <b>lea</b> and <b>mov</b> instruction that is, unlike the <b>mov</b> instruction, <b>lea</b> instruction stores the address directly to the destination instead of storing the value contained in that address.

<img src="/assets/img/uploads/assembly-02/03.webp">

Address stored in <b>EAX</b> register before executing the <b>lea</b> instruction is <b><i>0xf7fad808.</i></b> Now lets execute the next instruction and then we will check the address at <b>EAX</b> again.

<img src="/assets/img/uploads/assembly-02/04.webp">

Now we can see that the address stored at <b>EAX</b> is <b><i>0x56557008,</i></b> now in the next instruction this address will be pushed at the top of the stack.

<img src="/assets/img/uploads/assembly-02/05.webp">


In the next instruction, the program is calling <b>puts</b> function and one of my friends told me that <b>printf()</b> is not visible in the disassembly. Instead, it's being replaced by the <b>puts()</b> in line <b><main+11></b> The reason why this happens is because <b>puts()</b> is faster than <b>printf()</b> as it doesn't parse the string passed to it. So, due to compiler optimization, the compiler automatically replaces the <b>printf()</b> call with a constant string as argument with <b>puts()</b> call.

<img src="/assets/img/uploads/assembly-02/06.webp">

This is how "Hello world" is printed on the display and in the next instruction program is adding 1 in the value present at address <b>[ebp-0xc].</b>

```r
0x565561d1 <main+56>: add    DWORD PTR [ebp-0xc],0x1
```
Now the value <b><i>0x01</i></b> will be compared to the value <b><i>0x9</i></b> and again the condition is true. Therefore a jump will be taken and program flow will be redirected to the address <b><i>0x565561bf.</i></b>

<img src="/assets/img/uploads/assembly-02/07.webp">

This cycle will be repeated till the value is less than equal to <b><i>0x9</i></b> and "Hello world" will be printed 10 times before the program exits. So we can predict that program is something like this:

```r
#include <stdio.h>
int main()
    {
        int i;<br>
        for(i=0;i<10;i++)
        {
            printf("Hello world\n");
        }
    }
```

Thanks for reading , I hope you like my explanation. If you find any mistakes or have any query, then please let me know, you can contact me <a href="/contact">here,</a> or send an email to <a href="mailto:yashsaxena986@gmail.com">yashsaxena986@gmail.com.</a>

NOTE: The awesome artwork used in this article was created by <a href="https://dribbble.com/sirmon">Paul Sirmon.</a>

