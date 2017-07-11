---
layout: post
title: History of system calls - using a software interrupt
---

Pourquoi ?
pour la portabilité
les call gates, mais pas très répandu, et pas d'instruction spécifique

# Support

Software interrupts have been used to make system calls since `Windows NT`(NT 3.1, July 1993)
and Linux

# Usage

A program that wants to make a system call throught a software interrupt
has to perform a code like the following one:


{% highlight nasm %}
mov eax,1   ; system call number
int 80h     ; 0x80, interrupt vector in the IDT
{% endhighlight %}



# Linux

`INT 0x80`

# Windows

`INT 0x2E`

# References

https://stackoverflow.com/questions/15168822/intel-x86-vs-x64-system-call
https://stackoverflow.com/questions/3425085/the-difference-between-call-gate-interrupt-gate-trap-gate
