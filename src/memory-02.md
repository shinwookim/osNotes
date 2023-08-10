# Dynamic Memory Allocation
# Allocation Management

- How do we keep track what parts of memory is free and in-use.
### Bitmaps
- Four contiguous allocations
- Build data structures....bit map (either free or in-use mutually exclusive) --> single bit
-  Bitmap tells us what's in use and what's not
- Where do we put this bitmap...in memory...we need to allocate memory
*Byte-addressable* = each bit to hold 8 bits
==> 1/9 of memory is devoted to storing bitmap ==> bitmap is too big

Attempt 1. Have each bit represent more bytes.
1 bit per Chunk
Make bitmap very tiny in comparison...
But we wuld be overallocating (give the entire chunk)...chunk becomes the minimal unit of allocation. --> leads to **internal fragmentation** (inside the chunk).

Just like quantum....pick a good chunk size (Commonly, 4 kilobytes).
- Big enough, but also small enough
"The magic number"

Attempt 2.
Base 1 unary representation is bad. In other presetnation, length of string < actual intenger ...any other base gives use logarithmic.....let's use base 2
(Logirthimic growth)

Repetition of 1 → Run length encoding to compress size
*compression* (since allocations are contiguous, we expect to see alot of runs)
'Holes between allocations'


In a modern system, bitmap is mostly zero (lots of free space)...

Math: *sparse matrix*

Bitmap has sparseness when the majority of allocation is zero....

Interesting values are rare, common values are really common....

Then what if we just store interesting information?

To search for non-interesting values, we look for interesting values, and then assume that it is in background (if not found).
→ To just hold interesting values (make data structure more dense)...we need a new data structure....Linked List.....we can assume free

Combine sparseness, run-length encoding....
### Linked Lists
But not really,....
We don't really omit node, representing free space. (we could omit and impliclitly infer from end and start of allocated nodes). imperfect sparseness

imperfect Run Length Encoding...allocation C and D are neightbors ... instead of 12 byte run, we have two nodes with 9 byte, and 3 byte.....to allow for allocaiton and deallocation independently.

*analysis*
Where do we store the linked list?
 Worst case... all one byte allocations, one free one allocated one free....
since we coaslesce, no neighbors are both free.

In the worst case, we have chunk for each allocation.

Bitmap is n bits in size....
Linked list is n nodes in size.....and  nodes are bigger..... for example 128 bits in a 32 bit CPU.

Worst in the worst case.....128 times as large. (because there are no runs)...,.


Hence we cannot pre-reserve space for the nodes....we create new nodes when they are needed.....wasted space is proportional to the number of allocations.


## Allocation Strategies
Given a free space, where do we allocate?
1. First fit
– Find the first free block, starting from the beginning,
that can accommodate the request
• Next fit
– Find the first free block, starting where the last search
left off, that can accommodate the request
• Best fit
– Find the free block that is closest in size to the request
Worst fit
– Find the free block with the most left over after fulfilling
the allocation request
• Quick fit
– Keep several lists of free blocks of common sizes,
allocate from the list that nearest matches the request
wasted space...external fragmentation!

## Reclaiming Freed Memory

## `malloc()` and `free()`

sbrk



