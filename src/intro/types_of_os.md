# Types of Operating Systems

An operating system exists because there are multiple processes competing for the same limited resource. However, some systems have more resources (and more demand) than others. Hence, depending on how much resource a system has and what the demands are, there needs to be different operating systems that manage it.

| Many Resources             | Limited Resources (Smaller Systems) |
| -------------------------- | ----------------------------------- |
| Mainframe OS               | Real-Time OS                        |
| Server OS & Supercomputers | Embedded OS                         |
| Parallel Computer OS       | Smart Card OS                       |
| Personal Computer OS       |                                     |

## Systems with Many Resources
When we typically hear computers, we often think of personal computers. These are systems with lots of resources where resources can be shared among multiple programs simultaneously. Examples of these systems include Desktop PCs, Smartphones, and Tablets.

However, there are systems with even larger amount of resources. One example is a *mainframe computer*. A mainframe is a type of computer that has enough resources to virtualize an entire computer[^VM]. For these systems, with the amount of resources they have, resource management is almost trivial.

## Systems with Limited Resources
Although we will primarily focus on operating systems for computer systems with many resources, it is important to recognize that different systems have different needs. An operating system for a system with very few resources are often built to do the bare minimum.

For instance, **embedded systems** are built as small and cheap as possible. They maximize the resources that they have, and only have the resources that are absolutely necessary. In a system like this, the workload is known in advance and there are often no human 'users'. Hence, the job of an operating system becomes almost trivial. In fact, some of these systems do not have an OS and the software code runs on 'bare metal' (although these software can be considered essentially as an operating system). In other cases, the minimal operating system may simply be a library that glues and links programs together. 

A **real-time system**, is a system which has a set workload, and a deadline to complete them. Often, a major consideration in real-time systems is what will happen if a deadline is missed.
- If missing the deadline is bad, we call it a **soft real-time system**. For instance, an example of a soft real-time system may be a video codec. The codec needs to decode video and figure out how to display the pixels within a set time (e.g., 30ms). If the codec misses the deadline, the video gets laggy.
- If missing the deadline is catastrophic, we call it a **hard real-time system**. Examples of hard real-time systems include: Autopilot, Nuclear Power plant control, Health care devices. 



---

[^VM] In fact, virtual machines are one of the oldest kinds of operating systems.