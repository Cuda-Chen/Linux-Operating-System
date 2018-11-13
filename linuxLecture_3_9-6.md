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

## hlist_head

## hlist_node

## The Process List
* The ***process list*** is a circular 
doubly linked list that links the process descriptors of all existing
thread group leaders.

## The Head of the Process List
* The head of the ***process list*** is the
**init_task task_struct** descriptor; it
is the process descriptor of the so-called
***process 0*** or ***swapper***.
* The **tasks->prev** field of **init_task** 
points to the **tasks** field of the process 
descriptor inserted last in the list.

## The List of *TASK_RUNNING* Process -- in Early Linux Version
* Earlier Linux versions put all runnable processes in 
the same list called ***runqueue***.
* Linux 2.6 implements the ***runqueue*** differently.

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

## Flat Hierarchy of Scheduling Classes 
* An instance of **struct sched_class** must be provided for each scheduling class.
* Scheduling classes are related in a flat hierarchy
	* *Real-time processes* are most important, so they are handled before *completely fair processes*
		* which are, in turn, given preference to the *idle tasks* that are active on a CPU when there is nothing better to do.

## Connecting Different Linux Scheduling Classes
* The **next** element connects the **sched_class** instances of the different scheduling classes in the described order.
* Note that this hierarchy is already set up at compile time

## Priority of Linux Scheduling Classes (1)
* All existing scheduling classes in the 
kernel are in a list which is ordered by the 
priority of the scheduling class.
* The first member in **sched_class** called 
**next** is a pointer to the next scheduling 
class with a lower priority in that list.

## Priority of Linux Scheduling Classes (2)
* The list is used to prioritise processes of different 
types before others.
* In the Linux versions described in this course, the 
complete list looks like the following:
	* **stop_sched_class** -> **rt_sched_class** ->
**fait_sched_class** -> **ield_sched_class** ->
**NULL**

## Priority of Linux Scheduling Classes (3)

## Stop and Idle Scheduling Classes
* Stop and Idle are special scheduling classes.
* Stop is used to schedule the *per-cpu 
stop task* which pre-empts everything and can be pre-empted by nothing.
* Idle is used to schedule the *per-cpu idle task* 
(also called *swapper task*) which is run if no other task is runnable.

## stop_sched_class
* The **stop_sched_class** is to stop cpu, using on SMP system, for
	* load balancing
	* CPU hotplug
* This class have the highest scheduling priority.

## schedule() vs. Scheduling Classes

## *Runqueue in Linux Versions 2.64 ~ 3.9*
## Run Queues in Linux Versions 2.6.24 ~ 3.9
* Each CPU has its own *run queue*, and each active process appears on just one run queue.
* It is not possible to run a process on several CPUs at the same time.

## struct rq
## Fields of struct rq
## runqueues array
## Run Queue-related Macros

## CFS Run Queue
## Red-Black Tree
### Properties of a Red-Black Tree
## CFS Run Queue of a CPU
## struct cfs_rq
### Fields of struct cfs_rq
## struct sched_entity
### Fields of struct sched_entity
## struct rb_node and struct rb_root

