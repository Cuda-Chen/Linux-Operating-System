# Linux Lecture 3.9 Chap. 6

## When will the Content of **current_task** be Set?
* **current_task** is a per-CPU	variable.
* The value of **current_task** will be set at
the following two situations:
	* Variable Initialization
	* Context switch

## Set the Value of **current_task** at Variable Initialization

## Set a New Value of **current_task** at Context Switch
* When making a context switch at **__switch_to** to 
switch CPU to a different process (**next_p**),
**__switch_to** invokdes **this_cpu_write** to store
the **task_struct** address of **next_p** in the per-CPU
**current_task** variable of related CPU.

## Linked List

## Problems with Doubly Linked Lists
* The Linux kernel contains hundred of 
various data structures that are linked 
together through their respective doubly 
linked lists.
	* Drawbacks
		* a waste of programmers' efforts to implement a set of primitive operations
		* a waste of memory to replicate the primitive operations for each different list

## Data Structure **struct list_head**
* Therefore, the Linux kernel defines the **struct list_head** data structure, 
whose only fields **next** and **prev** represent the ***forward and back pointers*** 
of a generic **doubly linked list element**, respectively. 
* It is important to note, however, that the pointers in a **list_head** field store
	* the addresses of other **list_head** fields 
	* rather than 
	* the addresses of the whole data structures in which the **list_head** structure is included

## A Circular Doubly Linked List with Three Elements

## Macro **LIST_HEAD(name)**

## An Empty Doubly Linked List

## Macro **LIST_HEAD_INIT**

## Runqueue in Linux Versions 2.6 ~ 2.6.23

## The Lists of **TASK_RUNNING** Processes – in Linux Versions 2.6 ~ 2.6.23
* Linux 2.6 achieves the scheduler speedup by 
splitting the runqueue in many ***lists of runnable processes***, 
one list per ***process priority***.
* Each **task_struct** descriptor includes a 
**run_list** field of type **list_head**.
* If the process priority is equal to **k** (a value 
ranging between **0** and **139**), the **run_list** field 
links the process descriptor into the list of 
runnable processes having priority **k**.

## ***runqueue*** in a Multiprocessor System
* Furthermore, on a ***multiprocessor system***, 
each CPU has its own ***runqueue***, that is, its own set of lists of processes.

## Trade-off of ***runqueue***
* ***runqueue*** is a classic example of making 
a data structures more complex to improve performance:
	* to make scheduler operations more efficient,
 the runqueue list has been split into 140 different lists!

## The Main Data Structures of a runqueue
* The kernel must preserve a lot of data for ***every*** 
runqueue in the system.
* The main data structures of a runqueue are 
the lists of process descriptors belonging to the runqueue.
* All these lists are implemented by a single 
**prio_array_t** (= **struct prio_array** ) data structure.

## The **prio** and **array** Field of a Process Descriptor
* The **prio** field of the process descriptor stores the ***dynamic priority of the process***.
* The **array** field is a pointer to the **prio_array_t** data structure of its current runqueue.
	* P.S.: Each CPU has its own runqueue.

## Linux 3.9 Scheduler

## Linux Scheduler
* Linux is a multi-tasking kernel.
* Multiple processes exist at a system at the same time.
* The scheduler of a Linux kernel decided which process is executed by a CPU.

## Linux Scheduler Entry Point
* The main entry point into the Linux 
process scheduler is the kernel function 
**schedule()**.

## Overview of the Components of the Scheduling Subsystem

## Scheduling Classes (1)
* Scheduling classes are used to decide which task runs next.
* The kernel supports different scheduling policies
	* completely fair scheduling
	* real-time scheduling
	* scheduling of the *idle task* when there is nothing to do 

## Scheduling Classes (2)
* *Scheduling classes* allow for implementing different *scheduling policies* in a modular way
	* Code from one class does not need to interact with code from other classes.
* When the scheduler is invoked, 
it queries the scheduler classes which process is supposed to run next.

## Scheduling Classes (3)
* Scheduling classes are represented by a special data structure **struct sched_class**. 
* Each operation that can be requested by the 
scheduler is represented by one pointer.
* This allows for creation of the generic scheduler 
without any knowledge about the internal 
working of different scheduler classes.



