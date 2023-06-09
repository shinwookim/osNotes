# Abstracting Details
> An operating system is a piece of software that manages resources and **abstracts details**.

The second part of this definition states that an operating system abstracts details. We saw before that the abstractions provided by the operating systems help us write programs without having to know specific details about the hardware. For example, we can write a simple Hello World program by calling the `printf()` function without having to worry about the CPU speed, RAM capacity, or screen size.

Often, the operating system provides a standard interface (much like a standard library) that allow us to interact with and manipulate the hardware without having to worry about the specifics.

## Example: Memory
Let's look at another example. In the early days of computing, resources such as *memory capacity* and *CPU time* were so limited that we could only run one program at a time. For instance, to run four programs (A-D), we would need to run program A to completion, then program B, then C, and lastly D. During this time, there was no need for resource management. Since only one process was running at a given time, we simply gave that process **exclusive access** to all of our system resources until the program terminated.

However, as computers became much more powerful, users now wanted to run multiple programs simultaneously. This meant that the systems resources would need to be shared among the running programs Memory, for instance, had to be divided so that programs could each load instructions and data. Similarly, CPU time had to be divided so that a portion of CPU time went to each process.

Yet, our computers were not powerful enough to have unbounded (or close to unbounded) resources. For example, while we had much more in memory capacity than before, there was not enough to give every program everything it requests.[^trivial] Hence, computer scientists had to devise a efficient way to *share* and manage resources among various processes.

Furthermore, with multiple programs running, we now ran the issue of a program accessing resources held by other programs. For example, how would we ensure that a music player doesn't access the region of memory that is being used by a password manager? 

We could force each software developer to implement their own memory protection scheme for each program. But this is cumbersome and difficult. Instead, we can abstract a notion of **exclusive access** to the resources. With an interface where programs seemingly has exclusive access to the resource, accessing resources of another is now stopped. Furthermore, it frees up the programmer to focus on software design principles rather than worrying about a presence of other processes or sharing resources.


[^trivial] If all of the programs were able to get everything it requested, resource management would become trivial as we could allocated all the resources to all the process.