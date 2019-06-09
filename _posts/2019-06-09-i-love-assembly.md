---
layout: post
title: I Love Assembly
date: 2019-06-09 01:05 +0530
---

![🤯🤯](\assets\iloveasm\1.png)

I apologize, the title might’ve been misleading. The objective of this post is to write a simple program that’ll spit out “I ❤ Assembly” to the console. And of course, we’ll use Assembly to do this.

In most languages used today this’ll be a one or two line snippet. But in Assembly, it takes a little more than that. No worries though, by theory it’s the same as in any other language. All we have to do is instruct the operating system what to print.

![It really is, Once the concepts are clear! ](\assets\iloveasm\2.gif)



# iLoveAsm.asm

Open up your favorite text editor and type out the following.

```
BITS 64
section .data
    message DB "I ❤ Assembly", 10
section .text
global _start
_start:
    mov RAX, 1
    mov RDI, 1
    mov RSI, message
    mov RDX, 16
    syscall
    
    mov RAX, 60
    mov RDI, 0
    syscall
```

Okay, this'll intimidate some one for sure. I was. But let me say, even though this looks complicated for a program that just prints a line of text to the console, it isn't. Only two high-level actions are performed by this code. This is it's anatomy, if I may say so.

As you may see, the above block of code is divided into two *sections*: **data** and **text**.

The **data** section contains the data that the program will access when it executes. When the program is run, the contents of this section will be loaded into memory and made available to the program. In this case, the data section contains a string of text called ***message***, which we'll be printing out to the console.

The **text** section is where all the logic (code) goes. This is a list of Assembly mnemonics/instructions that tells the computer what to do. In this case, it'll tell the computer to print out the ***message*** to the console.

To properly understand the program, we'll break the program down and examine each line one after the other.

`BITS 64`

The above instruction or more accurately directive specifies that NASM should generate code designed to run on a processor operating in 64-bit mode. NASM is the "assembler" or the program that converts our program to a machine understandable format, sorta like a compiler.

`section .data`

This is where you can provide static data of any kind for the program. When the program runs, the OS will load this into memory. It'll be made available to the program to read and/or manipulate.

`message DB "I ❤ Assembly", 10`

This line declares a string called message, with a value of "I ❤ Assembly". The said line is comprised of the following parts.

- message - The name of the variable that'll be used later to refer to the string being declared.

- **DB** - This is the data type. In this case, DB means that a series of bytes are being declared (DB - Define Byte). Bytes are 8-bit integers that range from 0–255. Which means that each byte can be represented by a single ASCII character. There are other data types that refer to larger integers like **DW** (Define Word), **DD** (Define Doubleword), **DQ** (Define Quadword) etc.
- **"I ❤ Assembly"** - This is the data that message refers to, and is what we'll print out to the console later. Each letter, comma, space and exclamation point will account one byte. The Heart emoji though is a combination of 4 bytes. (❤ - U+2764)
- **, 10** - This is the ASCII code for the new line character. It is appended to the end of the "I ❤ Assembly" text. It is comparable to the "\n" in other languages which can be used as "I ❤ Assembly\n".

`section .text`

This marks the end of the data section and the beginning of the text section, which is where the code goes. The code is a series of instructions which control the computer and modify its state. This is done by setting a number of registers which will tell the host OS (Linux) what we want to do, and then instruction it to do it once we're ready. This will require a number of LOCs, but the make a single operation.

`global _start`
`_start:`

These two lines define the entry-point of the program. The first instruction following the **_start:** will be executed first. This is how a computer knows where to start executing a program.

`mov RAX, 1`

This is the first real LOC: the first instruction which the processor will actually execute. Breaking it down…

**mov** is the instruction type, assembly mnemonic or *opcode* (operation code). It's a direct command, which tells the processor what kind of action to carry out. In this case, `mov `stands for "move". It's used to move data and values around between registers and memory.

**RAX** refers to a register. `RAX `is a 64 bit register. The 32 bit alternative is aptly named as `EAX`. This is a temporary storage location, which values can be written to, read from, or operated on. There are lots of registers: `RAX`, `RBX`, `RCX`, `RDX`, `RBP`, `RSP`, `RDI`, `RSI`, etc. Many of them have special purposes or are reserved for certain things, but `RAX `is a **general purpose register**. This means it can be used by a programmer for whatever they need. You can think of registers as built-in global variables if you're comfortable with high-level languages. In fact, they're significantly faster than typical variables, because they're not located in system memory: they're located inside the processor itself.

**1** is an immediate value, which means the value "`1`" is encoded directly into the instruction. We're not moving information from a register or from a memory address, it's just the literal value "`1`".

All in all, this instruction tells the processor to plug the value 1 into the register RAX. It can be read as "move 1 into `RAX`". If you're used to higher level programming languages, you can think of it as `RAX = 1`.

After this instruction executes, `RAX` will be set to the value 1. We do this because we're preparing to make a call to the operating system. 1 stands for **sys_write**, which tells the operating system to write some data to a file or stream. `RAX `is the register Linux checks to figure out what a program wants it to do and it stores the return value of a function call as well.

------

The next 3 LOCs will supply the arguments required for the ***sys_write()*** syscall. If you google sys_write, you'll find that it accepts 3 arguments.

`sys_write(unsigned int fd, const char * buff, size_t count)`

The first argument is the file descriptor ***fd*** where data is written to, held in the second argument ***buff*** which is of size ***count***. Let's work on supplying those arguments now.

`mov RDI, 1`

This is very similar to the first line of code. Except that the register we're moving the value to is RDI. In this context, the value of 1 stands for STDOUT, which will be passed as the file descriptor to the system call to sys_write.

`mov RSI, message`

Okay, by now, we've told Linux that we want to write some data (`RAX`=1) to console/STDOUT (`RDI`=1). The above LOC tells Linux what to write. Here we set RSI to the beginning of the "I ❤ Assembly" text. When Linux executes our program, it'll find the location in memory where the ***message*** string starts.

`mov RDX, 16`

Now, we tell Linux how many bytes of ***message*** to write. This is to signal the end of the string to Linux. Let me tell you why we did that. We didn't set `RSI` to the entire string "I ❤ Assembly". We only set it to the start of the string. Registers in 64-bit capable computers only have 64 bits or 8 bytes each. The entire string is 16 bytes in total, which means that `RSI `won't be able to store the entire string in it. Instead, it stores the memory location of the first letter, which is 64 bits in length and it continues to read the next address until the end. This end of data is defined by the 3rd argument to `sys_write`.

If this register is set to a higher number that the correct amount, Linux will access memory that it isn't supposed to access 👿. Try setting the size to an arbitrary value to see your program print out data that it isn't supposed to access 😎.

`syscall`

Okay, we're all set. Only thing left to do now is to call Linux. This line lets the operating system know that there's some work for it to do and the details of the operation is loaded to the relevant registers.

When the above call is executed, control is passed from the User Mode where our program is running to the Kernel Mode. System calls makes this possible.

------

The next part will deal with exiting the program safely.

`mov RAX, 60`

Once the sys_write syscall is completed, the `RAX `register is again set to another value, **60**. You may ask what that is for. As it happens, **60** is a system call that corresponds to `sys_exit()` in 64-bit Linux. This tells the OS that the program is now safely exiting.

`mov RDI, 0`

Next, we set `RDI `to 0. If you're conversant with higher level languages, this corresponds to ***return 0***. This value will be used as the argument to `sys_exit`. 

`sys_exit(0)`

What do we do next? **SYSCALL**!! 😃

`syscall`

This will signal the end of the program. And it'll exit the program safely. For the curious ones out there, skip the above 3 lines and try to run the program. It'll work, sure, but with a Segmentation Fault.

![segfault](\assets\iloveasm\3.png)

# Running the program

Save the above program to disk. I've saved mine as *iLoveAsm.asm*.

![iLoveAsm.asm](\assets\iloveasm\4.png)

In order to create an executable, there's **2** steps that has to be completed in order.

First we'll use an assembler to convert the program into machine code. A program called ***nasm*** ***(Netwide Assembler)*** can be used to do this in Linux.

`nasm -f elf64 iLoveAsm.asm`

This runs the ***nasm*** program, telling it to assemble the file "iLoveAsm.asm". The contents of "iLoveAsm.asm" will be converted from assembly language into machine code and the output will be written to a new file called "iLoveAsm.o". This is called an object file.

The `-f elf64` option tells nasm that this is a Linux based 64-bit program.
If nasm reports any errors, check that you typed the program exactly as it appears above and try again. If it's successful, there won't be any output.

The file "iLoveAsm.o" should now be present alongside "iLoveAsm.asm". This is the machine code version of the program.

However, it's still not quite ready to run. In order for the operating system to run it, we have to use a linker to convert it into an executable file format. In this case, we'll be using the GNU linker (called ***ld***) to turn our object file into an executable file ready for Linux to run:

`ld iLoveAsm.o`

This runs the program `ld` and tells it to take our object file "iLoveAsm.o" and make it into an executable file. By default it will name this new executable file "a.out".

Now for the moment of truth: run the program!
`./a.out`

If everything went well, you should see the following output:

![voila!](\assets\iloveasm\5.png)

Hope this helps!! See ya!

[Medium Article](https://medium.com/@amod077/i-love-assembly-868ea263137f?source=friends_link&sk=b9fe34ebe75df7ca573d3c2adb53e09e)