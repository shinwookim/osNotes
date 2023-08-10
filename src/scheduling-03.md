# Interactive Scheduling
Interactive Scheduling
> Impatient users waiting!

In a system in which users interact, we need to add preemption and interleaving to provide the notion of parellelism, as we described before. In this system, the quantitative metrics may be less critical than, *say*, **responsiveness**. However, responsiveness is relative to the user and can't be measured making it a bad metric. Yet, we can still measure response time!

Suppose we have a system in which the user clicks on the screen and a dialogue box pops up. We can measure the response time to be the time taken from the action (click) to the response (dialogue box).

### Round Robin Scheduling
The first preemptive scheduling algorithm we will look at is the **round robin scheduling** scheme. In this scheme, we give each ready process some unit of CPU time. (Note the processes don't run to completion as they did in batch systems. Instead, they are preempted, and other processes are inteweaved.) This unit time is called the **quantum** and is the maximum time we are willing to let programs. That is, a process runs for the *quantum*, before  a preemption occurs.
![](Round%20Robin.png)

However, this does not mean all processes run for the entirety of the quantum. While this is true for CPU bound processes, in reality these processes are rare. How do we know that they are CPU bound? If the processes were I/O bound, they would have blocked much before the quantum. In fact, when we block, the timer would never actually occur, since we would've switched processes already. Hence, only CPU bound processes actually hit the quantum.

Back to our example: What if we are preempted just after a click? We need to wait for another quantum to display the dialogue box. And since, we need to allow other processes to run before resuming ours, the worst case for *waiting time* is $\text{quantum} \times \text{number of jobs}$. Conversely, $\text{response time} \propto \text{quantum} \times \text{number of jobs}$.

Thus, to increase response time, we can either reduce the number of ready jobs, or reduce the quantum. However, the system can't (or shouldn't) reduce the number of jobs (as that would be killing other processes). Hence, there is an incentive to reduce the quantum.

But how short can we make the quantum? Suppose the quantum was equal to the unit time it takes to perform a context switch (say we call this 1 time unit).

| CS  | A   | CS  | B   |
| --- | --- | --- | --- |

But this would mean that out of every 2 time unit, only 1 unit is spent doing useful work. Hence, our CPU is only as 50% efficient as advertised (Our 4GHz CPU is running at an effective speed of 2GHz). To maximize, the time spent doing useful work, we need to reduce the number of context switches. That is, we need to increase the time given to doing process work. Hence, we now have an incentive to make the quantum as long as possible. 


Thus, choosing an effective quantum is a crucial task of balancing these impulses. But how do we know our choice for a quantum is effective? Often in systems, when we need to select a particular value for a parameter (such as page size or quantum), determining that value analytically is not viable (due to trade-offs). Instead, we need to **benchmark** with some program (that mimics standard usage) and evaluate them empirically. Generally (20 - 100 ms) is reasonable for a quantum time.

### Priority Scheduling
In an interactive system, however, some processes are more important than others. For instance, a user may wish to give more resources to a program in the foreground rather than one in the background. Hence, we need a way to express some notion of priority. To allow for us to use this information in scheduling, it is best to inject the notion of priority and quantify it.

Some modern systems, accomplishes this *under the hood*. They give priority to the active process (one that the user is actively using) and reduces the processes in the background (one that is minimized to the task bar). In some cases, a process may depend on another process (parent-child relationship), in which case the OS can assign priority accordingly. However, many systems rely on the users to explicitly provide the priority. In a Unix-based system, a user can select a priority of a process with an integer value between -20 and 20 (with normal processes being assigned priority 16).



- For instance, program in foreground vs. background
- Process that depends on other process
- User can explicitly provide the priority <--
	- Unix based system - -20 to + 20 (integer) - normal process starts at 16
		- `nice` lowers priority (or bumps as super user)
	- Windows - Task Manager
Somehow, we inject the notion of priority and quantify it.



Assume 4 states of priority
- We want the pri4 jobs to run before pri 1 jobs.
	- We can give longer quantum to high priority jobs --> P4 will finish faster. But most of the processes are IO bound and spend the time blocked.
		- Are P4 more likely to be CPU bound compare to P1? No! Hence more CPU time does not actually help for P4
	- What if instead of giving a longer quantum, we give more quantum in a row. This is the same. Once we block, we hope to run that process again at some point. If we give an additional quantum (but not in a row), ie give enough time to unblock. We need to buy some time to allow P4 process to un block.
	- Hence, we can run P4.1, the P4.2, then P4.1, then P4.2
	- But did we put enough space in them? (Did we wait long enough to unblock?) <-- depends on the number of ready process in the given priority.
		- Worry: Not enough process in priorirty to allow for unblock.
		- Worry: Too many process in  P4 means no P3 process can run.

Give all P4 quantum. If they block, when do we give them a second quantum?
Let's run P4.1 P4.2 --> P3.1 P3.2 P3.3 P3.4 --> Then go back to P4 again. (Hopefully by then, we've unblocked.)

P4 -> P3 -> P4 -> P2 -> P3 -> P4 -> P1 -> P2 -> P3 -> P4.

^ No one is starved out.

![](Priority-Scheduling.png)

### Other Scheduling Algorithms
RoundRobuin and FCFS have worked unusally well.. but are there another idea?
- Shortest Process Next: SJF applied to Interactive Systems
	- SJF was provably impossible because we cannot know the job's length.
	- In a interactive system, job length is defined by the user.
	- When we open a program, it takes different times when we open it different times
	- So how do we? change the meaning of a job. Instead a job being the entire execution of a task...instead, we consider the job to be the next burst of computation.
	- Hence, we choose the job that require the shortest burst of computation. --> burst of computation length is more stable than the overall computation length.
	- Programs are not consistent in its behavior. Typically:
		- First phase: input, build data structure (I/O Bound)
		- Next, run algo (CPU Bound)
		- Produce result (print/write to file) - I/O Bound
	- Let's use a histroical weighted average. --> naturally biases towards more recent behavior.
	- Use this average to pick the process that is the shortest.
	- Hence the scheduling algo is biased towards I/O bound process.
	- Still worry: CPU bound process will never run!
		- If we have one long-running job that is relatively IO bound.
		- But I/O bound process are more likely to be user-driven (good for user!)
		- As a system, this scheme is patently unfair.
		- BButwhat if we have a slight bias (not an unfair bias-avoid starvation): 
		- Are we ok with I/O bound processes running with priority?
			- This is generally fine, because we expect I/O bound processes to terminate or block quickly (which frees up time to do more processes).
			- Bit this is not the best scheme to do that
- Guaranteed Scheduling – N processes get 1/N of the CPU Time
	- Not actually an algorithm --> example of Policy: The rules a particular mechanism should follow (i.e., the parameters of an algorithm) -- goal we are trying to achieve
		- How to actuall achieve it is an Mechanism- The way something is done (e.g., an algorithm)
		- To implement, we need to know the number of processes
		- the amount of time each processed recieved, the amount of time they've been around(compute fraction of time they've recieved so far) -- > compare to 1/N
		- Pick a process that has not recieved 1/N
			- Anyone? 
			- When we are guaranteed: the process is new, or they are I/O bound processes (because they block-has been blocking alot).
			- Again, bias towards picking IO bound processes
			- However, we won't starve out a process unlike above. since CPU bound process will fall under the guarantee and rum --> fixed starvation!
			- When are the processes over the guarantee? A process could be temporarily grantee (after they execute)...
				- Or if all the other processes are blocked... CPU bound process will get more CPU time (if they can)
	- Problem: Using this algorithms on a shared computer with multiple users. User A runs 100 process. User B runs 1 process. User A gets more CPU time than B. --> Is this fair?
-  Fair Share– N users get 1/N CPU time
	- In a multi-user system, we might want to divide the system into users.
	- We want to be fair not only amongst processes but also users
- Lottery Scheduling– Give out tickets, pull one at random, winner runs
	- Is a mechanism that implements.. guarantee scheduling. Odds of winning is 1/N (if every process holds one ticket).
	- Is a mechanism that implements Fair Share...give each user X number of tickets\
		- can have collusion among process (parents process gives children process)
	- Is a mechanism that implements ..priority scheduling (if high priority process holds more tickets) - 
	- Assuming uniformly generated random generator (law of large numbers)


Earliest Deadline First (EDF)
- Real-time: How you do homework
Systems that have deadlines (we know deadlines in advance)

Often admissions scheduling

User Threads:
In each process, each process can schedule different threads using different implementation of threads