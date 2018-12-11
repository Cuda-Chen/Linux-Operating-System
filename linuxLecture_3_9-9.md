# linux lecture 3.9 chap. 9

## Processes Switch

## switch_to Macro
* Assumptions:
	* *local variable* **prev** refers to the process descriptor of the process being switched out.
	* **next** refers to the one being switched in to replace it.
* **switch_to(prev,next,last)** macro

## Why 3 Processes Are Involved in a Context Switch?

## Why Reference to C Is Needed?
* To complete the process switching.

## The last Parameter

## Code Execution Sequence & Get the Correct Previous Process Descriptor

## From schedule to switch_to
1. schedule()
2. __schedule()
3. context_switch()
4. switch_to()

## Simplification for Explanation
* The **switch_to** macro is coded in ***extended inline assembly language*** that makes for rather complex reading.
* In fact, the code refers to __registers__ by means of a
special __positional notation__ that allows the compiler to
freely choose the general-purpose registers to be used.
* Rather than follow the extended inline assembly language,
we'll describe what the **switch_to**
macro typically does on an 80x86 microprocessor by
using standard assembly language.

## switch_to (1)
* Save the value of **prev** and **next** in
the **eax** and **edx** registers, respectively:
```
movl prev, %eax
movl next, %edx
```
The **eax** and **edx** registers correspond to the **prev** and **next** parameters of the macro.

## switch_to (2)
* Saves the contents of the **eflags** and **ebp** registers in the **prev** Kernel Mode stack.
* They must be saved because the compiler assumes that they will stay unchanged until the end of **switch_to**:
```
pushfl
pushl %ebp
```

## switch_to (3)
* Saves the content of **esp** in **prev->thread.sp** so that the field points to the top of the **prev** Kernel Mode stack:
```
movl %esp,484(%eax)
```
* The **484(%eax)** operand identifies the memory
cell whose address is the contents of **eax** plus **484**.

## switch_to (4)
* Loads **next->thread.sp** in **esp**.
	* From now on, the kernel operates on the Kernel Mode stack of **next**
, so this instruction performs the actual process switch from **prev** to **next**. 
```
movl 484(%edx), %esp 
```

## switch_to (5)
* Saves the address labeled **1** (shown later in this section) in **prev->thread.ip**.
* When the process being replaced resumes its execution
, the process executes the instruction labeled as **1**:
```
movl $1f, 480(%eax)
```

## switch_to (6)
* On the Kernel Mode stack of **next**
, the macro pushes the **next->thread.ip** value
, which, in most cases, is the address labeled as **1**:
```
pushl 480(%edx)
```

## switch_to (7)
* Jumps to the **__switch_to( )** C function:
```
jmp __switch_to 
```

# switch_to
## The switch_to() function
* The **__switch_to( )** function does the bulk of the 
process switch started by the **switch_to( )** macro. 
* It acts on the **prev_p** and **next_p** parameters that
denote the ***former process*** (e.g. process **C** of slide 7)
and the ***new process*** (e.g. process **A** of slide 7). 
* This function call is different from the average function
call, though, because **__switch_to( )** takes the
**prev_p** and **next_p** parameters from the **eax** and **edx**
registers (where we saw they were stored), not from the stack like most functions.

# *Before and Including Linux 2.6.24*
## Get Function Parameters from Registers
* To force the function to go to the registers
for its parameters, the kernel uses the 
**__attribute__** and **regparm**
keywords, which are nonstandard
extensions of the C language implemented
by the **gcc** compiler.

## Function Prototype of __switch_to( ) 
* The **__switch_to( )** function is declared in the
**include/asm-i386/system.h** header file as follows:
```
__switch_to(struct task_struct *prev_p, struct task_struct * next_p) __attribute__(regparm(3));
```

# After and Including Linux 2.6.25
## Function Prototype of __switch_to( )
```
__notrace_funcgraph struct task_struct * __switch_to(struct task_struct *prev_p, struct task_struct *next_p)
```

## __switch_to (1)
* Executes the **smp_processor_id( )**
macro to get the index of the local CPU,
namely the CPU that executes the code.
* stores the result into the cpu local variable.

## Definition of cpu_number
```
DEFINE_PER_CPU_READ_MOSTLY(int, cpu_number);
EXPORT_PER_CPU_SYMBOL(cpu_number);
```

## Initialization of cpu_number
```
void __init setup_per_cpu_areas(void)
{
  unsigned int cpu;
  unsigned long delta;
  int rc;

         :
  for_each_possible_cpu(cpu) {
     per_cpu_offset(cpu) = delta + pcpu_unit_offsets[cpu];
     per_cpu(this_cpu_off, cpu) = per_cpu_offset(cpu);
     per_cpu(cpu_number, cpu) = cpu;
                  :
  }
         :
}
```

## __switch_to (2)
* Loads **next_p->thread.sp0** into the **sp0**
field of the TSS relative to the local CPU
; as we'll see in the section "Issuing a System Call
via the **sysenter** Instruction"
, any future privilege level change from User Mode to
Kernel Mode raised by a **sysenter** assembly
instruction will copy this address into the **esp** register:
```
tss->x86_tss.sp0 = thread->sp0;
```
* P.S. When a process is created
, function copy_thread() set the **sp0** field to point the next byte of the last byte of the kernel mode stack of the new born process.

## __switch_to (3)
* Loads in the ***Global Descriptor Table*** of the local CPU
the ***Thread-Local Storage*** (TLS) segments used by the
**next_p** process.
* The above three Segment Selectors are stored in the
**tls_array** array inside the __process descriptor__.
```
#define GDT_ENTRY_TLS_MIN        6

per_cpu(gdt_page, cpu).gdt[GDT_ENTRY_TLS_MIN + 0] = next->tls_array[0];
per_cpu(gdt_page, cpu).gdt[GDT_ENTRY_TLS_MIN + 1] = next->tls_array[1];
per_cpu(gdt_page, cpu).gdt[GDT_ENTRY_TLS_MIN + 2] = next->tls_array[2];
```

## __switch_to (4)
* Stores the contents of the **gs** segmentation registers in **prev_p->thread.gs**; 
```
#define savesegment(seg, value)                         \
        asm("mov %%" #seg ",%0":"=r" (value) : : "memory")


#define lazy_save_gs(v)         savesegment(gs, (v))


__switch_to(…){ …
struct thread_struct *prev = &prev_p->thread;
        …
lazy_save_gs(prev->gs);
     …
}     
```

## __switch_to (5)
* If the the **gs** segmentation register have been used either
by the **prev_p** or by the **next_p** process (having nonzero
values), loads into **gs** the value stored in the
**thread_struct** descriptor of the **next_p** process.
```
__switch_to(…){ …
struct thread_struct *next = &next_p->thread;
        …
 if (prev->gs | next->gs)
    lazy_load_gs(next->gs);
        …
} 
```

## __switch_to (6)
* Updates the I/O bitmap in the TSS, if necessary. :
```
void __switch_to_xtra(struct task_struct *prev_p, struct task_struct *next_p,
                      struct tss_struct *tss)
{  struct thread_struct *prev, *next;
   prev = &prev_p->thread;
   next = &next_p->thread;
                ...
   if (test_tsk_thread_flag(next_p, TIF_IO_BITMAP)) 
   {
     memcpy(tss->io_bitmap, next->io_bitmap_ptr, max(prev->io_bitmap_max, next->  
     io_bitmap_max));
   } else 
                ...
}
```

## Make current to Point to the next Process  (7)
* Function **__switch_to()** invokes
**this_cpu_write(current_task, next_p)** to make macro **current** to
point to the **next** process.

## 
