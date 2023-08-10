# Processes and Threads
We will begin our study of the operating system by examining how it plays a role in managing the CPU time. For our purposes, we will consider an operating system that runs on a computer with a modern single-core processor (that is capable of executing instructions at multi-gigahertz sppeds).

## Multiprogramming / Multitasking
> *Running multiple processes **simultaneously***

Unlike old mainframes, it is not unreasonable to expect a modern computer to be able to run multiple programs at the same time. For instance, we might use our computer to listen to music while at the same time reading through a PDF. But, we've discussed numerous times that a processor can only run one instruction at a time; so how can our CPU run two (or more) programs simultaneously?

### Pseudo-parallelism
Consider an old animated film which is built out of hundreds of still frame images that have been sewed together. As each image is flipped in rapid motion, our eyes blend the images to provide an illusion of motion. Computers, similarly, utilize the same principle (called **pseudo-parallelism**) to provide an illusion of concurrency. The CPU switches between running instructions from multiple programs at the millisecond level which for our human eyes looks like two (or more) programs running simultaneously.

In this chapter, we will look at how the operating system decides what program to switch to (a process known as **scheduling**) and how it actually switches to it (context switches). A well optimized scheduling scheme is critical for the operating system as a fault one could incur drastic performance penalties.

Consider running four programs (A,B,C,D) on a single-core processor. If we were to give each program equal amount of time to use the CPU, then each process would only be able to run instructions 1/4 of the total time; the other 3/4 of the time, the program would sit idle as it waits for its turn to control the CPU. Consequently, from the program's point of view, it would take four times as longer to run[^slow-hardware].

However, with proper scheduling, we can dramatically decrease this performance penalty. Much like juggling (where the object spends most of the time in the air/not in our hands), many programs spend most of their times idling (for instance, they may be waiting for user-input). Utilizing this fact, we can interweave instructions of another program during this idle time and achieve parallelism with minimal performance drawbacks.

Note that pseudo-parallelism and scheduling is used even if our processor had multiple codes (which could run multiple programs in parallel). A modern computer system is capable of running hundreds of programs concurrently (certainly more processes than the number of CPUs we have), and by the pigeonhole principle, simply relying on parallel CPUs will never suffice.

[^slow-hardware]: Since the details of pseudo-parallelism is abstracted from user-level programs, it would appear to the program that our hardware was really slow.

![](Basics%20of%20Scheduling.png)

## Process
> A running program and its associated data

As an aside, let us take a moment to consider the difference between a *program* and a *process*. Simply put, a process is an instance of a program. As such, we can instantiate multiple instances of a program, with each instance (process) having their own data. For example, we can open two tabs in chrome that load different webpages because each tab has its own set of data.

### Life Cycle of a Process
![](Process%20Lifecycle.png)
### Primitive Batch System
**(1)** When a process is created (by other processes such as the shell or by `exec()`), it is first put in the **ready** state/queue. At this stage, the process has everything it needs to run (it is **loaded**) except the actual CPU time.

**(2)** As the process is waiting for the CPU time, at some point, the scheduler comes by and selects one of the ready processes to begin running (**scheduling**). Note that the scheduler only considered processes which are ready and that the actual details of scheduling are abstracted to the process.

**(6)** Once the process has finished running instructions, it **exits**. The `exit()` system call then sends us into Kernel mode, in which we can run the scheduler again. In C, a wrapper function calls `main()` which uses the return value from it to call `exit()`.

```mermaid
graph LR; 
  A[Created]
  B[Ready] 
  C[Running]
  D[Exit]
  A --> B
  B --> C
  C --> D
```
This primitive system (**batch system**) has no parallelism (or pseudo-parallelism) and although it works, it is not ideal. This system cannot utilize the previously mentioned idle time of a process.

### Blocked state allows for pseudo-parallelism
**(4)** Hence, we add the **blocked (waiting)** state. While a process is running, sometimes it may need to wait for an event (such as I/O). During this time, the process is idle while it waits for some data. Since a process need to call the system call `read()` to request data from other parts of the system, we enter kernel mode (as a consequence of the system call) where we can once again run the scheduler.

In fact, `read()` is a **blocking system call,** in which it prevents further instructions of that process from running. Note that a blocked process is different from a ready process. Since the scheduler only considers ready processes, even if we had the CPU time, the blocked processes will not run.

**(3)(5)** Once the event occurs (e.g., we get the data), we need to return to that blocked process. To do so, we need to enter kernel mode to run the scheduler again. However, this time, the process waiting for the event cannot make the system call (since the process is blocked and not running). Instead, since the data is coming from hardware, we experience a *hardware interrupt* which puts us back in kernel mode (allowing us to run the scheduler once again). This puts the current process back into the ready state as well as allow the blocked process to unblock. Note that we could put the blocked process into either ready or running state; however, modern OS puts the blocked process back in the ready state and lets the scheduling algorithm decide the next instructions to run.

The presence of a blocked state establishes a notion of pseudo-parallelism, allowing us to run multiple processes at once.

### Greedy processes require interrupts 
Up to this point, the scheduler runs either when a process exits (invoking the `exit()` system call), or due to a hardware interrupt. This means that the scheduler is only taking an active role, however, and may lead to issues.

Now, consider a simple program with an infinite loop which *does nothing*. That is, this program has a single instruction which tells the CPU to jump to the jump instruction. Notice that with our current model, a process only leaves the running state (i) when it exits, or (ii) when it is interrupted. Since this program has an infinite loop, it will never exit freely. And hardware interrupts are rare, especially as we go on. Since our process will be the only process running, no other process runs to ask for an event. Thus, the chance of hardware interrupts decreases and decreases.

Since the OS is responsible for the management of the resources, we would expect our OS to deal with a greedy process that '***starves out***' other processes. This is especially a crucial job for a modern OS because this single process is breaking the notion of pseudo-parallism.

Once again, we need a way to run the scheduler to *pause* the running process and allow other processes to run. Note that we don't kill the process (even if the program is *useless*) as that is not the purpose of the OS. We want to let users run programs regardless of what the process actually accomplishes. Furthermore, it is provably impossible to build a software to analyze programs running an infinite loop (**halting problem**).

Back to invoking the scheduler; To run the scheduler, we need to be in kernel mode. As such, we need some kind of interrupt. There are two main ways to generate events for an interrupt
1. First, we can ask the running process to generate the event. That is, we ask the running process to time-periodically call the `yield()` system call. A system that relies on processes to generate events is called a **cooperative multitasking system** and was common decades ago (e.g., Windows 3.1).
2. However, it seems unwise to rely on the process to voluntarily yield their time. We might have naughty programs which do not yield, leading to the same problem as our example program. In this case, since we cannot generate an event from the software (as the naughty program does not generate it), we must rely upon a hardware preemption timer. A timer built into the hardware delivers an interrupt periodically by setting a maximum time a process is allowed to run before a context switch is needed. If a context switch occurs before the timer, the signal is ignored. If there is no context switch since the last signal, however, a hardware interrupt allows the OS to run the scheduler. This type of system is called a **preemptive multitasking system**.
**(7)** Lastly, processes can be killed by other processes (e.g., `kill`) in what is called an **abnormal termination**.

This process model gives us a view at what is going on  inside the system. Some of the process run programs that carry out commands from a user; other processes are part of the system and handle tasks such as carrying out requests for file services or managing the details of running a disk or tape drive. 

We might say that the processes are structured into two *layers*. The lowest layer of the process-structured operating system handles interrupts and scheduling. In fact, the details of starting and stopping processes are abstracted away in the *scheduler*. The rest of the operating system is structured in a process form; these sequential processes lie above the scheduler. ![](two-layer-process.png)
## Process Table
To implement the process model, the operating system must maintain some sort of data structure which contain some information about each process. This data structure is called a **process table** (in Linux, it is actually a linked list) and in it each process has a process table entry (also called **process control blocks**). 

Most importantly, we need to keep track of the process's *state* (from the process life cycle above). We would also record the priority (to use in scheduling), and even some statistics about the process. Furthermore, it would be beneficial to keep track of what *file*s the process is using (important because in UNIX, *everything is a file*) and related security information. Lastly, to manage the memory of the system, the OS needs some way to keep tabs on the address space of our process.

Note that how the process table entry is exactly laid out is an implementation detail in will vary among different designs. For instance the process table entry might keep track of the registers, program counter, and CPU status work; these details might also be relegated to storage in the program stack. However, here is what we might find in a typical process table entry:
| Process management                                                                                                                                                                                                                       | Memory Management                                                  | File management                                                                                 |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------- |
| Registers<br /> Program counter<br /> CPU status word<br /> Stack pointer<br /> Process state<br /> Priority/scheduling parameters<br /> Process ID<br /> Parent process ID<br /> Signals<br /> Process start time<br /> Total CPU usage | Pointers to text, data, stack<br />*or*<br />Pointer to page table | Root directory<br /> Working (current) directory<br /> File descriptors<br /> User ID, Group ID |
