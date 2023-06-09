# System Call vs Function Call
If a system call is just another control transfer with a return, why do we need a specific dedicated instruction (`syscall`) for them? In other words, why can't we just use a jump instruction (`jal`) instead to make the system call?

In MIPS, we learned that a jump instruction like `jal` was a [J-type instruction](https://inst.eecs.berkeley.edu/~cs61c/resources/MIPS_help.html) which used the first 6 bits as the opcode and the remaining 26 bits as an immediate storing the address of the function that was being called[^syscall_type]. However, when we used the `syscall` instruction, we did not need to supply any operands to determine what system call we were making. Instead, we put an integer in register `$v0`. This integer was not an address of any function, but was an enumerated enumerated ordinal which could be looked up on a table.

This is a key difference between function calls and system calls. Even though they make look similar, a function call works by transferring control to a function that is *addressed*, while the system call works by transferring control to a piece of code that is *indexed*.

[^syscall_type] As an aside, in MIPS, the `syscall` instruction is considered to be of there are more R-type instructions available (due to the presence of the `func` field) than there are J or I type instructions.
