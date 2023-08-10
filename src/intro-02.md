# Types of Operating System
An operating system exists to manage resource among the multiple programs that are running. However, some systems have more resources (and more programs) than others. Hence, depending on how much resource a system has and what its demands are, there are different operating systems that can manage it.

| Many Resources            | Limited Resources (Smaller Systems) |
| ------------------------- | ----------------------------------- |
| Mainframe OS              | Real-Time OS                        |
| Server & Supercomputer OS | Embedded OS                         |
| Parallel Computer OS      | Smart Card OS                       |
| Personal Computer OS      |                                     |

## Systems with Many Resources
When we  hear the word *computer*, we often think of **personal computers**. These are systems with lots of resources which can be shared among multiple programs simultaneously. Examples of these systems include Desktop PCs, Smartphones, and Tablets.

However, there are computers with even larger amount of resources. One such computer is a **mainframe**. A mainframe is a type of computer that has enough resources to virtualize an entire computer[^VM-OS]. For these systems, with the amount of resources they have, resource management is almost trivial.

[^VM-OS]: In fact, the earliest operating systems were designed to run on mainframes. Known as *Virtual Machines*, they aided in emulating multiple full-computers on the mainframe simultaneously.

## Systems with Limited Resources
Although we will primarily focus on operating systems for computer systems with many resources, there are systems on the other side of the spectrum. For instance, **embedded systems** are built as small and cheap as possible. They maximize the resources that they have, and designed only to have the resources that are absolutely necessary. Luckily, many embedded systems have workload that is known in advanced and very few variable inputs (i.e., humans) if at all. Hence, the job of an operating system becomes almost trivial.

Many operating systems designed for such systems are built to do the bare minimum. For example, a minimal operating system may just be a library that glues and links programs together. In fact, some of these systems may not have a traditional operating system at all. Since the exact workload is known, engineers might choose to run the software code '*bare metal*'.

A **real-time system**, is a system which have a set workload, and a deadline to complete them. Often, a major consideration in these systems is what will happen if a deadline is missed. If missing the deadline is bad, we call it a **soft real-time system**. For instance, a video codec may be considered a soft real-time system. The codec needs to decode video and figure out how to display the pixels within a set time (e.g., 30ms), but if the codec misses the deadline, at worst, the video gets laggy. If however, missing the deadline is catastrophic, the system is called a **hard real-time system**. Examples of hard real-time systems include: autopilot system, nuclear power plant controller, and health care devices.