# Scheduling
# Scheduling
> How to choose which of the *ready processes*/*threads* gets to *run* next

## CPU Bound vs. I/O Bound Processes
Nearly all processes alternate bursts of computing with I/O requests; the CPU runs for a while without stopping, then a system call is made to read or write to a file. When the system call completes, the CPU computes again until the next I/O request.

Suppose we measure a process by the amount of wall-clock time elapsed from start to termination (*elapsed real time*). If a process uses most of that time running instructions, we call it **CPU bound**. Conversely, if a process spends most of the time blocked (from blocking system calls), we call it **I/O bound**.
![](CPU-IO-Bound-Scheduling.png)
Now, if we wanted to run two processes, a naïve approach may be to run the processes sequentially. However, this approach will double our run-time. However, since I/O bound processes spend most of their time *blocked*, we might try to interweave the two processes to reduce run-time.

For instance, suppose a single I/O bound process requires 1 unit time to complete. During that time, the actual time spent doing computation might only be 0.3 units. Now, if we were to interweave an identical process, we might be able to complete both processes in 1 unit time if we run the computation for the second process while the orignal process is blocked (this is much faster than running them sequentially which would require 2 unit time). Note that there is overhead, however, as a result of the necessary context switches between the two processes. 

But what about for CPU bound processes? Again, by recognizing that different process use the same amount of time differently, we can attempt to interweave processes to gain performance benefits. However, for CPU bound processes, the time they spend blocked is minimal and thus our performance benefits will also be less (especially once we factor in overhead from context switches and scheduling). 

### Premption and its effect
We've discussed how processes may voluntarily *block* (via system calls like `read()`), but process can also stop non-voluntarily. Previous, we introduced the idea of preemption with the hopes of dealing with greedy processes. With preemption, a hardware timer sends a signal time-periodically, to *preempt* long-running processes. 

As a process, preemption is not desirable (it prevents the system from running the process's code). In fact, it exists to maintain the pseudo-parallelism for the user, but is not necessary from the process's point of view. For the process, a preemption context switch is unnatural (unlike I/O context switches which are incurred when the process calls for it). It's an additional overhead on top of the operating system, which itself is an overhead.

Indeed, in the worst case, we might preempt right before a blocking system call, which means when we return, we block again (almost immediately). This is undesirable as a system too, as we incurred 2 unncessary context switches.

Hence, setting an appropriate length for the preemption timer is important. If the timer length is too long, it won't be effective (as we would have already made a blocking system call before the preemption signal). Yet, if it is too short, we would inccur unnecessary context switches, slowing down our system performance. 

Assuming an appropriate length for the preemption timer is set, I/O bound processes are typically unaffected by the preemption timer (as they have short bursts of computation and many calls to blocking system calls). On the other hand, CPU bound process are frequently preempted. Thus, if we interweave two CPU bound processes, we now have to account for the costs associated with preemptive context switches. Empirically, this may slow the total performance enough to the point that interweaving is slower than the sequentially running the processes (batch system).

### With time, CPU bound processes become I/O bound
Thus, for our goals, we would prefer if the vast majority of the processes on our system are I/O bound (and they are!). Hence, the key question is: *Can we turn a CPU bound process into a I/O bound process?*

The traditional answer to this question was **better hardware**. With faster CPUS, we could improve the computational parts of a CPU bound process. (Note that I/O bound process typically do not reap the benefits of a better CPU because most of their time is spend waiting for I/O and I/O access speed is not proportional to CPU speed). Now, if we can continuously improve the computational performance of a CPU bound program, eventually we will reach a point where the I/O access time overtakes the computational time. And hence our CPU bound program becomes an I/O bound program and interweaving processes become reasonable.

Historically, with [Moore's Law](https://en.wikipedia.org/wiki/Moore's_law), it was reasonable to expect that CPU speeds to increase at an exponential rate. However, in the modern day, this expectation is no longer reasonable, and this approach is less accepted. Yet, still the general principle of faster hardware converting CPU bound processes to I/O bound process stil stand.

Also, as an aside, why can't we easily improve the performance of I/O bound processes? Unlike, CPUs, improvements in I/O devices have been largely stagnant. Although there certainly are some improvements (such as the development of solid state drives), a lot of I/O devices have largely stayed the same (Hard Disk Drive speeds have been the same for almost a decade!). Furthermore, many I/O in these programs represents *human interactions* (which is slow and unpredictable!).

## When to schedule
Now that we've justified the need for a scheduling, let's look at how scheduling actually happens. Note that scheduling occurs in the OS (kernel). So to determine when we schedule, we need to look at when we transition to OS code from process code. 
1. When a **process is created**, we are in the OS. Hence, we can run the schduler here.
2. Conversely when a **process exits**, we are once again put in the OS. Hence, we can run the scheduler here.
3. When we are **blocked** (e.g., for file I/O), we make a system call to get data. With the system call, we are put in the OS, and since we only schedule ready processes, we can schedule here (and pick another process).
4. When we have a hardware interrupt (such as an **I/O interrupt**), we run the OS code. Hence, we can run the scheduler here.
5. As mentioned before, when we have a **clock interrupt**, we schedule (for preemption). 

## Where to schedule...Three-Level Scheduling
Now that we've looked at when we schedule, let's look at where we do the scheduling.
![](three-level-schedule.png)
We've been discussing the scheduler (in the CPU) as choosing a ready process to give CPU time. And although this will be our primary focus, there is another way to *schedule*. Due to the Von Neumann architecture, all processes need to be a RAM resident. Hence, if we deny a process access to RAM, we've implicitly scheduled a process (actually, we can't force a certain process to run, but we can make sure that it can't run).
1. When we open a new program, the job goes into the queue, and the OS allocates RAM. However, if our resources are full (e.g., RAM is full, CPU running), the **admission scheduler** may deny the task from accessing RAM. However, in most modern systems, an admission scheduler is not present. We defer this task to the user (and expect them to make good choices).
2. The **memory scheduler** kicks a process out of RAM. Note if we kick the process out of RAM, but never return it, we've essentially terminated the process. Hence, if we don't want to kill the process, we need some way to preserve the contents inside RAM. The only possible location to store memory contents is on the disk (**swapping**), so the memory scheduler may copy the contents of memory onto the disk and return it at a later point. Note that the von Neumann architecture does not allow for running instructions off of the disk, hence by doing so, we've once again implicitly prevented a process from running. Like the admission scheduler, the presence of the memory scheduler in modern systems is minimal.
Hence, our primary focus will be on the CPU scheduler.
