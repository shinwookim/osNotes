# Batch Scheduling
## Batch Scheduling
> Non-interactive jobs that can be run “overnight”

We will begin our studies of the various scheduling methods by first focusing on scheduling schemes on a batch system, As discussed before, in a batch system, a processes typically run from creation to termination without any blocks or human interactions.

### Evaluation Metrics
To evaluate the various scheduling algorithms, we will focus on 5 criterias (2 quantiative, 2 computer science metrics, 1 qualitative):
1. **Throughput**: Number of jobs completed per unit time.
2. **Turnaround Time**: Time from job submission to job completion.
	- Note that in a batch system, there is no preemption. Thus, turnaround time will be the execution time (which is equal to the wall clock time) + time spend not running in ready state.
	- The **average turnaround time** is the average of *all* turnaround times for a set of *all* jobs.
3. **Asymptotic Behavior** (*As we get more processes, how much more work does our scheduler do?*)
4. **Implementation Difficulty** (*Complex algorithms with minimal improvements might not be 'worth it*)
5. **Fairness** (*Do comparable processes get comparable service*)
	- Note that the focus is *comparable* processes, and not equal processes.
	- We will always determine fairness using this framework (by comparing comparable processes).

### First come, first served
The first non-preemptive batch scheduling algorithm we will look at is **first come, first served**. As the name suggests, processes are run based on the order the tasks were submitted, with each process running after the previous process terminates.
![](FirstComeFirstServed.png)
#### Analysis
Let us conduct an analysis of the the *first come, first served* scheduling scheme using the metrics we described previously:
1. **Throughput**
$$\frac{\text{number of jobs}}{\text{total time}}=\frac{4}{16}=0.25 \text{ job/unit time}$$
2. **Average Turnaround Time**

$$
\begin{align*} 
A: 4 - 0  = 4\\
B:(4+3) - 0 = 7 \\
C: (4+3+6) - 0 = 13\\
D: (4+3+6+3) - 0 = 16\\
\therefore \sum{\text{turnaround}}=4+7+13+16=40\\
\implies \frac{\sum \text{turnaround}}{\text{number of jobs}}=\frac{40}{4}=10
\end{align*}
$$
3. **Asymptotic Behavior**: Queue operations (enqueue, dequeue) can be implemented as a constant time operation ($O(1)$).
4. **Implementation Difficulty**: Queue implementation is ***relatively easy***
5. **Fairness**: If we consider all jobs to be comparable to each other, the scheme appears to be ***fair***. The tasks that got submitted first, got serviced first. However, if we consider just tasks B and D (which both have comparable runtimes), it *may* be argued that this scheme is slightly unfair. Even though processes B and D had a similar run-time, Task B ran much before task D (especially considering turnaround time).

Without any baseline to compare to, the quantitative measurements we calculated are meaningless. In fact, if we consider all the possible scheduling algorithms (infinite number), there are only $4!$ possible non-preemptive batch schedules (assuming we have 4 jobs as depicted above). In all of these cases, the total run-time is always the same. Hence, without preemption, throughput is always 0.25 and hence it's an uninteresting metric. (Note that we oversimplified the overhead which occurs from scheduling. However, this is fine as in a batch system, overhead only occurs at process termination).

However, this is not the case for average turnaround time. Embedded in turnaround time, is a notion of execution time. In our example, the average execution time was 4 time units, whereas the average turnaround time was 10 time units. Hence, on average each process is waiting for 6 time units. As a process, to minimize this wait time, we want to go first. But with multiple processes in the queue, not all of them can go first. As a comprompise, (from a process's point of view), we want the other processes before us to be as short as possible. That is, as the second process, we hope the first process is short (reduce wait time); and as the third process, we hope the first and second processes are short, and so on.

### Shortest Job First
Thus, leads to the idea of the **shortest job first** algorithm. In this scheme, we schedule the shortest processes to go before the longer processes.
![SJF](SJF.png)
#### Analysis
Again, we'll perform a similar analysis:
1. **Throughput**: As we described before, throughput is the same for all batch scheduling schemes.
$$\frac{\text{number of jobs}}{\text{total time}}=\frac{4}{16}=0.25 \text{ job/unit time}$$
2. **Average Turnaround Time**

$$
\begin{align*} 
B: 3 - 0  = 3\\
D:(3+3) - 0 = 6 \\
A: (3+3+4) - 0 = 10\\
C: (3+3+4+6) - 0 = 16\\
\therefore \sum{\text{turnaround}}=3+6+10+16=35\\
\implies \frac{\sum \text{turnaround}}{\text{number of jobs}}=\frac{35}{4}
\end{align*}
$$
However, for average turnaround time, we see an improvement (since we reduce the average wait time for processes). In fact, it's provable that this scheme produces optimal average turnaround time (regardless of the inputs) — see CS 1501 for proof.

3. **Asymptotic Behavior:** Algorithmically, we can attempt a *total ordering* scheme. This is a simple sorting problem and can be solved with a $O(n)$ insertion and $O(1)$ removal from the data structure. However, by using *partial ordering*, we can actually improve the performance. One such partial-ordering implementation may be the data structure: priority queue (via heap). Recall that  heap operations can be done in $O(\log n)$ time (both insertion and removal). Hence we can achieve logorithmic run-time, asymptotically.
	- However, there is a catch: How can we figure out how long each process will take? The *halting problem* tells us that determining infinite loops is impossible. Hence, determining a run-time without running is al impossible (as it's a harder problem than the halting problem; we must determine not only halting but also the run-time length)
	- We can attempt a static analysis, but this approach won't be fruitful. Since some instructions are longer (such as `syscall`s), simply counting the number of instructions does not provide an accurate description of run-time. Furthermore the limitations of the halting problem will still apply (if there are any infinite loops).
	- Hence, computing run-time is impossible; but what if we look at past behavior? That is, we can measure the time it took for a program to execute and store it somewhere to use in scheduling later on. This is especially useful in a batch system, since run times are ususally pretty stable (due to the lack of user input).
	- But what about programs that have never been run before? Practically, we can have the programmer (or compiler) inject the expected run time into the code. Yet, programmers can lie; in fact, they are incentivized to lie since this will make their process run before others. Hence, to counteract this, we simply kill the process after the given expected run-time passes. Now, programmers are left with an incentive to slightly overestimate (run-time), since they don't want their processes killed.
4. **Implementation Difficulty**: Although implementing a heap is slightly more difficult than FCFS, it's still pretty straightforward. 
5. **Fairness**: Consider all jobs to be comparable. In the SJF scheme, as short jobs arrive, they are pushed to the front of the queue. This means that the longer jobs might never run. In FCFS, with infinite processes, all jobs will eventually run (in infinite time). However, in the SJF scheme, with infinite processes, the process at the end of the queue (long processes) will never run. Hence, no matter what we define as a comparable job, we have **starvation** which is never fair!