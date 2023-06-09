# Interrupts
We've seen how, when a user program makes a system call, the operating system stops the hardware from running user code and switches to the OS code to complete the requested task. Thus, our code is *interrupted* while the operating system does the requested task, and when it's done, the hardware is returned to our program. This is the idea of an **interrupt**.

When an system receives an interrupt, it must stop the current program, handle the interrupt, then come back (or crash the program), much like a system call. In fact, a system call is a type of interrupt called **trap**; however, an interrupt is much larger than a system call. Grouping by the source, there are:
1. Software Originated Interrupts (I.e., Trap, Exceptions) which come about because a user program has requested it.
2. Hardware Interrupts which comes about because a hardware device has requested it.

To handle these interrupts (regardless of the origin), the processor first sets the program counter to the memory address of the operating system code which can handle the interrupt. The address that is loaded into the program counter comes from a structure on the chip called the **interrupt vector,** which is indexed by the particular types of interrupt. At each index is an address which points to the OS code which can handle the interrupt.

For instance, system calls are all mapped to the same entry in the interrupt vector (usually index 0, but sometimes index 128 in x86). Then, to handle the specific system call, the operating system has a second table (Linux calls it a System Call Table) which is indexed by the ordinal stored in a register like `$v0`.

This means that there is a 2-step process of dispatching a particular system call:
1. Read the entry in the interrupt vector to get the address of the operating system code that handles system calls.
   1. Load that address into the program counter
   2. Escalate privilege to allow the processor to execute kernel mode instructions
2. Read the entry in the System Call Table to get the address of the operating system which handles that particular system call.
   1. The system call table is indexed by an ordinal that is stored in a register like `$v0`.
   2. Run the code that handles the particular system call. Once the code completes, drop the privilege level (to user mode).
   3. Return to where the system call originated.