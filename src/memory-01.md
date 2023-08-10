# Memory Management
Up to this point, our discussion of how the operating system works was motivated by our desire to run multiple processes at once. Indeed, we began by discussing the various ways to achieve multiprogramming ([Processes and Threads](03%20Processes%20and%20Threads.md)), then how to choose when to run each process ([Scheduling](04%20Scheduling.md)), some issues arising from multiprogramming ([Deadlocks](06%20Deadlocks.md)), and lastly how we can solve some of these problems ([Inter-Process Communication](05%20Inter-Process%20Communication.md)).

Yet, to actually run multiple processes, there is another crucial resource that must be accounted for. As a consequence of the *Von Neumann Architecture*, the code and data of a program must be loaded in memory prior to execution. Hence, to actually run multiple processes, multiple programs must be loaded into memory. This sharing of memory means that the operating system must manage the memory as well.

## Exclusive access
In the early days of computing, memory capacity, like many other system resources, were severely limited. During this time, computer systems only had enough memory capacity to hold the code and data of only one program at a time (and perhaps a minimal operating system and device drivers, although these were often stored on their dedicated chips). As a consequence, multiprogramming was often not possible, and memory management became trivial as each process received **exclusive access** to memory. 

### Degree of Multiprogramming

However, as technology advanced, the CPU's computation speed became faster and memory capacity increased. With a faster CPU, more and more processes became I/O bound, which meant more and more processes could be run simultaneously (at least in terms of CPU usage).

The plot above depicts how many programs we can run simultaneously. We see that as processes become more I/O bound, we can run more and more processes.
![](Assets/degree-of-multiprgm.png)
To utilize this computational power for multiprogramming, the computer system had to load the code and data of multiple programs onto memory before those programs could begin running. Hence, there now was a desire to manage memory so that it could be shared by multiple processes.

Our first approach revolves around the same principle we used the share the CPU time. In our discussion of scheduling, we used *preemption* (or *time-splicing*) to allocate some of the CPU's time to each process. Now, we will devise a scheme to allocate some of the memory to each process.

## Fixed Partitions
Thankfully, as technology improved, memory capacities eventually grew enough to the point where we could put the code and data of numerous process into RAM. But this raised the question of how do we manage the part of memory each process could access? If, for example, Process *B* overwrote the part of memory storing the code for Process *A*, the results could be catastrophic!

Hence, the needs for a scheme to manage memory among processes arose. The first of these schemes is called **fixed partition**. As the name suggests, in this scheme, memory are partition into fixed and rigid regions. The sizes and number of partitions could be specified by the system administrator, but changes to it often required a reboot of the entire system and thus changes were not made frequently.

To ensure the same degree of multiprogramming, we need to have enough partitions to accommodate for the various processes. However, since some processes require a lot of memory, and others less, we might also want to make our partitions different sizes to accommodate different needs.

Now that we have the partitions created, how do we assign a job to a partition? Since there might be more processes than there are available memory, we need to have some queue where the jobs can wait for the partition.

We can either have separate queues for each partition (like a grocery store line), or have a single unified queue (like a line at a bank).
![](Pasted%20image%2020230329003904.png)

## Relocation and Protection
Notice that in our fixed partition solution, we are physically indexing the memory. However, as a consequence, two main issues arise.

Let's imagine a program that writes some data in memory. To actually write, the program generates a memory address and dereferences it (using *loads and stores*) to access memory. This would be fine if the memory address generated was within the partition, but what if the address lies in some other partition? Since the addresses physically index memory, the program is able to read and/or write to a location that belongs to another process. Hence, we now need a way to ensure that the partition is safe from being (maliciously or unintentionally) accessed by other processes. This is the **protection problem**

Furthermore, since partition 2 and 3 are the same sizes, our program may be placed in either when being loaded into memory. However, most memory addresses that a program accesses are generated at compile time. This is fine for an instruction like `branch` (*i-type*) which increments or decrements the program counter by an offset, but for an instruction like `jump` (*j-type*) which assigns to the program counter an absolute address generated at compile time, we may now end up jumping to the outside of our partition. Hence, since our programs can move between different executions, we need a way to ensure that the program does not leave its partition. This is the **relocation problem** (although in its weakest form).

### Solving the relocation & protection problem
So, what can we do to ensure that when the process is running, it does not leave the partition? Since, we know where each partition starts and ends, we can compare the program's memory addresses to these bounds. But, when can we check these bounds? 

We could attempt to have the compiler check the address it generates, but malicious users can simply bypass the compiler and manually code in assembly.

If we were to have the operating system check these bounds, each access to memory would require a system call and a context switch to the kernel. About 40% of assembly instructions are *loads and stores*, and another 20% are *branches and jumps*. Hence, if we were to have the OS check these bounds, 60% of all instructions (actually 160% because all instructions live in memory) would require a context switch to the operating system, making this too expensive and not viable.

Lastly, we can try to build in hardware support. While giving control to the operating system is too slow, the hardware already has control of which instructions to execute. Since we want to minimize the costs of bound checking, we need a fast hardware that can store and load data. Thus, we need to add two registers (*limit and base* or *extents*) to keep track of these bounds.

When memory is accessed, the address is compared to these `limit` and `base` registers and a signal is generated if the address is outside the bounds, which allows the operating system to take control and throw an exception.

Furthermore, this approach allows us to easily solve the relocation problem as well. The culprit behind the relocation problem was the use of an absolute address. To mitigate this, we could either (1) use only relative addresses or (2) rebase (rewrite) memory addresses when they are loaded into memory.

For example, given a logical address, `0x1204` and the limit registers (`limit: 0x2000` and `base: 0x9000`), we can calculate a valid address by simply adding the logical address to the address held by the base register: `0x1204 + 0x9000 = 0xa204`. This, simply, converts the physical address to a relative address.

Although this solution works, it is not an ideal one as it slows down the instruction execution time of the CPU, making the overall CPU speed slower. And with two more registers, it increases the costs of each context switch as well. Furthermore, since the sizes of the partitions are not dynamic, we may not be able to run a process that is too big for all the partitions (even if it fits in the total memory). Thus, this is not the solution we use today.

## Swapping
Recall that the presence of the *memory scheduler* allowed us to kick processes out or '*evict*' them from memory (into the disk). Since, the Von Neumann architecture does not allow us to run programs from the disk, evicted processes are effectively blocked (until they are loaded into memory again later).

When we do load in the program again, it may be placed in a different address than before. Hence, the relocation problem occurs again. Luckily, *rebasing* and *scaling by a base register* still works to solve this issue. However, writing *position independent code* (using only relative addresses) does not as even if we change all the instruction addresses, data addresses, like pointers, would still be wrong after the adjustment.

## Room to Grow
In fixed partitions, we assumed that a program had a fixed need (statically sized memory). However, from our programming experience, we know that this is not the case. In fact, many programs use a function like `malloc()` or recursion to request memory dynamically. Hence, we need to account for extra space for dynamic allocation. Note that a similar effect to dynamic allocation can be gained with just swapping and fixed partitions, by statically over-allocating space.