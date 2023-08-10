# Page Replacement Algorithms

Normally...reduce tabular data structure...but, assuming we have at a table that works (using chunking)...

Page Table...

Translation....using division.....shifts...or wiring


How do we ensure a page is in the frame?

Page table entry
extra data....
- Dirty bit
- Referenced bit
- Valid bit 
Memory management because address space > physical RAM size.
→ some pages are not in ram in a given moment

sentinel values in page frame number....but not ideal (we cannot use it for storage)....we cannot use that for a legitimate frame value -→ we waste at least one frame

Valid bit stores is the page in frame?
1: page to frame mapping that is actually i ram
2: page is not in ram and cannot be. used

IF a proces does not have any ram → all page table entries have a valid bit of 0.
But we can't run a program without ram.
When the scheduler is run and selects, the program counter is set to the address of the first instruction (the entry point address) <-- set by scheduler

The first thing the CPU does is attempt to translate the entry point address to physical memory address (attempting to *fetch an instruction*). since program counter is built in terms of virtual addesses.
Virtual adddress --> MMU --> Translation
But the page table entry is invalid!
Page fault
> Exception raised by the MMU indicating a requested virtual address is not in memory
Exceptions --> trigger the operating systems
1. Determine faulting virtual address
2. If request page is "bad": segfault on error
What is bad?
bad- address that points to the free space between heap or stack (not part of any allocated region/portions of address space)

OS can determine the regions using stack pointer (and where stack started), the brk

- Because we never put free space pages into ram (page fault)
good - accessing code from code region.

1. If requested page is "good": find an empty frame
	1. If physical memory is full, choose a frame to evict
	2. If evicted frame is dirty, write frame to disk
2. Load page from disk or grow stack heap

What if we don't load the RAM? --> scheduling decision (via memory handler) --> overriding the scheduler but why? let's have scheduler do the scheduling

--> hence load the page

which frame do we select?
- since frame swize = page size, pick an empty frame
Then OS resets PC back to address, and try again.

*Loading*: copy from executable into page frame
(demand paging = start off with no ram, and at page fault, bring in page into RAM) vs pre-paging (Unix model <-- fork())

The dirty bit (modified bit) = set to 1 if the value of the page in the frame have changed (then save it) ==> vuirtual memory becomes swapping
(portion of disk called swap space -- file or parition)

dirty = does the version on disk match version in RAM?

dirty page 1: code
dirty page 2: if you being it in from swap space (it comes in clean).

## Page Replacement Algorithms
Consequence: If we choose bad...

Disk access is slow

afraid of future page fault.

If we kick something out that's used again we will get another page fault

perfect choice... kick something out that's never used again

best choice...pick one that will yield a page fault at the furthest point in the future.
In a given windows of time, if we push them apart, the number of occurrences deceases.
<-- Optimal (Impossible because you don't know the future) on-line setting: we must make decision with what we have

off-line: we know the future

In reality we need an algorithm to approximate opt.

how? approximate using the past

1. FIFO - time of entry into system is the predictor (evict the page that has been there the longest by the time of first entry)
	1. Bad because some of the oldest page is the operating system
	2. evicting the page fault handler is bad! (we can pin them so they cannot be evicted)
	3. Background services (servers) are the longest running processes....this is not a great choice :( age of page is not a good predctor of usefulness
2. Not Recently Used: Evicy the page that is the oldest, preferring pages that are not dirty
| Preference   | Referenced | Dirty |
| ------------ | ---------- | ----- |
| First Choice | 0          | 0     |
|              | 0          | 1     |
|              | 1          | 0     |
| Last Choice  | 1          | 1     |

Use the reference bit to keep track of when the page has been referenced.
In demand paging, the initial value of the R-bit is `1`. Everytime we use this valid page-to-frame mapping, we set to 1. ??? When does the R bit go back to zero? NRU introduces a time-periodic refresh where all R-bits are set back to zero.

By introducing the notion of dirty bit, we can also avoid swap disk-writes.

If there are multiple, we can pick an arbitrary one.

*Issues with NRU:* If the page fault occurs just after the refresh, we may end up a page that is useful.

refresh rate using memory references.

Combining NRU and FIFO does ok in practice...2nd chance.
Start with the oldest page....look at the referenced status....give it as second chance.
Move the oldest page to the back of the queue (update the timestamp+clear the reference bit)....Consider it to be the newest page in the system. <-- best of what we've seen so far.

However, dequeue-enqueue cycle is slow!


Let's make it circular!

Each step clockwise...represents slightly newer page 
Newest page is one step counter clock wise from the tail

Instead of dequeue-enqueue, we can simply adjust the head pointer to head.next (after clearing the reference bit)....clock is exactly same as 2nd chance, but is more preferable.

FOR NRU, one bit of history is a bit shallow....enter LRU

The challenge of LRU: we need a total ordering of LRU.

*Method 1*: Organize the pages in a queue
- LRU head, Not LRU is tail
- How much work on page fault? $O(1)$
- But updating the queue....at every memory access $O(N)$. while the program executes...*seems to expensive*

*Method 2:* Keep a timestamp
- Page fault $O(N)$ comparison.
- Keeping a timestamp only costs $O(1)$ at every memory access.

What is time?
- a number
- Monotonically increasing
- how often should we increase? the resolution?
	- May be at each memory access? (because multiple memory accesses can occur at each instruction)
	- nano-second resolution? how many bits to devote to the timestamp
we need a large enough counter...64 bit counter....expensive (move it to RAM, maybe even to disk!)....on a 32 bit CPU ...this cannot be done realistically .

LRU is hard to implement :( (implausible or impossible!))
Let's approximate lru.

Extension of NRU to more bits/ Approximate of LRU ==> Aging scheme
- 8 bit time stamp
	- MSB: Was the page referenced at tick `0`?
	- Next tick: shift each bit then use MSB to store whether page was used
If we get a page fault at tick 4: pick page 1 because it's not been used (because leading zeros).

Another page fault? How do we break tie?
- Page 3 
Compare the value as an unsigned integer (treat as binary number instead of array of `r` bits) --> evict the smallest one (implies look at leading zeros)

Aging= old observation are less useful...but they stick around for a while (until they fall off the end)

**Working Set:** set of pages used by the $k$ most recent memory refernces (the set of pages we are using right now)
$w(k,t)$ = the size of the working set at time $t$.

Demand paging: a bunch of initial page faults (in attempt to look for the working set).
We automized the discovery of the working ser via *Demand Paging*

Page faults occur because the working set is changing:
1. Changing in size - *eg. growing data structure*
2. Changing in membership - *e.g.,**evict unused page, and load new page*

Overlays are programmers expressing the working set at every point in the program.

At page fault, if we need to evict....select for eviction a page that is not in the working set.
- FIFO fails at this
---
Given a current working set: choose the best page replacement algorithm:
1. Given a working set, if a page fault occurs....kick out something not in the working set.
	1. We can do more...kick out every page not part of the working set.
	2. Prevent future page evictions (be more aggressive)
		1. Future page fault...can simply allocate into an empty frame.
		2. *But...*if we kick everything in the working set...we might kick out pages that are not currently in use...but might be used again later
		3. In previous algs, we might get lucky and find the page already loaded in to the RAM.
Determining the working set is hard (and sometimes impossible!)

### Working Set Page Replacement
When a page fault occurs:
```
// page.age = page.timestamp - current virtual time
for valid page in RAM
	if (page.R == 1)
		set time of last use to current virtual time
		set page.R = 0
	if (page.R == 0 && page.age > tau)
		remove this page
	if (page.R == 0 && page.age <= tau)
		remember the smallest time
```
In the worst case.... still have to kick some one out.
- It's choice of what to evict ... :good!
- Performance...*slow* (Scan all pages for every page fault)
- Also possibly many disk writes (mass eviction of dirty pages)...large amounts of disk io

WSClock...implmentable and pretty good!
- Similar to working set but....we have an early exit from the scan all pages
	- Evict the first page we find that is out of the working set & clean
- Why clock? more like next fit....we pick up at the next serach, where we left off.
	- Along the way, schedule the dirty unref to be written to disk
	- If we do not find the early exit...- Disk work can be done independently...so by the time we exit, the page should've been written.
	- Gives us set of future clean pages.

---

3cs of cache miss
- compulsory misses - something that has never been used
- conflict - hash - conflict
- capacity - not enough space (we had it before)
*temporal locality*

Page 0 is a *compulsory miss*
Page 1, 2, 3 are also compulsory miss

Then next request to page 0...
- fifo is dumb --> conflict miss (bad interpretation)
- capacity miss

3 frames -> 9 faults (not enough ram)
4 fraes -> 10 faults
Even with more frames (more ram), we increased page faults.

*Belady's Anomaly* oocurs for any data structure that is not a stack. yet another reason why fifo is bad!!!

LOCAL VS GLOBAL

Suppose we are using LRU

which page to evict?
LRU picks B2 (earlies time)
this is global...but how?
Page table only talks of your own pages (A's)...now we need to keep track of page table for B...and all other processes.

OR... *inverted page table* (__spareness__ ... we already used chunking)
has a tuple process id and frame number.
...only need one for the entire system


Inverted Table requires search --> O(n) n= numbre of frames...more ram = slower search = slower eviction

hence without Inverted Table....we cannot do global allocation.

LRU can also pick A3
this is local...
if the working set doesn't fit in the allocation of frames...page fault!....lots of potential disk writes.
Trashing
So assume that each page fault is occuring because member ship change (not size)... if the page fault frequency gets too big, the system allocates more frame to the process.

Conversely, if freq < min threshold(b), take away frames b!=0, because compulsory page faults. zero never occurs!

---

One way to look at VM: address space on disk ... virtual memory is a cache for address space