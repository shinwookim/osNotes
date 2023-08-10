# Hardware Can Impact Software
# Hardware can impact software
## CPU Architecture
There are many micro-architectural concerns which can influence the implantation and discussion of the operating system. The simplest computer architectures follow a "Read->Execute->Retire" pattern for running code, but there are many types of architectures that are far more complex (each with its own implications that may impact how the operating system is built/runs).

For example, an architecture that executes instructions out-of-order may already have executed instructions that follows a division-by-zero error that may need to be retroactively retired; and depending on the architecture, the exceptions raised from this errors may differ too.

However, since our primary focus for this course is not hardware, we will largely ignore these concerns.

Yet, sometimes hardware design has such profound implications that it can changes how the software works. Consider the Hard Disk Drive (HDD):

An HDD stores bits as North/South magnetic poles on one side on a platter which rotates at 7200 RPM (or similar speeds); and data is laid out in concentric circles (called cylinder and tracks). 

As a consequence, each track carries differing amounts of capacity it can store as the the circumference for each track is geometrically different. Furthermore, since the inner and outer tracks are rotating at the same angular velocity (and this different linear velocity) the read/write speed is faster on the outer track than it is on the inner track. 

For this operating system which might be interested in writing to disk, where we physically place the data can be a big concern especially if we are worried about performance. Later, we will see how the operating system can utilize this when we discuss file systems.

## I/O via Interrupt
Another way hardware impacts the operating system is in how it handles I/O operations. Like software originated interrupts (such as system calls), the hardware can also generate interrupts which may trigger the operating system. 

One example of this is when a device wants to signal some state to the operating system. Because a hard disk is so slow, it may not be practical to run the `read()` instruction an wait until the disk returns with the data. During the time we are waiting for the disk to fetch data, we are wasting the potential of our CPU from running more important instructions.

Instead, one method might be to signal the hard drive to fetch data and continue running other instructions until the disk is ready. Once the disk is ready to transmit data, it (the disk or the bus) can generate an interrupt which tells operating system to receive the incoming data.

Although there are many hardware-specific details which we will often gloss over, it is worth considering how interactions with different hardware designs can influence how software runs.