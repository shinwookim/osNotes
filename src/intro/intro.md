# Introduction to Operating Systems
On a modern computer, we can run software without having to know the specific details of the underlying components. For instance, we can run a program such as Microsoft Word on a variety of computers with varying amounts of RAM; we can also print a document without needing to know the specific make and model of our printers. 

This is all made possible thanks to a piece of software called the Operating System which hides the specific hardware details. In this course, we will examine the abstractions that an Operating System provides to user-level programs. Additionally, we will explore how to efficiently allocate resources between competing processes. Specifically, we will examine how to share and manage the CPU, memory, persistent storage, I/O devices, and communications.

## What is an Operating System?
A computer is a complex and connected system that consists of various hardware devices such as a processor (and sometimes multiple processors), main memory (i.e., RAM), disks (e.g., Hard Drive & Solid State Drives), and other peripherals devices (e.g., a mouse, network interface, or printer).

Because there are so many parts that must constantly interact with one another, effectively managing and using each component to its full potential can be a daunting task. For instance, if we had to consider CPU speed, RAM capacity, and screen resolution when writing a simple Hello World! program, programming would quickly become a nightmare.

Hence, most computers are equipped with a software called an **operating system** which:
1. manages resources in the background and
2. abstracts specific hardware details by providing a standard method of accessing hardware.