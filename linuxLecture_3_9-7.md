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


