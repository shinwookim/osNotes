# Synchronization
# Finding a solution to critical regions
So if these approaches do not work, what does? To find a solution to the problem of race condition, we should first establish the goals of our solution. Whatever our solution (or implementation) may be, it must satisfy these goals:
1. No two process can be inside their critical regions at the same time
2. Solution should not depend on assumptions about CPU speed or number of CPUs
3. No process outside the critical region may block another process
4. No process should have to wait forever to enter its critical region
	- That is, no process should have to wait indefinitely due to the implementation of our solution (not because the work inside the critical region is infinite).

### Strict alternation
Although there are various implementaions, the basic idea of strict alternation relys on a `turn` variable which explicitly keeps track of whose turn it is to enter the critical reigon.
``` c
/* Process A*/
while (TRUE) {
	while (turn != 0)
		; /* loop */
	critical_region ();
	turn = 1;
	noncritical_region ();
}
/* Process B*/
while (TRUE) {
	while (turn != 1)
		; /* loop */
	critical_region ();
	turn = 0;
	noncritical_region ();
}
```
Notice that while it is not each processor's turn, they wait idle. For instance, if process *A* is prempted inside the critical reigon, and process *B* begins to run, it will wait idle for the entire quantum. This type of waiting in which we continuously test a variable is called **busy waiting**. This should generally be avoided since it wastes the CPU time.

Strict alternation does accomplish our first goal. Indeed, with strict alternation no two process can be inside the critical region simultaneously.

However, by explicitly declaring whose turn it is, strict alternation enforces the *ABABAB....* pattern of access to the critical region. Yet, this have some unintended consequences. For instance, if a process (suppose *A*) completes one iteration of work and still have quantum left, it will be blocked from entering the critical region again. That is, even though there are no processes currently inside the critical region, another process is blocked from entering. Hence, our third objective (no process outside the critical region can block another) is violated with strict alternation.

What if instead of explicity identifying which process's turn it was, we identifie whether the resource was in use?
``` c
int locked = 0;
enter_critical_region(){
	while (locked)
		; // <--- What if we are preempted here
	locked = 1;
}

leave_critical_region(){
	locked =0;
}
```
Similar to the example of lock variables, we can be preempted before we update the lock. Thus, another process can sucessfully enter the critical region (as lock has not been updated yet) and when we return to our original process, we will still enter the critical region with another process currently in it. Hence, this approach also does not work as the code that reads and locks the lcok is still *non-atomic*. That is, the code that *handles* the race conditon contains a race condition.

### Hardware-supported 
It appears the adding more code doesn't really help. So what if, instead, we try adding hardware support? We have said that the problem comes from the fact that reading and acting is a non-atomic instruction. So, what if we make it atomic?

Some architectures support just this in an instruction called **Test and Set Lock** (`tsl`). In this case, the hardware guarantees atomicity and the issue is solved. Often, modern architecture will have a more general *atomic exchange* instruction which reads and writes values in a single instruction.

### Peterson's solution
However, relying on hardware-support is totally against our second goal and is not generalizable. For thirty years, it was thought that there was no software solution to this problem. But, in 1981 Gary L. Peterson found an algorithm which achieves *busy waiting* called **Peterson's algorithm**.
``` C
#define FALSE 0
#define TRUE1
#define N 2        // # of processes
int interested[N]; // Set to 1 if process j is interested
int last_request;  // Who requested entry last?

void enter_region(int process){
	intother = 1-process;          // # of the other process
	interested[process] = TRUE;    // show interest
	last_request = process;        // Set it to my turn
	while(interested[other]==TRUE && last_request == process)
		; // Wait while the other process runs
}
void leave_region (intprocess) {
	interested[process] = FALSE; // I’m no longer interested
} 
```

## Producer-Consumer Problem
In 1965, Edsger W. Dijkstra described a classical computing problem called the **producer-consumer problem** (also called **bounded buffer problem**) which demonstrates concurrency and its potential issues of deadlocks.

In this problem, there is a producer who *produces* something and a *consumer* who consumes it. Notice that the natural way for this problem would be to run the producer and consumer sequentially. In one iteration, a producer would produce one instance, and then a consumer would consume it, before re-running the iteration.

But what if there was a disparity in the production and consumption speeds? If production was much faster than consumption, we may want multiple instances of the consumers. Similarly, if consumption was much faster, we may want multiple instances of the producers. Hence, this problem would in fact benefit from introducing parallelism and having a shared **cache**.

In this problem, we will call the shared cache a buffer. The buffer holds whatever the producers produce, and is where the consumers get whatever they consumer. For simplicity, we will be backing this buffer with an array which means our buffer has a fixed maximum size (hence the bounded buffer name). Therefore, we need to check to make sure neither the producer nor consumer over-run or under-run the buffer.
``` c
/* Shared variables */
#define N 10;
int buffer[N];
int i = 0, out = 0, counter = 0;

/* Producer produces X*/
while (1) {
	if (counter == N) sleep();
	buffer[in] = ...; // Production of items into the buffer
	in = (in + 1) % N; // Allows the array to be circular
	counter++;
	if (counter == 1) wakeup(consumer);
}

/* Consumer consumes X*/
while(1) {
	if (counter == 0) sleep();
	... = buffer[out]; // Consumption of items from the buffer
	out = (out + 1) % N; // Allows the array to be circular
	counter--;
	if (counter == N-1) wakeup(producer);
}
```
In the code, we provide conditional statements in the producers to guard over-runs and in the consumers to guard under-runs. In either case, if there is a over/under-run, we invoke the `sleep()` system call which puts the producer/consumer process in the blocked state. (Note the contrast with the `yield()` system call which similarly context switched to another process but kept the current process in the ready state). 

With the `sleep()` system call, the process is now at the scheduler level. But, if we put ourselves to sleep, who wakes us up? Before, when we put ourselves into the blocked state (e.g., `read()` for I/O), a hardware interrupt informed the OS to wake us up. However, here, the `sleep()` system call is invoked without any condition. Hence, we must rely on another process (producers rely on the consumer; consumers rely on the producer) to wake us up using the `wakeup(process)` system call.

### Race conditions
Looking at the code for the producer-consumer problem, we immediately notice a few race conditions. Similar to what we saw in the previous example, the consumer and producer both share access to the `counter` variable; and if we assume that there are multiple instances of producers and consumers, the `in` and `out` variables are also being shared. In any case, accessing a shared variable can cause race conditions to manifest and cause issues similar to the previous example.

But, let us look at another issue. When the consumer runs first, the buffer is initially empty (`counter = 0`), so the consumer makes a `sleep()` system call. Suppose, just before the system call is made, the preemption timer goes off and the producer is scheduled. Now, the producer, after one iteration of producion, will see that the `counter = 1` and will attempt to wake the consumer. However, the consumer has not gone to sleep yet! Thus, now we must figure out how to handle wake-ups to a non-sleeping process.
### Race condition handling with mutexes
One approach would be to do nothing (a no-op). But in this case, we may end up with the producer, asleep, waiting for the consumer to wake them up and the consumer, asleep, waiting for the producer; a **deadlock**.

We know that a deadlock occurs from a race condition. Thus, if we are able to solve the race condition, we have sucessfully prevented a deadlock!

We notice that the race condition occurs because we are preempted after we read the `counter` but before we call `sleep()`. That is, the two non-atomic code (*read and call `sleep()`*) is a critical region that must be handled:
``` c
/* Consumer*/
enter_crit()
if (counter == 0) sleep();
exit_crit()

/* Producer */
enter_crit()
if (counter == N) sleep();
exit_crit()
```
But now, suppose that the consumer runs first and enters the critical region being preempted. The producer would run next, attempt to enter the critical reigon, fail and block. We would then return to the consumer which calls `sleep()` putting both processes to sleep. With both process blocked, this is another issue of deadlock (although this time, it is guaranteed to occur). Thus, we are left with the same problem as before; another deadlock.

It seems that there is nothing we can do. If we move the `sleep()` outside the critical region, we just return to what we hand before. And if we `sleep()` inside the critical region, we end up creating another deadlock!

Hence, we need to make the `sleep()` system call (1) sleep and (2) leave the critical region atomically. Furthermore, we need to be able to (3) re-enter the critical reigon when we are awakened, so that we can execute `exit_crit()` once we resume.

### Condition variables
**Condition variables** provide an easy way to do just this; they give us a linkage between the sleeping process and the process that wakes. As an implementation, we often see it in the form of `pthread_cond_wait(condition, mutex)` (which is part of the pthread library). 

Notice that this function is parameterized by a `mutex` (which is what locks the critical region) and the `condition` which provides the linkage (who we are waiting for).

Therefore, it appears that we have found a deadlock-free solution to the producer-consumer problem:
``` C
#define N 10
int buffer[N];
int counter = 0, in = 0, out = 0, total = 0;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t prod_cond = PTHREAD_COND_INITIALIZER;
pthread_cond_t cons_cond = PTHREAD_COND_INITIALIZER;


void *producer(void *junk) {
	while(1) {
		pthread_mutex_lock(&mutex);
		if( counter == N )
			pthread_cond_wait(&prod_cond, &mutex);
		buffer[in] = total++;
		printf("Produced: %d\n", buffer[in]);
		in = (in + 1) % N;
		counter++;
		if( counter == 1 )
			pthread_cond_signal(&cons_cond);
		pthread_mutex_unlock(&mutex);
	}
}

void *consumer(void *junk) {
while(1) {
	pthread_mutex_lock(&mutex);
	if( counter == 0 )
		pthread_cond_wait(&cons_cond, &mutex);
	printf("Consumed: %d\n",buffer[out]);
	out = (out + 1) % N;
	counter--;
	if( counter == (N-1) )
		pthread_cond_signal(&prod_cond);
	pthread_mutex_unlock(&mutex);
	}
}
```
Note that since it is generally bad practice to constantly lock and unlock (as they incur undesired performance costs), we've made the critical region one iteration of production/consumption.

### Semaphore-based solution
Previously, we did a no-op when a process tried to wake an already-awake process. What if instead, if we wake-up a non-sleeping process, we simply make sure that they do not sleep? That is, what if we counted the number of wake-ups and sleeps to determine whether a process should sleep or not?

This is the basic idea behind a **semaphore**.
``` c++
class Semaphore {
	int value;  // Keeps track # of sleeps and wakeups
				// Positive means more wake-ups
				// Negative means more sleeps
				// Zero means equal # sleeps/wake-ups
	processList p1; // Holds processes that are asleep
	
	void down(){
		value -= 1;
		int (value < 0){ // Didn't miss any wake-ups
			p1.enqueue(currentProcess); // To use in wake-up
			sleep();
		}
	}
	
	void up(){
		value += 1;
		if (value <= 0){
			// Value was negative prior to increment
			// Means there are processes asleep
			P = p1.dequeue() // value == p1.length()
			wakeup(P); 
		}
	} 
}
/* We also need some constructor to initialize
 * the starting value (starting number of wake-ups).
 * Typically, use zero (or some positive) as default
 * Negative value doesn't make sense
 * (since p1.length() == value)
 */
```
Note that different programers will use different nomenclature for `up()` and `down()`. In his original paper discussing semaphore, Dijkstra used `P()` and `V()` (from Dutch); In Unix, the operations are called `wait()` and `post()`.

Let's take a look at how we can use these semaphores in the producer-consumer problem:
``` C
/* Shared Data*/
const int n;
Semaphore empty(n), full(0), mutex(1);
Item buffer[n];

/* Producer */
int in = 0;
Item pitem;
while (1) {
	/* produce an item into pitem */
	empty.down();
	mutex.down();
	buffer[in] = pitem;
	in = (in+1) % n;
	mutex.up();
	full.up()
}
/* Consumer */
int out = 0;
Item citem;
while (1) {
	full.down();
	mutex.down();
	citem = buffer[out];
	out = (out+1) % n;
	mutex.up();
	empty.up();
	/* consume item from citem */
}
```
Notice that we are using several semaphores which serve different purposes. The first semaphore, *mutex*, is used for mutal exclusion. With a default value of 1, when either the producer or the consumer starts executing, the mutex is lowered and we consume the one *missed wake-up* to begin executing. When the other process tries to access the critical region, the *mutex* would be lowered to -1, and that process will be put to sleep. Once, the inital process completes one iteration, it raises the *mutex* which wakes the other process. Essentially, the `mutex.down()` becomes the `enter_critical_region()` and `mutex.up()` becomes the `exit_critical_region`

The other semaphores, *full* and *empty* act to manage access to the shared resource. We start by declaring the inital value of *empty* as the buffer size, and *full* as zero. Next, whenever we produce, we lower the *empty* semaphore and whenever we consumer, we lower the *full* semaphore. Thus, when the *empty* semaphore becomes zero, we've exhausted all the slots on the buffer, and the producer is put to sleep at the next `empty.down()`. Similarly, when the *full* semaphore becomes the buffer size, we have consumed all products produced by a producer and at the next `full.down()`, the consumer is put to sleep. Conversely, when the producer produces, it raises the *full* semaphore signfying a new product and wakes a consumer (if there are any asleep). The consumer, when they consume, raises the *empty* semaphore to signify that there is a new empty slot in the buffer and wakes a producer (if there are any asleep). Thus, the *full* and *raise* protect from buffer over-runs and under-runs by acting as the `counter` variable from before.

As an implementation, this would look something like:
``` C++
#include <semaphore.h>
#define N 10
int buffer[N];
int counter = 0, in = 0, out = 0, total = 0;
sem_t semmutex; // sem_init(&semmutex, 0, 1); in main()
sem_t semfull;  // sem_init(&semfull, 0, 0); in main()
sem_t semempty; // sem_init(&semempty, 0, N); in main()

void *producer(void *junk) {
	while(1) {
		sem_wait(&semempty);
		sem_wait(&semmutex);
		buffer[in] = total++;
		printf("Produced: %d\n", buffer[in]);
		in = (in + 1) % N;
		counter++;
		sem_post(&semmutex);
		sem_post(&semfull);
	}
}
void *consumer(void *junk) {
	while(1) {
		sem_wait(&semfull);
		sem_wait(&semmutex);
		printf("Consumed: %d\n", buffer[out]);
		out = (out + 1) % N;
		counter--;
		sem_post(&semmutex);
		sem_post(&semempty);
	}
}
/* This code uses the wait and post syntax from Unix 
 * semaphores
 */
```
Notice that we raise the *empty* or *full* semaphore before raising the *mutex* (and conversely we lower the *mutex* before lowering the *full* or *empty* semaphores). This small distinction can actually cause massive issues if we were to do so in the other way (raise *mutex* before *full* or *empty*). In that case, a preemption inside the critical region will case a deadlock as the post to *empty* or *full* from the other process (inside the critical region) can never execute as we sleep inside the critical region.

But if the *full* and *empty* semaphores are just the `counter` from before, how do we avoid the race condition associated with reading then writing to them? The answer is we can't! Once again, the solution to solving a race condition contains a race conditions. Whether it is the pthread library or semaphores, the instructions for checking the condition and acting upon it is still non-atomic.

Thus, to use these semaphores we need a way to protect the critical region inside the semaphore. Using another semaphore inside the semaphore (recursively) doesn't work. This just creates more race conditions that need to be created. Similarly, conditional variables such as `pthread_cond_wait` is not a viable option either.

However, it is important to note that, if we have one synchronization method, we can build the others. That is, we can construct semaphores from condition variables, or vice versa.

### Operating system provides a synchronization primitive
One approach may be to incorporate a synchronization primitive as part of the opearting system. Operating systems are **reentrant**, multiple process may make the same system call simultaneously. Since system calls are non-atomic, we may end up with race conditions within the kernel! Therefore, it makes sense for the operating system to contain a method to guard against race conditions (which user processes can hopefully also use via system calls).

Therefore, our operating system must support a primitive synchronization method. For instance, we can implement hardware support, or peterson's solution to create the synchronization primitives. Since we are working with the operating system (which we can trust), we can even implement preemption disabling without worrying about user processes abusing them. Then, as processes (or as libraries), we can use these primitives to construct a more robust synchronization method such as a sempahore, or a mutex.

Note that having the operating system provide us with a primitve also solves the issue of sleeping inside the critical region. Before the primary concern was, if a process starting sleeping inside the critical region, we needed a way for the process to sleep and leave the critical region atomically. Now, since the operating system is the one managing synchronization, it can safely put the process to sleep and then leave the critical region. Note that as long as we can ensure that the entity that sleeps is not the entity that needs to leave the critical region, the problem is fixed. Hence, the threading library, instead of the operating system, may also implement this approach.

Thus, the operating system provides us with a working synchronization primitive (usually a semaphore, but can also be a mutex or a condition variable) from which we can build the other types of synchronization.

#### Usage of semaphores
We said that in the semaphore-based solution to the producer-consumer problem, each semaphore has a different role. In discussing how the *mutex* provided us with a way to ensure mutual exclusion, we used the *missed wake-up interpretation* to show how the *mutex* put the other process (trying to access the critical region simulataneously) to sleep. Yet, if we consider the critical region to be a *shared resource*, we can use the *resource-protection interpretation* (much like for the *full* and *empty* semaphores) to understand how the *mutex* works. Just as we restrict who can access the shared resource at any given time, the *mutex* restricts who can access the critical region at any given time.

### Counting and binary semaphores, mutex
Up until now, the semaphores we have seen had an associated *value*. Semaphores that employ this scheme is called a **counting semaphore**. Conversely, a semaphore that just takes in 1 or 0 is called a **binary semaphore**.

Then, considering that a *mutex* just *locks* and *unlocks*, it seemes logical that the *mutex* should be constructed from a binary semaphore. Yet, it turns out that this is not the case.

Consider the following case with three process: *A*, *B*, and *C* which all try to access the same mutex:
1. At first, process *A* runs. To enter its critical region, process *A* downs the shared mutex before being preempted.
2. Next, process *B* begins to run. Process *B* in an attempt to enter its critical region, downs the shared mutex (which has already been lowered by *A*) and sleeps.
3. Similarly, process *C* runs next, downs the shared mutex and also sleeps.
4. Then, process *A* is once again scheduled and picks up where it was interrupted. It does some work (inside its critical region), and as it exits, it ups the shared mutex.
At this point, we are left with a question: who should we wake up?

It turns out if the mutex was created using a binary semaphore, up would wake up all the downed processes, causing a race condition. Hence, a binary semaphore is not an optimal choice to construct a mutex.

## Other synchronization methods
### Barriers
Binary semaphores, instead, can be used for another purpose: **barriers**.![](barrier.png)
A barrier is often used to **synchronize** multiple processes. In many cases, barriers can ensure that all processes reach the same point, before any process moves onto the next phase.

### Monitors
When we looked at the producer consumer problem, we saw just how careful we had to be with the placement of the semaphores. In fact, simply switching the order of the *mutex* and *empty* semaphores could guarantee us a deadlock.

Instead of taking care of race conditions with low-level synchronization primitives, monitors take a rather unique approach to addressing race conditions. The monitor is a collection of shared global states and methods to modify those states. Hence, it works at a much higher level and in languages such as Java:
``` java
class ProducerConsumer {
	private static final int n;
	Item buffer[] = new item[n];
}
public synchronized Item consumer(){
	while (count == 0) {
		try {
			wait();
		}
		catch (InterruptedException e) {
			System.err.println("interrupted");
		}
	}
	cItm = buffer[out];
	out = (out + 1) % n;
	count-=1;
	if (count == n-1) {
		// wake up the producer
		notify();
	}
	return cItm;
}
```
Notice that this solution to the producer-consumer problem appears to carry same issues which we have dealt with all along. It shares a common state and has code which access that state. Hence, much like the initial example, this code has a race condition.

Yet the single keyword `synchronized` allow us to ignore these race conditions. The keywords signify to the compiler (or in the case of Java, the Java Virtual Machine) that the code block may contain race conditions and that a lock should be inserted. Since, every instance of a Java has an innate lock variable, this allows programmers to easily deal with race conditions just by identifying it.

One key difference in the code (besides the `synchronized` keyword) is that the infinite loop which encompassed the producer and the consumer have been removed. Instead, the thread which calls the `consumer()` or `producer()` would wrap the function calls in an infinite loop. Yet, this is an important distinction to make because synchronized unlocks only when the method returns. Thus, having an infinite loop inside a synchronized function will cause a deadlock.

But this reveals to us the pitfalls of monitors. Because, synchronization is done at the method-level, often it is too broad and is overkill as it ends up hurting the good parallelism (even if it blocks the bad race conditions). Thus, it should be only applied with coarse granularity.

Luckily, the developers of Java also realized this and added support for atomic increment/decrement, which reduces the need for monitors.

### Message passing
Whether it was the *value*s in semaphores, or the *turn* in strict alternation, all the synchronization primitives we have discussed relied on some shared state inside of them. And while this dependence on a shared state is true for many systems (including modern multiprocessing computers with shared memory), if we expand our scope to distributed systems (multiple networked computers that appear to be one), we will discover that this is not the case. In a distributed system, we cannot read a *shared state*. Instead, we need to pass messages to check if we can enter the critical region. 

Hence, synchronization, although possible, in a distributed system, does not resemble the scheme we have so far discussed. Often, they require the use of network **message passing** (peer-to-peer, or hierarchical) and some other methods to overcome the unreliable nature of network communication.


