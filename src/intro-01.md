# What Is an Operating System?
A modern computer is a complex and connected system that consists of various hardware devices such as a processor[^1], main memory (also known as RAM), disks (namely Hard Drives and Solid State Drives), and other peripherals devices (such as a mouse, network interface, or printer).

Because there are so many of these parts that must constantly interact with one another, effectively managing and using each component to its full potential can be a daunting task. For instance, if we had to consider CPU clock speed, RAM capacity, and screen resolution (to name just a few) to write a simple "`Hello, World!`" program, programming would quickly become a nightmare.

Hence, most computers are equipped with a software called an **operating system** which:
- **Manages resources** of the system and
- **Abstracts** specific hardware **details** by providing a standard interface for accessing hardware.

[^1]: These days, it is common to find a computer that carries multiple processors.

## Managing Resources
> *An operating system is a piece of software that **manages resources** and abstracts details.*

Broadly speaking, a resource is something that is needed by a system to conduct its tasks. Put another way, a resource is something that restricts the computer system from carrying out its tasks. 

For us, we are especially interested in resources that are of a finite quantity[^2]. Let us now examine some of the resources we will be looking at in this course.

[^2]: We will see that managing resources whose quantity is limitless (or close to limitless) is a rather trivial task.

### CPU Time
The core of a computer lies within its CPU. The CPU is responsible for the carrying out the primary purpose of a computer: executing machine instructions.

Yet, due to the nature of physical components, the CPU can only run so many instructions in a given amount of time. For instance, a modern processor running at 3 GHz can run at most three billion machine instructions each second. 

While this may sound like a lot, the number of instructions (out of all executed instructions) that belong to a single program quickly shrinks when we consider how many programs are running simultaneously at any given time.

In fact, among the hundreds of programs running simultaneously[^3], each program is given a set amount of time when it can use the CPU to execute instructions. When a process runs out of this time, the control of the CPU is given to the next program, which begins to execute its instructions.

Thus, a process that wants to execute more instructions requires more **CPU time** allotted to itself. Hence, we can consider CPU time to be a resource that restricts a program’s capacity to do more work.

We will see later how we can optimally manage the CPU time when we talk about **scheduling**.

[^3]:  Try opening your task manager. You will see that there are hundreds of tasks even when you are not doing anything with your computer.

### Main Memory
Another common resource in a computer is memory. Since most computers store instructions in memory before the CPU executes them, how much memory we have determines how many instructions can be loaded into memory (which can then be executed by the CPU). Hence, **memory** is a key resource that is needed by a program to do more work.

We will see how to optimally manage memory usage when we talk about **memory management**.

#### Von Neumann Architecture
As an aside, most computers store both CPU instructions and data in the same memory location (RAM). This is one of the key principle of the **Von Neumann architecture**[^4], which allows the CPU to fetch both data and instructions using the same data path. This simplifies the electrical circuits and makes it easier to implement in hardware (which has led to their popularity)[^5]. 

One interesting side effect of the Von Neumann architecture is the possibility for self-modifying code. Since both code and data live in the same memory, it is possible to write a program which modifies its own code as it runs[^6]. This is possible because code and data are stored simply as `1`s and `0`s in memory and only interpreted as either code or data depending on the context as it is read.

[^4]: In contrast, computer systems that have separate memories for data and code are part of the **Harvard architecture**.
[^5]:  Although the Von Neumann architecture is far more prevalent, the L1 cache on many modern CPUs have separate code and data caches. Hence, it is arguable that these CPUs follow both the Von Neumann and Harvard architectures.
[^6]: Most programmers, However, heavily frown upon writing self-modifying programs as they can cause to severe security issues. In fact, most modern CPUs, compilers, and operating systems have safe guards in place to prevent this.

### I/O Devices
Input/Output device is a class of system resource that include disks and the vast array of peripheral devices (such as a mouse or a printer). Disks allow us to cheaply store code and data that is not currently in use. This frees up the limited memory capacity (which is much more expensive to expand) allowing us to load more meaningful code and data onto them.

However, since all program executables are stored on disks prior to running, disk capacity can also limit our abilities to run programs. We will see how to optimally manage disk space when we talk about **File Systems**.

On the other hand, the limiting factor in peripheral devices is how many programs can access them simultaneously. For instance, if we had two instances of Microsoft Word sending a document to a printer, one of the documents would need to wait until the other document finished printing. Similarly, limited access to a peripheral device can stop the process from executing, restricting our ability to do more work.

### Security
Security is a concept that is heavily intertwined with the resource management. A common form of security is access control, which determines whether an entity such as a user or a process has access to a specific resource. If the entity has the correct permissions or a set of conditions are met, access is granted; else it is not.

Since the operating system is responsible for managing resources, it is also a great place to implement security. Throughout this course, we will see various methods and implementations that allow for a better security of our computers. We will often ask questions such as “Even if you could physically access resource X, should you be allowed to?,” and explore the topics of security and availability.

### Example: Resource Protection and Exclusive Access
Let us now look at an historical example of how an operating system manages a resources.

In the early days of computing, resources such as memory capacity and CPU time were so limited that we could only run one program at a time. For instance, to run four programs (A-D), we would need to run program A to completion, then program B, then C, and lastly D. During these years, there was no need for resource management. Since only one process was running at a given time, that process was allowed to take control of all of the system resource until it terminated. Hence, there was really no need for an operating system.

Overtime, however, computers became much more powerful and we wanted to run a lot more programs at the same time. This meant that the systems resources would need to be shared among the running programs. Memory, for instance, had to be divided so that all the running programs could each load instructions and data. Similarly, CPU time was divided so that a portion of CPU time went to each process.

But, with multiple programs running arose new unexpected problems. Since 
access to physical hardware had to be shared among multiple programs, we now ran the issue of a program accessing resources held by other programs. For example, how would we ensure that a music player doesn't access the region of memory that is being used by a password manager?

One solution was to force each software builder to implement their own memory protection scheme for each program. However, this was cumbersome. More importantly took time away from the developer doing actual software engineering and made programming even extremely.

Thus, computer scientists devised a new way to ensure global memory protection. They created a piece of software, the operating system, which would act as a middle man between software and hardware. Each time a program wanted to access memory, it would make a request to the operating system which would grant or deny it. Hence, by forcing the programs to go through the operating system, computer scientists were able to ensure that no program could access a region of memory that did not belong to that program.

## Abstracting Details
> *An operating system is a piece of software that manages resources and **abstracts details***

For most developers, the abstractions provided by an operating system are often overlooked and taken for granted. For instance, we can write a simple `Hello World` program by simply using the `print()` function without having to worry about the CPU speed, RAM capacity, or any other nitty-gritty details about the underlying hardware. In fact, it is all thanks to these abstractions that a programmer's time is freed up to focus more on software design principles or solving more complex problems.

Soon, we will see that the operating system provides a standard interface, called a **system call**,  that (much like a standard library) allows the programmer to interact with and manipulate the hardware without having to worry about the specifics.

## Operating System as Software
> *An operating system is a **piece of software** that manages resources and abstracts details*

So far, we have established that an operating system manages resources which are used by various software. However, an operating system, itself, is also just another software. Hence, to carry out its tasks, the operating system needs some measurable amount of resources. At the least, the operating system needs to put its code into memory. This, however, means that the operating system uses up some amount of resources that might otherwise be available for other user programs. Hence, an operating system is an (necessary) overhead.