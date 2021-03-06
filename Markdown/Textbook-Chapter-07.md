# Chapter 7: Memory Management Techniques
<hr>

**Why do we need memory management?**
- Improved resource utilization (only allocate resources based on demand)
- Independent resources for each process
- Make sure other processes don't access protected memory
- Using more resources than capacity physical memory

## Simple Schemes for Memory Management
1. Fence Register to separate user and kernel
 
![](Pasted%20image%2020220324131235.png)
<p style="text-align: center; width: 80%; margin: auto"> <i> Figure 7.2: Fence Register</i></p>

2. Bounds Registers

![](Pasted%20image%2020220324131321.png)
<p style="text-align: center; width: 80%; margin: auto"> <i> Figure 7.3: Bounds Registers</i></p>

**Static Relocation**
- Once an executable is created, memory addresses cannot be changed
- Bounds registers are part of the PCB

**Dynamic Relocation**
- Place an executable into any region of memory that can accommodate the memory needs of the process
- Memory address generated by a program can be changed during the execution of the program

![](Pasted%20image%2020220324131930.png)
<p style="text-align: center; width: 80%; margin: auto"> <i> Figure 7.4: Base and Limit Registers</i></p>

## Fixed Size Partitions
Fixed size partitions divide the memory into fixed-size partitions
- Leads to ***internal fragmentation***

***Internal Fragmentation*** - wasted space inside each allocation

Also leads to another problem: ***external fragmentation***
- Memory chunks are non-consecutive and prevent allocations because of it

## Variable Size Partitions
Variable size partitions divide the memory into variable sizes
- Eliminates internal fragmentation
- Memory manager dynamically builds the table on the fly

![](Pasted%20image%2020220402144827.png)
<p style="text-align: center; width: 80%; margin: auto"> <i> Figure 7.5: State of the Allocation Table After A Series of Memory Requests</i></p>

However, we can still run into problems with external fragmentation. Imagine if P1 gets finished and we free up that space

![](Pasted%20image%2020220402145035.png)
<p style="text-align: center; width: 80%; margin: auto"> <i> Figure 7.6: External Fragmentation After P1's Completion</i></p>

Upon a request for space, the memory manager has to options to choose the allocation
1. Best Fit
The manager looks for the smallest space that can fit the allocation
- Better memory utilization
- Slower time tradeoff

2. First Fit
The manager looks for the first space that can fit the allocation
- Time complexity is much better

### Compaction
![](Pasted%20image%2020220402145536.png)
In the example above, the memory manager might choose to move P3 to start from address 0, creating a contiguous space of 10K
- Very expensive operation
- Virtually impossible in many architectures

## Paged Virtual Memory
Virtual memory gives the user the imaginary view that all the memory is contiguous. Paging is the vehicle for implementing this concept
- Broker divides contiguous view into logical entities called pages
- Physical memory consists of *page frames* (a.k.a. *physical frames*)
	- Both logical frames and physical frames have the same fixed size, *pagesize*

**Analogy**
![](Pasted%20image%2020220402145843.png)
<p style="text-align: center; width: 80%; margin: auto"> <i> Figure 7.7: Picture Frame Analogy</i></p>

Imagine that a teacher wants to collect photos of the students in his class. When a student comes to visit him during office hours, he puts the picture of that student in the frame.

**Important Notes**
- Physical frame is reused for different students
- Does not need a unique for each student since he only sees one at a time

![](Pasted%20image%2020220402150052.png)
<p style="text-align: center; width: 80%; margin: auto"> <i> Figure 7.8: Page Table</i></p>

The user's view is *virtual memory* and the logical pages are *virtual pages*
- CPU generates the virtual addresses
- Page table translates virutal address to physical address

![](Pasted%20image%2020220402150215.png)
<p style="text-align: center; width: 80%; margin: auto"> <i> Figure 7.9: Decoupling User's Contiguous View of Memory</i></p>

### Page Table
If ***page size*** is $N$, then ***page offset*** is found from the lower $log_{2}N$ bits of the virtual address

![](Pasted%20image%2020220402150457.png)
<p style="text-align: center; width: 80%; margin: auto"> <i> 32-bit virtual address with 8K Pages</i></p>

In the above example, we need $2^{13}$ bits to address all 8K, and the remaining bits are used for the VPN

**Converting virtual address to physical address**
![](Pasted%20image%2020220402150613.png)
<p style="text-align: center; width: 80%; margin: auto"> <i> Figure 7.10: Address Translation</i></p>

Since hardware has to look this up every memory access, the page table resides in memory, ***one per process*** to provide memory protection
- Page table takes VPN and gives PFN
- Page offset is the same as page sizes will be the same
- Introduce the **Page Table Base Register (PTBR)** which contains the base address of the page table for the currently running process

### Hardware for Paging
On every memory access, CPU
- Computes address of page table entry (*PTE*) that corresopnds to the VPN using the contents of the PTBR
- PFN fetched from this entry concatenated with page offset gives physical address

## Page Table Set Up
```C
typedef struct control_block_type {
    enum state_type state;
    address PC;
    int reg_file[NUMREGS];
    struct control_block *next_pcb;
    int priority;
    address PTBR;
}
```

# Segmented Virtual Memory
The user views memory not as a single linear address space, but rather as several distinct segments.

Each segment includes
- unique segment number
- size of segment

![](Pasted%20image%2020220402151420.png)
<p style="text-align: center; width: 80%; margin: auto"> <i> Figure 7.11: Segmented Address</i></p>

With segmentation, we can divide a program's memory between its code space, global data, heap space, and stack

### Segmentation Hardware Support
Each entry in the segment table is called a *segment descriptor*
- Gives start address for a segment and the size
- Each process has its own segment table
- Requires a *Segment Table Base Register*

## Paging vs Segmentation
| Attribute |  Paging | Segmentation |
| - | - | - |
|Address spaces per process |One | Several|
|Visibility to user|User is unaware of paging, looks linear|User aware of multiple address spaces|
|Software Engineering|No benefit|Enables modular design, increases maintainability|
|Size of page/segment|Fixed by architecture|Variable chosen by the user for each individual segment|
|Internal Fragmentation|Possible|None|
|External Fragmentation|None|Possible|

## Paged Segmentation
MULTICS introduced the concept of paged segmentation

![](Pasted%20image%2020220402152152.png)
<p style="text-align: center; width: 80%; margin: auto"> <i> Figure 7.12: Paged Segmentation</i></p>

The 36-bit virtual address generated by the CPU consists of two parts: an 18-bit segment number(s) and an 18-bit offset within that segment. Each segment has its own page table