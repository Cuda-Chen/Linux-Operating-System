# Linux Kernel 3.9 -- System Call 

## System Call
* Operating systems offer processes running in
User Mode ***a set of interfaces*** to interact with
***hardware devices*** such as 
	* CPU
	* disk
	* printers
* UNIX systems implement most interfaces between
***User Mode processes*** and ***hardware devices***
by means of ***system calls*** issued to the kernel.

## POSIX APIs vs. System Calls
* An ***application programmer interface*** is
a ***function definition*** that specifies how to
obtain a given service.
* A ***system call*** is an explicit request to the
kernel made via a ***software interrupt***.

## From a Wrapper Routine to a System Call
* Unix systems include several ***libraries of
functions*** that provide APIs to programmers. 
* Some of the APIs defined by the libc standard
C library refer to ***wrapper routines*** (routines
whose only purpose is to issue a ***system call***).
* Usually, each system call has a corresponding
wrapper routine, which defines the API that
application programs should employ. 

## APIs and System Calls
* An API does not necessarily correspond to a
specific system call. 

## Example of Different APIs Issuing the Same System Call
* In Linux, the
	* malloc
	* calloc
	* free
APIs are implemented in the libc library.
* The code in this library keeps track of the
allocation and deallocation requests and uses
the **brk()** system call to enlarge or shrink
the ***process heap***.

## The Return Value of a Wrapper Routine
* Most wrapper routines return an integer value,
whose meaning depends on the corresponding system call. 
* A return value of **-1** usually indicates that the
***kernel*** was unable to satisfy the process request.
* A failure in the ***system call handler*** may be
caused by
	* invalid parameters
	* a lack of available resources
	* hardware problems
	* , and so on. 
* The specific ***error code*** is contained in the **errno**
variable, which is defined in the **libc** library.

## Execution Flow of a System Call
* When a ***User Mode process*** invokes a ***system call***,
the CPU switches to **Kernel Mode** and starts the execution of a ***kernel function***.
	* in the 80x86 architecture a Linux system call can be invoked in two different ways
* The net result of both methods, however, is a
jump to an assembly language function called the
***system call handler***.

## System Call Number
* Because the kernel implements many different
system calls, the __User Mode process__ must pass a
__parameter__ called the ***system call number*** to
identify the required system call.
* The **eax** register is used by Linux for this purpose. 

## The Return Value of a System Call
* All system calls return an integer value. 
* The __conventions for these return values__ are
different from __those for wrapper routines__. 
	* In the kernel
		* positive or 0 values denote a successful termination of the system call
		* negative values denote an error condition 
			* In this case, the value is the negation of the error code that must be returned to the application program in the **errno** variable.
* The **errno** variable is not set or used by the kernel.
	* Instead, the wrapper routines handle the task of setting
this variable after a return from a system call.

## Operations Performed by a System Call
* The **system call handler**, which has a structure
similar to that of the other ***exception handlers***,
performs the following operations:
	* Saves the contents of most registers in the Kernel Mode stack.
	* Handles the system call by invoking a corresponding C function called the ***system call service routine***.
	* Exits from the handler

## Naming Rules of System Call Service Routines
* The name of the service routine associated
with the xyz( ) system call is usually
**sys_xyz()***; there are, however, a few
exceptions to this rule.

## Control Flow Diagram of a System Call

## DEFINE(sym, val)
* In the Linux kernel you find assembly as follows:
```c
#define DEFINE(sym, val) \ 
   asm volatile("\n->" #sym " %0 " #val : : "i" (val)) 
```
* which when used like this
```c
DEFINE(NR_PAGEFLAGS, __NR_PAGEFLAGS); 
```
* generates the following assembly
```
->NR_PAGEFLAGS $24 __NR_PAGEFLAGS
```
* It gets transformed using a sed script into something like:
```c
#define NR_PAGEFLAGS 24
```

## NR_syscalls – (1)
* 
```
DEFINE( __NR_syscall_max, sizeof(syscalls) - 1);
  -> #define __NR_syscall_max  sizeof(syscalls) - 1
```
* 
```
DEFINE(NR_syscalls, sizeof(syscalls));
  -> #define NR_syscalls  sizeof(syscalls)
```

## NR_syscalls – (2)
* 
```c
#define __SYSCALL_I386(nr, sym, compat) [nr] = 1,
  static char syscalls[] = 
  {
    #include <asm/syscalls_32.h>
  };
```
* Note that **asm/syscalls_32.h** is created **dynamically** (i.e., while compiling), so
you can't find it in source code.

## File asm/syscalls_32.h
* File **asm/syscalls_32.h** is created during compilation based on header files
, such as **/include/uapi/asm-generic/unistd.h**.

## Macro __SYSCALL_I386
*
```c
#define __SYSCALL_I386(nr, sym, compat) [nr] = 1,
```

## Array syscalls[] 
*
```c
static char syscalls[] = {
    [0] = 1,
    [1] = 1,
    [2] = 1,
    [3] = 1,
    //...
};
```

## System Call Dispatch Table 
* To associate each ***system call number*** with
its corresponding ***service routine***, the kernel
uses a ***system call dispatch table***, which is
stored in the **sys_call_table** array and
has **__NR_syscall_max+1** entries.
	* **NR_syscalls**
* The ***n***-th entry contains the service routine address of the system call having number ***n***.

## NR_syscalls
* The **NR_syscalls** is just a static limit on the
maximum number of implementable system calls; it
does not indicate the number of system calls
actually implemented. 
* Indeed, each entry of the dispatch table may
contain the address of the **sys_ni_syscall( )**
function, which is the service routine of the
"nonimplemented" system calls; it just returns the
error code **-ENOSYS**.

## sys_call_table Array
```c
#define __SYSCALL_I386(nr, sym, compat) [nr] = sym,
const sys_call_ptr_t sys_call_table[__NR_syscall_max+1] =
{
  /*
   * Smells like a compiler bug -- it doesn't work
   * when the & below is removed.
  */
  [0 ... __NR_syscall_max] = &sys_ni_syscall,
  #include <asm/syscalls_32.h>
};
```
* **“[0 ... __NR_syscall_max] = &sys_ni_syscall“** uses the
address of function **sys_ni_syscall** to initialize each element of
array **sys_call_table**. 
* Then use file **asm/syscalls_32.h** to reinitialize some entries of the array.

## Redefine Macro __SYSCALL_I386
```c
#define __SYSCALL_I386(nr, sym, compat) [nr] = sym,
```

## sys_call_table[]
```c
const sys_call_ptr_t sys_call_table[__NR_syscall_max+1] = 
{
    [0 ... __NR_syscall_max] = &sys_ni_syscall,
    [0] = sys_restart_syscall,
    [1] = sys_exit,
    [2] = sys_fork,
    [3] = sys_read,
    //...
};
```

## Ways to Invoke a System Call
* Applications can invoke a system call in two different ways:
	* By executing the **int $0x80** assembly language instruction
	* By executing the **sysenter** assembly language instruction, introduced in the Intel Pentium II microprocessors

## Ways to Exit a System Call
* The kernel can exit from a system call thus
switching the CPU back to User Mode in
two ways:
	* By executing the **iret** assembly language instruction.
	* By executing the **sysexit** assembly language instruction
, which was introduced in the Intel Pentium II microprocessors together with the **sysenter** instruction.

## Interrupt Descriptor Table
* A system table called ***Interrupt Descriptor Table*** (IDT)
associates each ***interrupt or exception vector*** with the
***address*** of the corresponding ***interrupt or exception handler***.
* The IDT must be properly initialized before the kernel enables interrupts.
* The IDT format is similar to that of the GDT and the LDTs.
* Each entry corresponds to an interrupt or an exception
vector and consists of an 8-byte descriptor. Thus, a
maximum of 256 x 8 = 2048 bytes are required to store the IDT.

## idtr CPU register
* The **idtr** CPU register allows the IDT to be
located anywhere in memory: it specifies
both the __IDT base physical address__ and __its limit__ (maximum length). 
* It must be initialized before enabling
interrupts by using the **lidt** assembly
language instruction.

## Types of IDT Descriptors
* The IDT may include three types of descriptor
	* Task gate
	* Interrupt gate
	* Trap gate 
		* Used by system calls

## Layout of a Trap Gate

## Vector 128 of the Interrupt Descriptor Table
* The vector **128**, or (**0x80**)_16, is associated with the kernel entry point. 
* The **trap_init()** function, invoked
during kernel initialization, sets up the
**Interrupt Descriptor Table entry**
corresponding to vector 128 as the next slide. 

## Set the IDT Entry for System Calls
```c
#ifdef CONFIG_X86_32
  #define SYSCALL_VECTOR                 0x80
#endif
```
```c
#ifdef CONFIG_X86_32
 set_system_trap_gate(SYSCALL_VECTOR, &system_call);
 set_bit(SYSCALL_VECTOR, used_vectors);
#endif
```

## set_system_trap_gate(0x80, &system_call)
* The call loads the following values into the gate descriptor fields:
	* Segment Selector
	* Offset
	* Type
	* DPL (**Descriptor Privilege Level**)
* Therefore, when a User Mode process issues an
**int $0x80** instruction, the CPU switches into Kernel Mode and
starts executing instructions from the **system_call** address.

## Save Registers
* The **system_call( )** function starts by saving
the ***system call number*** and all the CPU registers
that may be used by the ***exception handler*** on the
stack except for **eflags**, **cs**, **eip**, **ss**, and **esp**,
which have already been saved automatically by
the control unit. 
## Linux Macro (1)
* Macros are defined within **.macro** and **.endm** statements.
* The macro definition bellow is interchangeable between Linux and Mac.
```
.macro NEXT 
lodsl 
jmp *(%eax) 
.endm 
```

## Linux Macro (2)
* 
