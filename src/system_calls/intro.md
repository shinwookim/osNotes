# System Calls and Hardware Interrupts

So far, we have established that the operating system is a piece of software that manages resources and abstracts details. However, since an operating system is just another piece of software, to carry out its tasks, the OS itself needs: (1) some measurable amount of resources and (2) some way to assume control of the resources over other programs that are running.

However, this means that the operating system uses up some amount of resources that might otherwise be available for other user programs. Hence, an operating system is a (*necessary*) overhead that lies as an intermediary between the resource requestor (*i.e., programs*) and the resource.

