

The two-step process for handling system call (described previously) seems to be a bit too complex. But is it too expensive to implement?

We know from our study of the assembly language that some instructions can be slower than others. For instance, the `jal` and `j` instructions are much faster than any load instruction. Luckily for us, the `syscall` instruction is rather fast; it is only minimally more expensive than a division instruction.

However, while the `syscall` instruction may be fast, the system call themselves are rather slow. This is because of the work that follows the initial `syscall` instruction.

## Context Switch
> Switching from one running process to another

Recall that when we made function calls with the `j` instruction, we had to store and restore registers (following the calling convention). Often, we pushed the saved registers onto the stack and popped them just before we returned from a function. Although this was a lot of work (computationally), it was necessary because we only had one set of registers that needed to be shared among multiple functions. This was why following the calling conventions was so crucial a function implementers.

However, when we used the `syscall` instruction, we did not ever need to worry about saving registers and restoring them. How is this possible? A majority of the operating system code is still built from the user mode instructions which accesses and modifies the same underlying registers. Wouldn't that mean a system call, like function calls, need to preserve the state of registers?

This is exactly what the system call does (unbeknownst to the programmer) in the background. Unlike function calls, however, the system call needs to preserve the state of *all* general purpose registers as there are no calling conventions for system calls. This state is called **context** and preserving these is what makes system calls so slow.

### Writing context to memory
When we were dealing with functions, we were trying to preserve a subset of the context (depending on the *save registers* defined in the calling conventions). This meant that sometimes we could do some optimizations to improve the speed of our program. For instance, instead of pushing the state of the save registers to the stack, we could sometimes shift around the values of the registers to avoid the long memory write/load times. However, a system call needs to preserve the entire context, which means that this is not possible. As such, a system call needs to store the state of all registers into memory which makes the system call even slower.


## Memory Hierarchy
Back to context switch. We said that since there are no safe registers, we must store preserve all register values to memory. But, to which memory?

Due to the Von Neumann architecture, our instructions must be loaded onto memory, which means that our instructions can only run as fast as our hardware (memory speed). It doesn't matter how fast we can add two numbers, if writing/reading those numbers is slow. Thus, to improve performance, we need faster memory. 

Now, since the speed of light is finite, electrical signals can only move so fast. Therefore, the physical distance between the memory and the CPU matters. If the distance is far, it will take longer for signals to move, thus slower read/write time. Hence, is the need for registers. Registers are built as close as possible to the CPU (they are actually inside the CPU) to maximize their speed.

Furthermore, they are a type of Static RAM built from flip-flops, etc. which are faster but more expensive (they require more transistor to store each bit).

In contrast, dynamic RAM uses a threshold approach. Generally, the bits of dynamic RAM each acts like little leaky buckets of water, where a full bucket might be a 1 and an empty one a 0. However, with this approach, the DRAM needs to be needs to be flushed when we see that the full buckets have gotten too low. This adds overhead and makes it slower.![](Memory%20Diagram.png)
Therefore, if we are writing 32 registers to the DRAM to perform a context switch, it will take a lot of time. We can examine this in a simple java program with many print statements (many system calls). The program will run generally slow, but once we remove the print statements, it will run noticeably faster.

Thus, system calls are expensive not because of the control transfer, but because of the large context. There are engineering trade-offs. Faster memory is often smaller, closer to the CPU, and more expensive. Hence, to balance the speed and storage necessitates the existence of cache.

Later we will see that we can store the context in magnetic disk/solid state drive. Although hard disk drives are terribly slow, they are non-volatile and can hold huge amounts of information, which will be useful in certain cases. However, in most use cases, the performance penalty from latency is too big; Thus, if we are storing our context on a disk, we've probably failed at our job as the OS.

#### Context switch measures performance
We've seen that context switches incur costs due to hardware speed, etc. We could try and make the context switches faster, but for this course, we will assume that the speed of context switches has already been minimized by the computer architecture experts (who optimize the architecture), and OS implementers (who design the OS to not save any unnecessary context such as floating point registers). Thus, there is nothing we can do to reduce the speed of context switch.

Furthermore, the speed of a context switch will vary and depend on the hardware. Thus, using actual time measurements will be unreliable. Instead, we can use the number of context switches to argue whether our implementation is fast or slow (since we can control the number of context switches).

Much like asymptotic analysis (from data structures), context switches will be the means to analyzing performance. However, we will soon discover that this is very difficult and sometimes impossible due to practical reasons.

Luckily, from a programmer's perspective, the details of context switches are abstracted. That is, a `syscall` merely looks like a very long instruction.


[^rings] Some architectures have more than 2 partitions for their instruction sets. Most notably, the x86-64 architecture has four 'rings' ranging from ring 0 to ring 3.

[^kernel] The kernel refers to the core space of operating systems instructions. Often times, the word *kernel* is used interchangeably with the word *operating system*.