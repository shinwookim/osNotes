# System Calls
We saw before that the operating system lies between user-level software and the hardware as an intermediary between the resource requestor (i.e., programs) and the resource.

The precise manner in which a program communicates with the operating system to make those requests is called a **system call**. You've seen system calls if you've programmed using low-level assembly such as MIPS. System calls were used for input/output, process management (such as terminate), and perhaps even system-level randomness (for cryptography).

In a more broader sense, however, a system call is how a program asks an operating system to perform a task (such as read user input) on its behalf. As such, we can think of a system call to be similar to a control transfer (like the `jal` instruction in MIPS).

# A Deep Dive at `Hello, World!`
Lets look at a how a software uses system calls by examining a simple "`Hello, World!`" Program written in C:

```c
#include <stdio.h>
int main()
{
	printf("Hello, World!"); // <-- function call to printf()
	return 0;                // which is part of the standard library
}
```
All this program does is make use of the C standard library's `printf()` function to display a small string (`Hello, World!`) onto the standard output. As you can see, if we compiled this program and examine the assembly (MIPS), we get:
```mips
$LC0:
        .ascii  "Hello, World!\000"
main:
        addiu   $sp,$sp,-32
        sw      $31,28($sp)
        sw      $fp,24($sp)
        move    $fp,$sp
        lui     $2,%hi($LC0)
        addiu   $4,$2,%lo($LC0)
        jal     printf        # <-- function call to printf
        nop
```
As expected, we see that all this program does is move some arguments and use a jump instruction to transfer control from our program's code to that of `printf()`.

However, if we were to examine the code for the `printf()` function, we will see that the majority of the code focuses on the stringification and interpolation of the argument string (using appropriate format specifiers), rather than the work for printing to the screen. In fact, by the time we reach the end of the `printf()` function, we are left with a formatted string which has not been displayed to the standard output. 

So where does the printing occur? In reality, the task of displaying the string to the screen is much more complex than it looks. To print "`Hello, World!`" to the screen, we must:
1. First, read the font[^font] and make the appropriate calculations to determine how to draw each character.
2. Next, determine where the terminal is, and what line and column to display the string[^terminal].
3. Finally, with all the information loaded into memory, manipulate the physical hardware to turn on/off the necessary pixels.
[^font]: Fonts are files which describe how each character in a character set is drawn.
[^terminal]: We may also wish to consider line-wraps, word-breaking, font colors, and a lot more.

Even for a program as simple as our "`Hello, World!`" example, the process of merely displaying some string onto the screen involves a lot of detail about the underlying hardware. As such, tasks like these are best left abstracted. And since, we are abstracting details about the hardware, this is a job for the operating system! 

Thus, just before `printf()` returns, it makes a system call after specifying the location of output, `stdout`, and the formatted string[^gdb]. In assembly, this is shown as the `syscall` instruction.

[^gdb]: Write a short program, compile it and run it through a debugger. You should be able to verify this by stepping through the program one instruction at a time. 

# System Call vs Function Call
Here, the system call works just like a function call. Like our initial call to `printf()`, the system call will do some task (for us, display to the screen) and then return to its callee.

However, there are a few key differences between the two. First, unlike function calls which used the jump instructions, system calls were made using their own dedicated instruction `sycall`.

You may have learned about the different types of instructions while studying assembly. For instance, MIPS has a J-type for instructions like `jal`. These instructions used the first six bits as the opcode and the remaining 26 bits to store an address of the function it was jumping to[^mips-syscall].

However, the `syscall` instruction does not require us to supply any operands to determine which system call we were making. Instead of an address to jump to, we provide an integer (which in MIPS, we put into register `$v0`).

The key difference between function calls and system calls are that a function call works by transferring control to a piece of code that is **addressed**, while the system call works by transferring control to a piece of code that is **indexed**.

[^mips-syscall]: As an aside, the `syscall` instruction in MIPS is technically classified as an R-type simply due to the fact that the presence of the `func` field in R-type instructions supports more possible instruction bit patterns.

# Dual Mode
We've seen that a program can utilize system calls to get the operating system to do some task (such as displaying to the screen). But, what prevents a program from bypassing the operating system and doing this task itself? After all, isn't an operating system just another piece of code?

## Multiple Instruction Sets
An instruction set defines what a processor can do. Up until now, however, you've probably only been using a small subset of what the CPU actually offers. This is because the instruction set is partitioned into at least two sets.
1. First, there is the **User mode instructions/protected mode instructions** which are the instructions that our user program runs. These are the instructions that you've likely used if you wrote programs in assembly; they are also the instructions that are generated by compiling user-level software.
2. Next, there is the **kernel mode instructions/privileged mode instructions** which are instructions that only the operating system is allowed to run. These are the instructions that allow you to directly manipulate the physical hardware (among other things).
The second set of instructions is grants the operating system its authority. These are instructions that can only be called by the kernel. 

The processor maintains the '*current mode*' inside a register (which is often done using a single-bit flag inside the machine status register) and checks it whenever a privileged mode instructions is executed.

If this flag indicates we are in **kernel mode**, we can run both privileged and protected mode instructions. If it indicates we are in **user mode**, we can only run protected mode instructions.

1 Some architectures have more than 2 partitions for their instruction sets. Most notably, the x86-64 architecture has four 'rings' ranging from ring 0 to ring 3.
2 The kernel refers to the core space of operating systems instructions. Often times, the word kernel is used interchangeably with the word operating system.


# Exceptions
So what if we attempt to execute a privileged mode instructions while we are in user mode?

The answer to that depends on what specific instruction was called and what the CPU architecture is. Historically (and least desirably), x86 has ignored these calls (which have caused problems when trying to virtualize x86). However, in many modern architectures (such as MIPS, and many x86-64 instructions), a call to an unauthorized instruction will raise an **exception** (e.g., Integer division by zero, Page fault). 

The exception invokes the operating system, which usually sends a signal to the program (which by default terminates the program).
# System Calls to Enter Kernel Mode
As said before, to run privileged mode instructions, we must be in kernel mode. So how do we flip from user mode to kernel mode?

Since, the privileged mode instructions exists to grant the operating system authority, we need to invoke operating system. For a user program, that means making a system call. 

When the user program executes a `syscall` instruction, user mode is flipped to kernel mode and the operating system code begins to execute. And once the operating system completes its execution, the return instruction sets flips the mode bit back to user mode.

Hence, a system call indicates to the processor that the upcoming instructions belong to the operating system and is allowed to run in kernel mode.

# Interrupts
When a user program makes a system call, the operating system stops the hardware from running user code and switches to the operating system code and complete the requested task. 

In other words, our code is **interrupted** while the operating system does the requested task, and when it's done, the control is returned to our program. This is the idea of an **interrupt**.

Much like a system call, when the CPU receives an interrupt, it stops the current program, handles the interrupts, and then returns to the user program (or crashes it). In fact, a system call is a type of interrupt called **trap**; however, an interrupt is much larger than a system call.

Grouping by the source, there are:
- **Software Interrupts** (I.e., Trap, Exceptions) which come about because a user program has requested it.
- **Hardware Interrupts** which comes about because a hardware device has requested it.
Regardless of the origin, however, the handling of interrupts occur in the same way. First, the processor first sets the program counter to the memory address of the operating system code which can handle the interrupt. This memory address comes from a structure on the chip called the **interrupt vector**, which is indexed by the particular type of interrupt. At each index is an address which points to the operating system code which can handle the interrupt. 

For instance, on this chip, all system calls are mapped onto the same entry (usually at index 0, but also at index 128 in the x86 architecture). Once the operating system is invoked, the processor can check a second table (called the **system call table** in Linux) by indexing it using the ordinal in a register such as `$v0`. The entry in the system call table will contain the memory address of specific code which can handle the particular system call.

Hence, system calls are dispatched in a two-step process: first, by reading the entry in the interrupt vector and invoking the operating system code[^invoke-os] and next, by reading the entry in the system call table and transferring control to the code which can handle the particular system call.

[^invoke-os]: The memory address found inside the interrupt vector is loaded onto the program counter, and the instruction privilege is escalated to allow for the operating system to execute kernel mode instructions.

Once the system call operation is completed, instruction privilege is once again dropped to user mode, and the processor returns control to where the system call originated.
