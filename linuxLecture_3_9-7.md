# linuxLecture 3.9, Chap. 7

## NameSpace
* Currently, Linux implements six different types of namespaces.
* The **CLONE_NEW\*** identifiers listed in 
parentheses are the names of the 
constants used to identify namespace 
types when employing the namespace-related APIs
## Goals of Namespace
1. to wrap a particular global system resource in an abstraction 
that makes it appear to the processes within the namespace that 
they have their own isolated instance of the global resource
2. to support the implementation of containers, a tool for lightweight virtualization
	* ... that provides a group of processes with the illusion 
that they are the only processes on the system

## PID Namespace
* The global resource isolated by PID namespaces is the ***process ID number space***.
* This means that processes in different PID namespaces can have the ***same process ID***.
* PID namespaces are used to implement ***containers*** 
that can be migrated between host systems 
while keeping the same process IDs for the processes inside the container.

## Process PID
* As with processes on a traditional Linux (or UNIX) system, 
the process IDs within a PID namespace are unique
	* and are assigned sequentially starting with **PID 1**.
* Likewise, as on a traditional Linux system, 
**PID 1**—the **init** process—is special
	* it is the first process created within the namespace, 
and it performs certain management tasks within the namespace.

## Creation of a New PID Namespace
* A new PID namespace is created by calling **clone()** with the **CLONE_NEWPID** flag.

## PID Namespace Hierarchy
* PID namespaces form a hierarchy
	* A process can "see" only those processes contained in its own PID namespace
		* and in the **child namespaces* nested below that PID namespace. 
* If the parent of the child created by **clone()** is in a different namespace, 
the child cannot "see" the parent
	* therefore, **getppid()** reports the parent PID as being zero.

## Nested PID Namespaces
* PID namespaces are hierarchically nested in *parent-child relationships*.
* Within a PID namespace, it is possible to see
	* all other processes in the same namespace,
	* all processes that are members of *descendant namespaces*.

## “See” a Process
* Here, "see" means being able to make system calls that operate on specific PIDs.
* Processes in a *child PID namespace* cannot see processes 
that exist (only) in the parent PID namespace (or further removed ancestor namespaces).

## PID returned by getpid()
* A process will have one PID 
in each of the layers of the PID namespace hierarchy 
starting from the PID namespace in which
 it resides through to the *root PID namespace*.
* Calls to **getpid()** always report the PID
 associated with the namespace
 in which the process resides

## Traditional **init** Process and Signals
* The traditional Linux **init** process is treated specially with respect to ***signals***.
	* The only signals that can be delivered to **init**
 are those for which the process has established a *signal handler*.
* All other signals are ignored.
* This prevents the **init** process
—whose presence is essential for the stable operation of the system--
from being accidentally killed, even by the super user.

## **init** Processes of  Namespaces and Signals
* PID namespaces implement some analogous behavior for the namespace-specific **init** process.
	* Other processes in the namespace can send only those signals
 for which the **init** process has established a *signal handler*. 
	* Note that the kernel can still generate signals
 for the PID the kernel can still generate signals for the PID

## Signals  from Ancestor Namespaces
* Signals can be sent to the PID namespace **init** process
 by processes in ancestor PID namespaces.
* Again, only the signals
 for which the **init** process has established a handler can be sent
, with two exceptions
	* SIGKILL
	* SIGSTOP

## *init* Process and *SIGKILL* and *SIGSTOP*
* When a process in an ancestor PID namespace sends **SIGKILL**
 and **SIGSTOP** to the **init** process
, they are forcibly delivered (and can't be caught).
* The **SIGSTOP** signal stops the **init** process; **SIGKILL** terminates it.

## Termination of *init* Processes
* Since the **init** process is essential to the functioning of the PID namespace
 if the **init** process is terminated by **SIGKILL** (or it terminates for any other reason)
, the kernel terminates all other processes in the namespace
 by sending them a **SIGKILL** signal.

## Connection between Processes and Namespaces

## struct nsproxy
* A structure to contain pointers to all per-process
 namespaces - fs (mount), uts, network, ipc, etc.
* '**count**' is the number of processes
 holding a reference.

## Process Identification Number
* Unix processes are always assigned a
 number to uniquely identify them in their
 namespace.
* This number is called the *process
 identification number* or **PID** for short.
* Each process generated with **fork** or
 **clone** is automatically assigned a new
 unique **PID** value by the kernel.

## Process ID
* PIDs are numbered sequentially in each PID namespace
* Of course, there is an upper limit on the PID values
	* when the kernel reaches such limit, it must start recycling the
 lower, unused PIDs.

## PIDs in PID Namespaces
* Namespaces add some additional
 complexity to how PIDs are managed.
* PID namespaces are organized in a hierarchy.

## A Process May Have Multiple PIDs
* When a new namespace is created,
 all PIDs that are used in this namespace are visible to
 the *parent namespace*, but the *child namespace*
 does not see PIDs of the parent namespace.
* However this implies that some processes are
 equipped with more than one PID, namely,
 one per namespace they are visible in. This
 must be reflected in the data structures. 

## Global IDs
* *Global IDs* are identification numbers that
 are valid within the kernel itself and in the
 *initial global namespace*.
* For each ID type, a given global identifier
 is guaranteed to be unique in the whole
 system.

## Local IDs
* *Local IDs* belong to a specific namespace
 and are not globally valid.
* For each ID type, they are valid within the
 namespace to which they belong, but
 identifiers of identical type may appear
 with the same ID number in a *different namespace*.

## Global PID and TGID
* The global PID and TGID are directly
 stored in the **task_struct**, namely, in the elements **pid** and **tgid**

## PIDs and Processes
* Linux associates a different **PID** with each
 ***process*** or ***lightweight process*** in the system.
	* As we shall see later in this chapter, there is a tiny exception on multiprocessor systems.
* This approach allows the maximum flexibility,
 because every ***execution context*** in the
 system can be uniquely identified.

## Threads in the Same Group Must Have a Common PID 
* On the other hand, Unix programmers
 expect ***threads in the same group*** to have
 a common PID.

## Thread Group
* To comply with **POSIX 1003.1c** standard,
 Linux makes use of ***thread groups***.
* The ***identifier*** shared by the threads is the
 __PID of the thread group leader__, that is, 
the PID of the first lightweight process in the group.
* The thread group ID of a thread group is called **TGID**.

## Process Groups
* Modern Unix operating systems introduce the
 notion of ***process groups*** to represent a ***job*** abstraction.
* One important feature is that it is possible
 to send a signal to every process in the
 group.
* Process groups are used 
	* for distribution of signals
	* by terminals to arbitrate requests for their input and output

* Foreground Process Groups
	* A foreground process has read and write access to the terminal.
	* Every process in the foreground receives **SIGINT** (^C ) **SIGQUIT** (^\ ) and **SIGTSTP** signals.
* Background Process Groups
	* A background process does not have read access to the terminal.
	* If a background process attempts to read from its controlling terminal its process group will be sent a **SIGTTIN**.

## Group Leaders and Process Group IDs
* Each process descriptor includes a field containing the __process group ID__.
* Each group of processes may have a
 ***group leader***, which is the process whose
 PID coincides with the process group ID.

## Creation of a New Process Group
* A newly created process is initially inserted into
 the __process group of its parent__. 
* The shell after doing a fork would explicitly call
 **setpgid** to set the process group of the child.
* The process group is explicitly set for purposes of ***job control***.
* When a command is given at the ***shell prompt***,
 that process or processes (if there is piping) is
 assigned a new process group.

## Login Sessions

## Login Sessions vs. Process Groups
* All processes in a ***process group*** must be in the same ***login session***.
* A login session may have several process groups active simultaneously
	* one of these process groups is always in the ***foreground***, which means that it has access to the terminal.
	* The other *active process groups* are in the ***background***.

## child_reaper Field
* Every PID namespace is equipped with a
 process that assumes the role taken by
 **init** in the global picture.
* One of the purposes of **init** is to call
 **wait4** for orphaned processes, and this
 must likewise be done by the **init** process
 of the namespace.
* A pointer to the task structure of this
 process is stored in **child_reaper**.

## parent Field
* **parent** is a pointer to the parent namespace,
 and **level** denotes the depth in the
 namespace hierarchy.
* The ***initial namespace*** has level 0, any children
 of this namespace are in level 1, children of
children are in level 2, and so on.
* Counting the levels is important because IDs
in higher levels must be visible in lower levels.

## pidmap Field

## PID bitmap
* To keep track of which PIDs have been
allocated and which are still free, the
kernel uses a large bitmap in which each PID is identified by a bit. 
* The value of the PID is obtained from the position of the bit in the **bitmap**.

## Allocate a Free PID
* Allocating a free PID is then restricted
essentially to looking for the first bit in the
bitmap whose value is 0; this bit is then set to 1.

## Free a PID
* Freeing a PID can be implemented by
‘‘toggling‘‘ the corresponding bit from 1 to 0.

## struct upid
* **struct upid** represents the information
that is visible in a specific namespace.

## Fields of struct upid 
* **nr** represents the numerical value of an ID,
and **ns** is a pointer to the namespace to
which the value belongs. 
* All *upid instances* are kept on a hash table to
which we will come in a moment, and
**pid_chain** allows for implementing hash
overflow lists with standard methods of the kernel.

## The Kernel-internal Representation of A PID
* **struct pid** is the kernel-internal representation of a PID.

## Type enum pid_type
* Notice that *thread group IDs* are not contained in this collection.
* This is because the *thread group ID* is simply
given by the PID of the *thread group leader*, so a
separate entry is not necessary.

## level Field of struct pid
* A process can be visible in multiple
namespaces, and the local ID in each
namespace will be different.
* **level** denotes in how many namespaces
the process is visible (in other words, this
is the depth of the containing namespace
in the namespace hierarchy).

## numbers Field of struct pid
* **numbers** contains a ***upid instance*** for each level.

## tasks Field of struct pid
* The definition of **struct pid** is headed by a reference counter.
* **tasks** is an array with a ***list head*** for every ID
type. This is necessary because an ID can be
used for several processes.
* All __**task_struct instances**__ that share a given ID are linked on this list.
* **PIDTYPE_MAX** denotes the number of ID types.

## **pids** Field of **struct task_struct** 
* Since all **tast_struct** structures that share
an identifier are kept on a list headed by
**tasks**, a list element is required in **struct task_struct**

## Create a New pid Instance
* When a new process is created, a new **pid** instance is also created.

## **pids[]** Fields of a New Process

## Thread Group
* Processes in the same thread group are
chained together through the **thread_group**
field of their **tast_struct** structures.

## Function attach_pid()
* Suppose that a new instance of **struct pid** has
been allocated and set up for a given ID type. 

## **struct pid** related Helper Functions
* Obtain the **pid** instance associated with the **task_struct** structure.
* The auxiliary functions **task_pid**, **task_tgid**, **task_pgrp**, and **task_session** are provided for the different types of IDs.

## Numerical PID related Helper Functions (1)
* Once the **pid** instance is available, the numerical ID
can be read off from the **upid** information
available in the **numbers** array in **struct pid**.

## Numerical PID related Helper Functions (2)
* **pid_vnr** returns the local PID seen from the namespace to which the ID belongs.
* **pid_nr** obtains the global PID as seen from the init process.
* Both rely on **pid_nr_ns** and automatically select the proper level
	* 0 for the global PID
	* **pid->level** for the local one


