# Managing Resources
> An operating system is a piece of software that **manages resources** and abstracts details.

The first part of this definition states that an operating system manages resources. But, what exactly do we mean by a *resource*?  

Broadly speaking, a **resource** is something of a finite quantity that is needed by a system to conduct its tasks. That is, a resource is something that restricts the computer system's capacity to carry out its tasks.

## CPU Time
One of the most important, yet often overlooked, resource is **CPU time**. Given a fixed unit of time, the CPU can only execute a finite number of instructions. For instance, a 3 GHz processor can run at most 3 billion instructions each second. Since there are multiple processes[^proc] running simultaneously on a modern computer, the CPU time is the amount of time for each process to execute instructions before the next process begins execute its instructions. Hence, if a process wanted to do more work (execute more instructions), it would need more CPU time allocated to itself. That is, the CPU time restricts the process's capacity to execute more instructions.

We will see later how we can optimally manage the CPU time when we talk about [scheduling](../scheduling/intro.md).

[^proc] Processes: instances of a program
## Memory
Another common resource in a computer is **memory**. In most modern computers, CPU instructions are stored in RAM before they are fetched by the CPU. Consequently, how much memory we have determines how many instructions can be loaded into memory (to be fetched by the CPU). Hence, to execute more instructions, we need more memory. That is to say, memory restricts the systems's capacity to execute more instructions.

We will see how to optimally manage memory usage when we talk about [Memory Management](../memory/intro.md).

### Von Neumann Architecture
As an aside, computers which store both CPU instructions and data in the same memory (RAM) is said to be of the [Von Neumann architecture](https://en.wikipedia.org/wiki/Von_Neumann_architecture). In contrast, the [Harvard architecture](https://en.wikipedia.org/wiki/Harvard_architecture) has two separate memories (one for data and another for code). 

Most modern computer systems follow the Von Neumann architecture since it is much simpler to implement—the same data path can be used to fetch both data and instructions which simplifies the circuit. However, if we look closely, the L1 cache inside a CPU is split into a data section and code section. Hence, these computers also follow the Harvard architecture (depending on how deep we go under the veil of abstractions).

One interesting side effect of the Von Neumann architecture is the lack of distinction between code and data. Since both code and data live in the same memory, we can write a program which modifies its own code as it runs (although this is highly discouraged).[^self-mod-code] The code and data are stored simply as 1s and 0s in memory and when it is read, it is interpreted as either code or data depending on the context.

[^self-mod-code] Most modern CPUs as well as compilers and operating systems have safe guards in place to prevent a program from modifying its own code.
## I/O Devices
**Input/Output device** is a class of system resources that include disks and the vast peripheral devices (such as a mouse or a printer). With disks, we can store code and data currently not in use to save on the limited memory capacity (which is too costly to expand). Since all program executables are stored on disks prior to running, disk capacity limits our abilities to run programs. We will see how to optimally manage disk space when we talk about [File Systems](../memory/intro.md).

The limiting factor in peripheral devices is how many processes can access them simultaneously. For instance, if we had two instances of Microsoft Word sending a document to a printer, one of the documents would need to wait until the other document finished printing. Similarly, limited access to a peripheral device can stop the process from executing, restricting our ability to run more programs.

# Security
**Security** is a concept that is heavily intertwined with the resource management. A common form of security is *access control* which determines whether a entity such as a user or a process has access to a specific resource. If the entity has the correct permissions or a set of conditions are met, access is granted; else it is not.

Since the operating system is responsible for managing resources, it is also a great place to implement security. Throughout this course, we will see various methods and implementations that allow for a better security of our computers. We will often ask questions such as “*Even if you could physically access resource X, should you be allowed to?,*” and explore the topics of *security* and *availability*. 


