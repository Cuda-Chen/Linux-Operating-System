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
