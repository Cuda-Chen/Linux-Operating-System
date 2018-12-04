# Linux Lecture 3.9 Chap. 8

## How Processes Are Organized
* The ***runqueue*** lists group all processes in a
**TASK_RUNNING** state.
* Processes in a **TASK_STOPPED**, **EXIT_ZOMBIE**,
or **EXIT_DEAD** state are **not** linked in specific lists.
* Processes in a **TASK_INTERRUPTIBLE** or
**TASK_UNINTERRUPTIBLE** state are
subdivided into many classes, each of
which corresponds to a specific ***event***.
* In this case, the process state does not
provide enough information to retrieve the
process quickly, so it is necessary to
introduce additional ***lists of processes***. 
* These are called ***wait queues***.

## Wait Queue
* ***Wait queue*** impelement ***conditional waits*** on ***events***
* Therefore, a wait queue represents
***a set of sleeping processes***, which are woken up by the
kernel when some condition becomes true.
* The condition could be related to
	* an interrupt, such as for a disk operation to terminate
	* process synchronization
	* timing: such as a fixed interval of time to elapse

## Wait Queue Implementation
* ***Wait queues*** are implemented as *doubly
linked lists* whose elements include pointers
to process descriptors. 
* Each wait queue is identified by a ***wait queue head***,
a data structure of type **wait_queue_head_t**

## Wait Queue Synchronization
* Since wait queues are modified 
	* by ***interrupt handlers***
	* by major ***kernel functions***
* the doubly linked lists must be protected from
concurrent accesses, which could induce
unpredictable results.
* Synchronization is achieved by the **lock** ***spin
lock*** in the ***wait queue head***.

## Data Structure of Elements of a Wait Queue
* Elements of a wait queue list are of type **wait_queue_t**. 

## private Field and task_list Field of a Wait Queue Element
* Each element in the wait queue list
represents a ***sleeping process***, which is
waiting for some event to occur; its
descriptor address is stored in the **private** field.
* The **task_list** field contains the pointers
that link this element to the ***list of
processes*** waiting for the same event.

## flags Field
* **flags** has the value
**WQ_FLAG_EXCLUSIVE** or it does not—
other flags are not defined at the moment.
* A set **WQ_FLAG_EXCLUSIVE** flag indicates
that the waiting process would like to be
woken up exclusively.

## Wake up All Sleeping Processes in a Wait Queue ?
* However, it is not always convenient to
wake up all sleeping processes in a wait queue.

## Thundering Herd 
Multiple sleeping processes are awoken
only to race for a resource that can be
accessed by one of them, and the result is
that remaining processes must once more
be put back to sleep.
	* Waste CPU time.

## Sleeping Process Types
* ***exclusive processes***
	* denoted by the value **WQ_FLAG_EXCLUSIVE** (1) in the **flags** field of the corresponding wait queue element
	* are selectively woken up by the kernel.
* ***nonexclusive processes***
	* denoted by the value **~WQ_FLAG_EXCLUSIVE** (0) in **flags**
	* are always woken up by the kernel when the event occurs.

## Examples of Different Sleeping Process Types
* A process waiting for a resource that can be
granted to just one process at a time is a typical
exclusive process.
* Processes waiting for an event like the
termination of a disk operation are nonexclusive.

## func  Field of a Wait Queue Element
* the **func** field of a wait queue element is
used to specify how the processes
sleeping in the wait queue should be
woken up.

## Declare a New Wait Queue Head
* A new wait queue head may be defined by using the
**DECLARE_WAIT_QUEUE_HEAD(name)** macro, which
	* statically declares a new wait queue head variable called **name**
	* initializes its **lock** and **task_list** fields.

## Initialize a Wait Queue Element
* The **init_waitqueue_entry(q,p)** function
initializes a **wait_queue_t** structure *q* as follows:
```c=
static inline void init_waitqueue_entry(wait_queue_t *q, struct task_struct *p)
{
    q->flags = 0;
    q->private = p;
    q->func = default_wake_function;
}
```
	* The nonexclusive process **p** will be awakened by
**default_wake_function( )**, which is a simple
***wrapper*** for the **try_to_wake_up()**.

## Define a New Wait Queue Element
* Alternatively, the **DEFINE_WAIT** macro: 
	* declares a new **wait_queue_t** variable.
	* initializes it with the descriptor of the process currently
executing on the **CPU**. 
	* initializes it with  the address of the
**autoremove_wake_function( )** ***wake-up function***. 

## Functions to Add/Remove Elements from a Wait Queue
* Once an element is defined, it must be inserted into a wait queue.
* The **remove_wait_queue( )** function removes the
corresponding wait queue element of a process from a
wait queue list. 
* The **waitqueue_active( )** function checks whether a
given wait queue list is empty.
* A ***process*** wishing to wait for a specific ***condition***
can invoke any of the functions shown in the
following list. 
	* **sleep_on( ) **
	* **interruptible_sleep_on( ) **
	* **sleep_on_timeout( )**
	* **interruptible_sleep_on_timeout( ) **
	* **wait_event** and **wait_event_interruptible** macros

## Comparisons between the above Functions (1)
* The **sleep_on()**-like functions cannot be
used in the common situation where one
has to test a ***condition*** and atomically put
the process to sleep when the condition is
NOT verified; therefore, because they are a
well-known source of race conditions, their
use is DISCOURAGED.

## Comparisons between the above Functions (2)
* Moreover, in order to insert an exclusive
process into a wait queue, the kernel must
make use of the
	* **prepare_to_wait_exclusive( )** function
	* just invoke **add_wait_queue_exclusive( )** directly.
* Any other helper function inserts the process as nonexclusive.

## Comparisons between the above Functions (3)
* Finally, unless **finish_wait( )** are used,
the kernel must remove the wait queue element from the list after the waiting process has been awakened.

## Wake up Sleeping Processes
* The kernel awakens processes in the wait
queues, putting them in the **TASK_RUNNING**
state, by means of one of the following macros: 
* **wake_up, wake_up_nr**
* **wake_up_all**
* **wake_up_interruptible**
* **wake_up_interruptible_nr**
* **wake_up_interruptible_all**
* **wake_up_interruptible_sync**
* **wake_up_locked**

## wake_up Macro
* the **wake_up** macro is essentially equivalent to the following code fragment:
```c
void wake_up(wait_queue_head_t *q) 
{struct list_head *tmp; 
 wait_queue_t *curr; 
 list_for_each(tmp, &q->task_list)
 {curr = list_entry(tmp, wait_queue_t, task_list);
  if(curr->func(curr,TASK_INTERRUPTIBLE|TASK_UNINTERRUPTIBLE
    , 0, NULL) && curr->flags) 
    break; 
 } 
} 
```

## Explanation of Macro wake_up – (1) 
* The **list_for_each** macro scans all items in the **q->task_list** doubly linked list, that is, all processes in the wait queue.
* For each item, the **list_entry** macro computes the address of the corresponding **wait_queue_t** variable. 
* If a process has been effectively awakened (the function returned 1) 
and if the process is exclusive (**curr->flags** equal to 1), the loop terminates.

## Explanation of Macro wake_up – (2) 
* Because all ***nonexclusive processes*** are
always at the beginning of the doubly
linked list and all ***exclusive processes***
are at the end, the function always
	* wakes the nonexclusive processes
	* then wakes ONE exclusive process, if any exists

## Process Resource Limits
* Each process has an associated set of resource limits,
which specify the amount of system resources it can use.
* These limits keep a user from overwhelming the system (its CPU, disk space, and so on). 

## Locations That Store the Resources Limits of a Process
* The resource limits for the current process are stored in
the **current->signal->rlim** field, that is, in a field
of the process's ***signal descriptor***.

## RLIMIT_AS and RLIMIT_CORE 
* **RLIMIT_AS**
	* The maximum size of process ***address space***, in bytes.
	* The kernel checks this value when the process uses **malloc( )** or a related function to enlarge its address space.
* **RLIMIT_CORE**
	* The maximum ***core dump file size***, in bytes. 
	* The kernel checks this value when a process is aborted, before creating a core file in the current directory of the process.
	* If the limit is 0, the kernel won't create the file.

## RLIMIT_CPU and RLIMIT_DATA
* **RLIMIT_CPU**
	* The maximum CPU time for the process, in seconds.
	* If the process exceeds the limit, the kernel sends it a **SIGXCPU** signal, and then, if the process doesn't terminate, a **SIGKILL** signal.
* **RLIMIT_DATA**
	* The maximum ***heap size***, in bytes. 
	* The kernel checks this value before expanding the heap of the process.

## RLIMIT_FSIZE and RLIMIT_LOCKS
* **RLIMIT_FSIZE**
	* The maximum file size allowed, in bytes.
	* If the process tries to enlarge a file to a size greater than this value, the kernel sends it a **SIGXFSZ** signal.
* **RLIMIT_LOCKS**
	* Maximum number of file locks (currently, not enforced).

## RLIMIT_MEMLOCK and RLIMIT_MSGQUEUE
* **RLIMIT_MEMLOCK**
	* The maximum size of ***nonswappable memory***, in bytes. 
	* The kernel checks this value when the process tries to
lock a page frame in memory using the **mlock( )** or **mlockall( )** system calls
* **RLIMIT_MSGQUEUE**
	* Maximum number of bytes in **POSIX** ***message queues***.

## RLIMIT_NOFILE and RLIMIT_NPROC
* **RLIMIT_NOFILE**
	* The maximum number of open ***file descriptors***.
	* The kernel checks this value when opening a new file or duplicating a file descriptor.
* **RLIMIT_NPROC**
	* The maximum number of processes that the user can own.

## RLIMIT_RSS and RLIMIT_SIGPENDING
* **RLIMIT_RSS**
	* The maximum number of page frames owned by the process (currently, not enforced).
** **RLIMIT_SIGPENDING**
	* The maximum number of pending signals for the process.

## RLIMIT_STACK
	* The maxinum stack size, in bytes.
	* The kernel checks this value before
expanding the User Mode stack of the process.

## struct rlimit
* The **rlim_cur** field is the *current resource limit* for the resource.
* The **rlim_max** field is the ***maximum allowed value*** for the resource limit.

## Increase the rlim_cur of Some Resource
* By using the **getrlimit( )** and **setrlimit( )** system calls,
a user can always increase the **rlim_cur** of some resource up to **rlim_max**. 
* However, only the **superuser** can increase
the **rlim_max** field or set the **rlim_cur**
field to a value greater than the corresponding **rlim_max** field. 

## RLIM_INFINITY
* Most resource limits contain the value
**RLIM_INFINITY (0xffffffff)**, which
means that no user limit is imposed on the corresponding resource. 
* However, the system administrator may choose to impose stronger limits on some resources. 
* **INIT_RLIMITS**

## How the Resource Limits of a User Process Are Set?
* Whenever a user logs into the system, the kernel
creates a process owned by the superuser, which
can invoke **setrlimit( )** to decrease the
**rlim_max** and **rlim_cur** fields for a resource.
* The same process later executes a ***login shell*** and becomes owned by the user.
* Each new process created by the user inherits the
content of the **rlim** array from its parent, and
therefore the user cannot override the limits enforced by the administrator.

## Process Switch
* To control the execution of processes, the kernel must be able to 
	* suspend the execution of the process running on the CPU
	* resume the execution of some other process previously suspended. 
* This activity goes variously by the names
	* ***process switch***
	* *** task switch***
	* ***context switch***

## Hardware Context
* While each process can have its own address space,
all processes have to share the **CPU registers**.
* So before resuming the execution of a process,
the kernel must ensure that each such register is loaded
with the value it had when the process was suspended.
* The set of data that must be loaded into the registers
before the process resumes its execution on the
CPU is called the ***hardware context***.

## Hardware Context Repositories
* The *hardware context* is a subset of the
*process execution context*, which includes
all information needed for the process execution.
* In Linux, a part of the hardware context of
a process is stored in the ***process descriptor***,
while the remaining part is saved in the **Kernel Mode stack**.

## Process Switch and Hardware Context
* Assumptions
	* local variable **prev** refers to the process descriptor of the process being switched out.
	* **next** refers to the one being switched in to replace it.
* A ***process switch*** can be defined as the
activity consisting of saving the hardware
context of **prev** and replacing it with the
hardware context of **next**.

## The Place Where Process Switches Occur
* Process switching occurs **only** in Kernel Mode.
* The contents of all registers used by a
process in User Mode have already been
saved before performing process switching.
	* This includes the contents of the **ss** and **esp**
pair that specifies the User Mode stack pointer address.

## Task State Segment in Linux
* The 80 x 86 architecture includes a specific
segment type called the ***Task State Segment*** 
(TSS), to store hardware contexts.
* But Linux doesn't use TSS for hardware context switches.
* However Linux is nonetheless forced to set up a TSS for each distinct CPU in the system.

## Task State Segment Components Used by Linux
* When an 80 x 86 CPU switches from User Mode to Kernel Mode, it fetches the
***address of the Kernel Mode stack*** from the TSS.
* When a User Mode process attempts to
access an I/O port by means of an **in** or **out**
instruction, the CPU may need to access an
**I/O Permission Bitmap** stored in the TSS to
verify whether the process is allowed to address the port.

## tss_struct Structure
* The **tss_struct** structure describes the format of the **TSS**.
* The **init_tss** per- CPU variable stores one TSS for each CPU on the system. 
* At each ***process switch***, the kernel updates
some fields of the TSS so that the
corresponding CPU's **control unit** may safely
retrieve the information it needs.
* But there is no need to maintain TSSs for processes when they're not running.

## Task State Segment Descriptor
* Each TSS has its own 8-byte ***Task State Segment Descriptor*** (TSSD).
* This descriptor includes ~

## Busy Bit
* In the Intel's original design, each process in the system should refer to its own TSS. 
* The second least significant bit of the **Type** field (4 bits) of the corresponding TSSD is called the ***Busy bit***
* *In Linux design, there is just one TSS for each CPU, so the Busy bit is always set to 1.*

## TSSD-related Registers
* The TSSDs created by Linux are stored in the ***Global Descriptor Table*** (GDT), 
whose base address is stored in the **gdtr** register of each CPU.
* The **tr** register of each CPU contains the TSSD Selector of the corresponding TSS. 

## TSSD and init_tss Array

## The thread Field
* At every process switch, the hardware context of
the process being replaced must be saved somewhere.
* It cannot be saved on the TSS, as in the original
Intel design, because Linux uses a single TSS for
each processor, instead of one for every process.
* Thus, each ***process descriptor*** includes a field
called **thread** of type **thread_struct**, in
which the kernel saves the hardware context
whenever the process is being switched out. 

## Overview of the thread_sturct
* Data structure **thread_struct**  includes
fields for most of the CPU registers,
except the general-purpose registers such
as **eax**, **ebx**, etc., which are stored in the
***Kernel Mode stack***.

## Where Could a Process Switch Occur?
* A **process switch** may occur at just one
well-defined point: the **schedule( )** function. 

## Performing the Process Switch
* Essentially, every process switch consists of two steps:
	* Switching the ***Page Global Directory*** to install a new ***address space***.
	* Switching the ***Kernel Mode stack*** and the ***hardware context***,
which provides all the information needed by the kernel to execute
the new process, including the CPU registers.
