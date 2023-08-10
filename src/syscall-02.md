# Costs of a System Call

# Cost of a System Call
Notice that the two-step dispatch for handling system call is rather complex. But, can it actually impact the performance of our system? 

If you've programmed using the assembly language, you may have noticed that some CPU instructions are faster than other. For example, the jump instructions (such as `jmp` and `jal`) are often much faster than instructions that require reading from the memory (such as load instructions). So let's examine the speed of the `syscall` instruction:

If you run some benchmarks, you'll quickly find that the `syscall` is a rather fast instruction[^syscall-speed]. However, while the `syscall` instruction itself may be fast, the actual system call takes a longer time[^verify-syscall-slow]. Next, we'll explore why this is.

[^syscall-speed]: The `syscall` instruction in most architecture is comparable in speed to the division instruction.
[^verify-syscall-slow]: One way to verify this is to write a program that has many print statements (which each makes system calls). The program will run terribly slow, but once you remove the print statements, it will be noticeably faster.

## Context Switch
> Switching from one running process to another

When we make a function call using the `jmp` instruction, the data that is currently in the registers must be preserved until we return. If we were programming directly using assembly language, that means copying the values from certain registers onto main memory (in accordance with the architecture's calling convention[^mips-calling-convention]) just before and after the `jmp` instruction. This is a lot of work (computationally), but necessary as there is only one set of registers that must be shared between the function and its callee. As function implementers, following the calling convention is crucial as failure to do so could end with lost or corrupted values upon the function's return. 

[^mips-calling-convention]: In MIPS, the values of the saved registers are pushed onto the stack when the function is called, and popped when the function returns.

This is also the case in system calls. Since the majority of the operating system code is built from user-mode instructions,(privileged mode instructions are only used when they are needed), they use the same general purpose registers that a user-level software might use. As such, to properly return to the program at the end of a system call, the registers must be preserved at the start of a system call and restored at the end.

Yet, one key difference is in the number of registers that must be preserved. With functions, only the saved registers (determined by the architecture's calling convention) needed to be preserved; however, with system calls, the entire set of general purpose registers (known as **context**) must be preserved. Consequently, some optimizations that might be useful for function call (such as shifting registers around to reduce the number of load/stores) is not possible with system calls. As such, we find system calls to be much more slower than their functional counterparts due it requiring more memory read/writes.

*subset of context; define context switch*; state

## Memory Hierarchy
A system call requires us to write the context (contents of all general purpose registers) onto memory. But, to which memory do we write?

We know that due to the implications from the Von Neumann architecture, all instructions are loaded to memory before they can be fetched by the CPU. As a consequence of this, the speed of our CPU is limited by how fast we can access memory. It doesn't matter how fast we can add two numbers, if writing and reading those numbers is slow, our performance will also degrade. Hence, improving the performance of our CPU requires us to improve the performance of memory accesses.

Since the speed of light is finite, electrical signals can move only so fast. Therefore, the distance between the memory and the CPU's computation unit matters. If this distance is far, it will take more time for the electrical signals to travel, eventually leading to slower memory access performance. As such, hardware designers put small static RAM chips, called **registers** as close as possible to the CPU's computational components. These registers, additionally, are built from flip-flops and are extremely fast. However, this means that registers are also expensive (as they require significant number of transistors to store just a few bits).

To overcome this problem, hardware designers also created dynamic RAM which uses a leaky memory approach. DRAM is less stable than its static counterpart, but is often much cheaper. Each bit of memory data is stored as the presence or absence of an electric charge on a small capacitor on the chip and as time passes, the charges in the memory cells leak away. As such, DRAM needs to be flushed periodically to ensure the correct values are store in it. Consequently, however, this adds additional overhead and makes accessing DRAM much slower[^cache]. 

[^cache]: In general, we find fast memory to be small, close to the CPU, and more expensive. For normal use cases, we often build caches (such as L1/L2/L3 cache) to balance the speed and store needs of the system.

Therefore, performing a **context switch** of writing 32 registers onto DRAM requires a lot of time. Hence, system calls are expensive not because of the control transfer, but because of the large context. 

Later, we will see that there are even slower types of memory in the form of magnetic disks and solid state drives. Although these are terribly slow, they are non-volatile and can hold huge amounts of information which may prove useful in specific cases. However, for context switches, their performance penalty from latency is too big in normal uses.

# Context Switches Can Measures Performance
A large proportion of the performance penalty incurred by making a system call comes from the slow context switches that is needed to preserve state. While making context switches is technically possible (up to a point), that is a job best left for experts in computer architectures. For operating system implementers, there is not a lot that can be done to reduce the time it takes for a context switch.

Additionally, even if we were to reduce the speed of the context switch for one CPU, this would not do us any good if our operating system needs to support various hardwares (such as UNIX or Linux).

Yet, one thing we can do is to use context switches to analyze the performance for our operating system. Since context switches incur such a large performance penalty, we can argue that the number of context switches a certain piece of code performs is proportional to its performance.

Lastly, the details of how the system call works (including how or if it performs context switches) is left abstracted from the user-level software. To this program, a `syscall` just appears like an instruction that takes a very long time.

