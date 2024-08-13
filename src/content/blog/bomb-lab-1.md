---
title: "Bomb Lab (1/2)"
description: "Bomb Lab 的作业报告（第一部分）"
date: 2024-08-08
tags: ["CS:APP"]
---


This post is my personal report on the Bomb Lab assignment. It covers topics such as assembly language instructions, disassembly, and the use of the GDB debugger.

## Preparation

For an introduction to the assignment, check out the [Writeup file](https://csapp.cs.cmu.edu/3e/bomblab.pdf). After downloading the source code materials from the [official website](https://csapp.cs.cmu.edu/3e/labs.html), extract the package using the following command:

```bash
tar -xvf bomb.tar
```

After extracting, the directory structure should look like this:

```
bomb/ 
├── bomb 
├── bomb.c
└── README
```

First, take a look at the `bomb.c` file. In each phase, it reads input and then processes it through functions named `phase_[NUM]` before executing the `phase_defuse()` function. The functions that detonate the bomb are encapsulated, so our task is to analyze each of the six `phase_[NUM]` functions one by one.

As per the assignment requirements, we create a file named `psol.txt` to record the answers for each part.

Next, we'll use the GDB debugger. Enter the TUI interface with the following command:

```bash
gdb -tui bomb
```

## Phase 1

Disassemble the `phase_1` function:

```bash
(gdb) disas phase_1
```

> `disas` is short for `disassemble`.

The output looks like this, with analysis following:

```text
Dump of assembler code for function phase_1:
   0x0000000000400ee0 <+0>:     sub    $0x8,%rsp
   0x0000000000400ee4 <+4>:     mov    $0x402400,%esi
   0x0000000000400ee9 <+9>:     callq  0x401338 <strings_not_equal>
   0x0000000000400eee <+14>:    test   %eax,%eax
   0x0000000000400ef0 <+16>:    je     0x400ef7 <phase_1+23>
   0x0000000000400ef2 <+18>:    callq  0x40143a <explode_bomb>
   0x0000000000400ef7 <+23>:    add    $0x8,%rsp
   0x0000000000400efb <+27>:    retq
End of assembler dump.
```

1. The function that detonates the bomb is called `<explode_bomb>`.
2. At <+14> and <+16>, if the return value of `<strings_not_equal>` is non-zero, it skips the detonation at <+18>.
3. Thus, the input must be a string that matches the string content at <+4>.

Randomly input a string in the first line of the input file and run the code.

> Use `layout asm` to enter the disassembly instruction view.

First, set breakpoints:

```bash
(gdb) b phase_1
(gdb) b explode_bomb
(gdb) r psol.txt
```

> You might need to enter `refresh` to refresh the TUI view.

Input `si` to step through instructions, and continue with `si` until the next breakpoint (or <+14>). When you enter `<strings_not_equal>`, input `finish` to end the run.

> `si` allows step-by-step execution of instructions, and you can repeat the last command by pressing RETURN. We'll repeat this process later.

Access the contents of the `%esi` register or directly output the content at `$0x402400`:

```bash
(gdb) x/s $esi
(gdb) x/s 0x402400
```

The output is:

```text
0x402400:       "Border relations with Canada have never been better."
```

Copy this content into our input file, insert a new empty line, and rerun the program.

> If you don't insert a new empty line, the bomb will detonate. According to the document, the program reads continuously until it encounters an EOF marker. Without an EOF (RETURN here), it will fail.

During this, you can check the value of `%eax` with `(gdb) p $eax`. If the output is `$1 = 0`, our input is correct.

> You don't need to exit GDB to rerun the program. After saving the modified `psol.txt`, run `(gdb) r psol.txt`.

## Phase 2

The disassembled output is:

```text
Dump of assembler code for function phase_2:
   0x0000000000400efc <+0>:     push   %rbp
   0x0000000000400efd <+1>:     push   %rbx
   0x0000000000400efe <+2>:     sub    $0x28,%rsp
   0x0000000000400f02 <+6>:     mov    %rsp,%rsi
   0x0000000000400f05 <+9>:     callq  0x40145c <read_six_numbers>
   0x0000000000400f0a <+14>:    cmpl   $0x1,(%rsp)
   0x0000000000400f0e <+18>:    je     0x400f30 <phase_2+52>
   0x0000000000400f10 <+20>:    callq  0x40143a <explode_bomb>
   0x0000000000400f15 <+25>:    jmp    0x400f30 <phase_2+52>
   0x0000000000400f17 <+27>:    mov    -0x4(%rbx),%eax
   0x0000000000400f1a <+30>:    add    %eax,%eax
   0x0000000000400f1c <+32>:    cmp    %eax,(%rbx)
   0x0000000000400f1e <+34>:    je     0x400f25 <phase_2+41>
   0x0000000000400f20 <+36>:    callq  0x40143a <explode_bomb>
   0x0000000000400f25 <+41>:    add    $0x4,%rbx
   0x0000000000400f29 <+45>:    cmp    %rbp,%rbx
   0x0000000000400f2c <+48>:    jne    0x400f17 <phase_2+27>
   0x0000000000400f2e <+50>:    jmp    0x400f3c <phase_2+64>
   0x0000000000400f30 <+52>:    lea    0x4(%rsp),%rbx
   0x0000000000400f35 <+57>:    lea    0x18(%rsp),%rbp
   0x0000000000400f3a <+62>:    jmp    0x400f17 <phase_2+27>
   0x0000000000400f3c <+64>:    add    $0x28,%rsp
   0x0000000000400f40 <+68>:    pop    %rbx
   0x0000000000400f41 <+69>:    pop    %rbp
   0x0000000000400f42 <+70>:    retq   
End of assembler dump.
```

At first glance, `<read_six_numbers>` suggests that we need to input six numbers. I randomly input:

> I chose 111, 222, 333, 444, 555, 666 for their high recognizability, making it easier to track my input in the registers.

Upon examining the instructions, it’s clear that there are many jump instructions. By setting breakpoints at `<explode_bomb>`, we can cautiously step through the instructions.

At <+14>, there is a comparison, and at <+18> and <+20>, the bomb will detonate if the values are not equal. `$0x1` obviously means 1, so let's print the value at `%rsp`:

```bash
(gdb) p *(int *)$rsp
```

> Since we know the input data is an integer, we dereference and cast it to an integer pointer.

We find our first input needs to be 1.

Next, at <+32>, the process is similar. Using the same method to check `$eax` and `(%rbx)`, we find the second input must be 2. Similarly, the third input is 4, the fourth is 8, and so on.

Thus, the answer for this phase is `1 2 4 8 16 32`.

> Checking `%eax` can be done in two ways: `p $eax` or continuously tracking its value with `display $eax`. Since we are continuously accessing two registers, it’s better to use `display` instead of `p`. To stop tracking, use `undisplay`.

Apart from automatically running the program, we can manually analyze it. The instructions clearly contain a loop.

Given the structure of the `%rsp` register, at <+2>, it reaches the stack bottom. All subsequent steps climb upwards.

> `%rsp` is the stack pointer register, following the LIFO principle. `%rsp` manages function calls and can store local variables.

The bottom of the stack should be `0x1`, or 1, otherwise, it will detonate in step 2. The third step jumps to <+52>.

From <+52> to <+27>, the value of register `%eax` is equal to `%rbx-4` (i.e., the stack bottom of `rsp`, which is 1). After <+52> and <+27>, the offsets cancel each other. In <+32>, we find the value at the top of the stack is `1 + 1 = 2`.

The rest of the process is similar. After reaching <+41>, it jumps to <+48>, then to <+27>... The flowchart looks like this until `%rbx` increments to the top of the stack, exiting the loop (see <+41>):

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/%E6%9C%AA%E5%91%BD%E5%90%8D%E7%BB%98%E5%9B%BE.drawio%20(2).png)

The loop translates into the following C statement[^2]:

```c
// int rsp[6], rbx = rsp+1, rbp = rsp+6; int eax;
while (rbp != rbx)
{
	eax = *(rbx-1);
	eax = eax + eax;
	if (eax == *rbx) 
		continue;
	else 
		explode_bomb();
}
```

## Phase 3

The disassembled result:

```text
Dump of assembler code for function phase_3:
   0x0000000000400f43 <+0>:     sub    $0x18,%rsp
   0x0000000000400f47 <+4>:     lea    0xc(%rsp),%rcx
   0x0000000000400f4c <+9>:     lea    0x8(%rsp),%rdx
   0x0000000000400f51 <+14>:    mov    $0x4025cf,%esi
   0x0000000000400f56 <+19>:    mov    $0x0,%eax
   0x0000000000400f5b <+24>:    callq  0x400bf0 <__isoc99_sscanf@plt>
   0x0000000000400f60 <+29>:    cmp    $0x1,%eax
   0x0000000000400f63 <+32>:    jg     0x400f6a <phase_3+39>
   0x0000000000400f65 <+34>:    callq  0x40143a <explode_bomb>
   0x0000000000400f6a <+39>:    cmpl   $0x7,0x8(%rsp)
   0x0000000000400f6f <+44>:    ja     0x400fad <phase_3+106>
   0x0000000000400f71 <+46>:    mov    0x8(%rsp),%eax
   0x0000000000400f75 <+50>:    jmpq   *0x402470(,%rax,8)
   0x0000000000400f7c <+57>:    mov    $0xcf,%eax
   0x0000000000400f81 <+62>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f83 <+64>:    mov    $0x2c3,%eax
   0x0000000000400f88 <+69>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f8a <+71>:    mov    $0x100,%eax
   0x0000000000400f8f <+76>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f91 <+78>:    mov    $0x185,%eax
   0x0000000000400f96 <+83>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f98 <+85>:    mov    $0xce,%eax
   0x0000000000400f9d <+90>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f9f <+92>:    mov    $0x2aa,%eax
   0x0000000000400fa4 <+97>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400fa6 <+99>:    mov    $0x147,%eax
   0x0000000000400fab <+104>:   jmp    0x400fbe <phase_3+123>
   0x0000000000400fad <+106>:   callq  0x40143a <explode_bomb>
   0x0000000000400fb2 <+111>:   mov    $0x0,%eax
   0x0000000000400fb7 <+116>:   jmp    0x400fbe <phase_3+123>
   0x0000000000400fb9 <+118>:   mov    $0x137,%eax
   0x0000000000400fbe <+123>:   cmp    0xc(%rsp),%eax
   0x0000000000400fc2 <+127>:   je     0x400fc9 <phase_3+134>
   0x0000000000400fc4 <+129>:   callq  0x40143a <explode_bomb>
   0x0000000000400fc9 <+134>:   add    $0x18,%rsp
   0x0000000000400fcd <+138>:   retq   
End of assembler dump.
```

At first glance, <+50> to <+118> appears to be a `switch-case` statement. Let’s first study the preceding part.

This problem makes it impossible to directly determine the number and format of the input. Here, I copied the answer from the second phase as a test. After stepping through the instructions, the program reaches <+24> and calls the `sscanf` function, then at <+29> compares the return value, requiring it to be greater than 1, or the bomb will detonate.

Reading the `sscanf` function documentation, the return value is the number of successfully matched and assigned input items, so we need to input at least two numbers.

> Type `man sscanf` in the command line to read the documentation: "On success, these functions return the number of input items successfully matched and assigned; this can be fewer than provided for, or even zero, in the event of an early matching failure."

Moreover, checking `p $eax` reveals that the return value is 2, which is acceptable. This means the problem requires only two inputs, so we retain the first two inputs.

At <+39>, if `rsp+8` is greater than 7, it jumps to <+106> and explodes. Check the variable at that position:

```bash
(gdb) p *(int *)($rsp + 0x8)
```

It turns out to be our first input, so it must not exceed 7. Our current input meets this requirement.

After stepping through the instructions to <+118> and <+123>, the bomb will explode if `(%rsp + 0xc)` and `%eax = $0x137` are not equal. The first number is our first input, so convert `$0x137` to decimal, which is 311. Hence, the second input must be 311 (when the first input is 1).

> Actually, the function calls at <+4> and <+9> already hint at this. The first input is stored at `0xc(%rsp)`, and the second at `0x8(%rsp)`.

Therefore, one solution to this phase is `1 311`.

> Note that the following part is a switch structure. Trying different values for the first number (within the range and unsigned) will yield different second values.

[^2]: [Analysis of Bomblab by Xu Yichang](http://home.ustc.edu.cn/~xuyichang/blogs/bomblab.html)