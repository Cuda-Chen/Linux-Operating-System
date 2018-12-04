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
