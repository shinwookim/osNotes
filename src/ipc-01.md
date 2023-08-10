# Inter-Process Communication
# Inter-Process Communication
Previously, we have said that the key difference in choosing between threads and processes were whether the tasks we wish to parallelize were cooperative or competitive in their nature.  

In that discussion, we found threads to be particularly useful for cooperative tasks because they could easily communicate with one another by sharing a common address space (which all threads could all read and write to).

Yet, this desire for communicating with one another is not unique to threads. In many cases, a process may also need to communicate with other processes. For example, in a shell pipeline, the output of the first process may need to be passed to the second process, and so on.

Here, we will study how processes can communicate with one another by answering the following questions:
1. How exactly can we pass information between multiple processes?
2. How do we make sure that a process does not get in the way of another?
3. How can we properly sequence processes (in a specified order)?

The answer to the first question is rather trivial. Much like in threads, we can simply share a portion of memory among multiple processes, often with the `mmap()` system call. As for the last two questions, they require a much more elegant solution.

It's important to note that much of our discussion of inter-process communication also applies to threads. Indeed, while the first question above applies only to processes, the other two questions apply equally to threads as they do to processes.

## Race condition
Suppose we wish to add two integers, 20 and 9, to the tail of a rudimentary queue (backed by an array). To do this in a performant manner, we choose to parallelize the two enqueue operations by instantiating two processes *A* and *B*. Process *A* will add 20 to the end of the array (at the tail index) and then increment the index. Similarly, process *B* will add 9 to the array (again, at the tail index) and increment the tail index. 
``` C
/* Shared Data */
int tail = 4;
A[] = {1, 8, 5 ,6, ' ', ' ', ...};

/* Process A */
A[tail] = 20;
// What if we process switch here?
tail++;

/* Process B */
A[tail] = 9;
// Or we process switch here?
tail++;
```
Note that by the nature of parallelizing tasks, we give up control of the specific order between 20 and 9. If we wished to specify an order, we would need to run the processes sequentially (or a single process executing multiple enqueue operations sequentially). Thus, upon successful completion of both processes, we expect the queue to be either `{1, 8, 5, 6, 20, 9}` or `{1, 8, 5, 6, 9, 20}`.

Indeed, in run after run, the output is just that. However, since our operating system's scheduling is *chaotic*, and preemption can occur at any stage in the process, we may find ourselves in the situation where we *process switch* between when we add to the array and increment the tail.

For instance, in such a case, after we add the 20 to the array (assuming process *A* runs first), our underlying array will hold `{1, 8, 5, 6, 20,' ', ...}` and the index `tail` would still be 4. Then, just as we are about to run the instruction to increment the `tail`, we may be preempted. If the scheduler then chooses to run process *B* at this point, *B* would write 9 to `A[tail]` overwriting the 20 added by process *A*. Hence our queue would now be `{1, 8, 5, 6, 9,' ', ...}`. Then, assuming we aren't preempted, process *B* would increment the `tail` such that it now holds 5 and then terminate. As process *B* exits, the scheduler is invoked again, and we choose to continue running process *A* from where we left off. Since process *A* was interrupted right before incrementing `tail` index, it would finally increment the tail so that it now holds 6, then terminate.

Looking at our queue, we are left with `{1, 8, 5, 6, 9,' ', ...}` which clearly does not match the expected output. Furthermore, the `tail` index now holds 6 when there are only 5 elements in the underling array, which could cause critical issues in future operations (such as in the case of a dequeue operation).

Situations like these in which multiple processes read and write to some shared data and the final result depends on who runs precisely when is called a **race condition**. A race condition occurs because the processor is not fast enough to execute the entire code (insertion and increment) before preemption. 

More specifically, a race condition occurs if we are preempted in between *read*ing some state (such as checking some condition), and *acting* upon it. The action may be to *update* some variable (such as incrementing `tail`), or making a *decision* (such as throwing an exception). Regardless, if we are interrupted between the *read* and the *act*, the state may have changed in between making our *action* incorrect and causing a race condition.

Blocks of code such as these in which a process might be preempted between the steps are called **non-atomic**. Non-atomic code is fine by with self, but when it is an instruction dealing with the access of a shared resource, it may manifests into a race condition, causing critical issues that result in unexpected behavior.
- Note that the length of the code does not equate to a code being atomic or non-atomic. In fact, many single-line codes are still non-atomic. For instance, in the example above, changing the enqueue code to be a single line (`A[tail++] = 20;`) will not fix the issue because when the new code is compiled, it will yield the same multiple machine instructions (which we can be preempted in between). We will typically assume that a single machine instruction to be atomic (although this may not necessarily be true depending on architectures).

Problems with race conditions are hard to spot, as it occurs only when the process is preempted precisely between reading and acting by a process that uses the same shared state. In fact, if the scheduler does not choose another process that uses the same shared state, the issue may lie dormant forever. Yet, it is important to identify this issue, as it may suddenly manifest with installation of new hardware/operating system, or purely by random chance even without any change in the code.

## Critical region
We call a region of code that accesses shared memory (code that contains a potential race condition), the **critical region**. Identifying critical regions is important because if we can ensure that no two process are simultaneously in the critical region, we have effectively solved the problem of race conditions.

Therefore, we need to devise a way to handle when we enter or exit the critical region, with the goal of ensuring that no process can enter the critical region if there is another process in the region already.
``` c 
enter_critical_region()
// Critical region work
// which works on shared memory
leave_critical_region()
```

### Disabling interrupts in the critical region
Our first attempt to create these handlers begin by noticing that the race condition only manifests when we are preempted whilst we are in the critical region. Hence, a naïve but simple approach may be to disable interrupts as we enter the critical region, and re-enable them as we exit.
``` c
enter_critical_region()
{
	disable_interrrupts();
}
leave_critical_region()
{
	enable_interrupts();
}
```

However, there are a few big problems with this approach. First, this allows the process to opt-out from preemption. That is, the operating system cannot enforce its policies nor manage any resources. With interrupts disabled, the processor cannot ever switch to executing the operating system's code. Hence, if a malicious application were to call `enter_critical_region()` but never leave, this would block any other processes from running and the operating system will not have any way to stop this. Hence, this approach gives way too much power to the applications (which they can and likely will abuse).

Furthermore, since without preemption, we are not able to run any other processes, we break the notionmpseudo-parallelism and interweaving processes. That is, no other process is allowed to run regardless of whether they try to access the same cirtical region. Clearly, this approach has many undesirable side affects.

### Mutual exclusion using lock variables
Since we can't prevent interrupts (nor do we wish to), we turn to using a *lock* (often called a **mutex** for mutual exclusion) to block entry into the critical region.
``` c
/*Process A*/
lock(&mutex); // Lock the mutex before entering the CR
A[tail] = 20;
tail++;
unlock(&mutex); // Unlock the mutex when we leave the CR

/*Process B*/
lock(&mutex); // If the mutex is already locked, we block
A[tail] = 9;
tail++;
unlock(&mutex);
```
Now, just before we enter the critical region, we lock this mutex. Then, if we are preempted inside a critical region and another process tries to access it, they will attempt to lock the mutex for themselves. Yet, if they discover that the mutex is already locked, they will block and stop their entry into the critical region.

This type of lock is often implemented in C via the pthread library's `p_thread_mutex_t`:
```c
#include <stdio.h>
#include <pthread.h>

int tail = 0;
int A[20];

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void enqueue(int value)
{
	pthread_mutex_lock(&mutex);
	A[tail] = value;	
	tail++;
	pthread_mutex_unlock(&mutex);
}
```

Although it may seem that we have now solved the problem of race conditions, in fact, this new code still contains the same flaw as our initial example.

We can still be preempted inside the `pthread_mutex_lock()` function call just before we update the mutex. If another process then tries to enter their critical region and calls `pthread_mutex_lock()`, they will check that the mutex is not yet locked and enter their critical region. But, if we resume the initial process, we will return to inside the `pthread_mutex_lock()` function call where having read the mutex to be unlocked, we will once again enter the critical region. Hence, we have returned to square one with multiple processes inside the critical region. That is,  the race condition was simply transported from the `enqueue()` function to inside the `lock()` function.