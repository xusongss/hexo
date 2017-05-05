---
title: Coding Rules for Writing Safety Critical Program
date: 2017-05-05 15:30:27
tags:
---


# Coding Rules for Writing Safety Critical Program
The large and complex software projects use some sort of coding standards and guidelines. These guidelines establish the ground rules that are to be followed while writing software.
1. How the code should be structured?
2. Which language feature should or should not be used?

In order to be effective, the set of rules has to be small and must be specific enough that it can be easily understood and remembered.

**The world’s top programmers working for NASA follow a set of guidelines for developing safety critical code. In fact, many organizations, including NASA’s Jet Propulsion Laboratory (JPL) focus on code written in C programming language.﻿
The reason is, there is extensive tool support for this language, including, logic model extractors, debuggers, stable compiler, strong source of code analyzers and metrics tools.**
![NASA-10-Coding-Rules](/img/NASA-10-Coding-Rules.jpg)
Sometimes coding rules are essential to use, especially where your life may depend on its correctness – code that is used to control the airplane that you fly on, spacecraft that carries astronauts into orbit, or the nuclear power plant a few miles from where you live. Introducing the NASA’s 10 coding rules that focus on security parameters, laid by JPL lead scientist Gerard J. Holzmann. The guidelines could be applied to other programming languages as well.
<!-- more -->
## Rule No. 1 – Simple Control Flow
  Write program with very simple control flow constructs – Do not use setjmp or longjmp constructs, goto statements, and direct or indirect recursion.

Reason: Simple control flow results in improved code clarity and stronger capabilities for verification. Without recursion, there will be no cyclic function call graph, and this proves that all executions that should be bounded are in fact bounded.

## Rule No. 2 – Fixed Upper Bound for Loops
All loops must have a fixed upper-bound. It should be possible for a verification tool to prove statically that a preset upper-bound on the number of iteration of a loop can’t be exceeded.
The rule is considered violated if the loop-bound can’t be proven statically.

Reason: The presence of loop bounds and absence of recursion prevents runaway code. However, the rule doesn’t apply to iterations that are meant to be non-terminating (for example, process scheduler). In those cases, reverse rule is applied – It must be statically provable that iteration cannot terminate.

## Rule No. 3 – No Dynamic Memory Allocation
Do not use dynamic memory allocation after initialization.

Reason: Memory allocators like malloc, and garbage collectors often have unpredictable behavior that can exceptionally impact performance. Moreover, memory errors can also occur because of programmer’s mistake, which includes

- Attempting to allocate more memory than physically available
- Forgetting to free memory
- Continuing to use memory after it was freed
- Overstepping boundaries on allocated memory

Forcing all modules to live within a fixed, pre-allocated area of storage can eliminate these problems and make it easier to verify memory use.

The one way to dynamically claim memory in the absence of memory allocation from the heap is to use stack memory.
## Rule No. 4 – No Large Functions
No function should be longer than what could be printed on a single sheet of paper in a standard reference format with one line per declaration and one line per statement. This means a function shouldn’t have more than 60 lines of code.

Reason: Excessively long functions are often a sign of poor structure. Each function should be a logical unit that is understandable as well as verifiable. It’s quite harder to understand a logical unit that spans multiple screens on a computer display.

## Rule No. 5 – Low Assertion Density
![Low-Assertion-Density](/img/Low-Assertion-Density.jpg)
The assertion density of the program should average to a minimum of two assertions per function. Assertions are used to check for abnormal conditions that should never happen in real-life executions. They should be defined as Boolean tests. When an assertion fails, an explicit recovery action should be taken.
If a static checking tool proves that assertion can never fail or never hold, the rule is considered violated.

Reason: Industry coding-efforts stats shows that unit tests often find at least one defect per 10 to 100 lines of code written. The chances of intercepting defects increase with assertion density.
Use of assertion is also important as they are part of strong defensive coding strategy. They are used to verify pre and post conditions of a function, parameter and return value of a function and loop-invariants. Assertions can be selectively disabled after testing performance critical code.

## Rule No. 6 – Declare Data Objects at Smallest Level of Scope
This one supports the basic principle of data hiding. All data objects must be declared at the smallest possible level of scope.

Reason: If an object is not in scope, its value cannot be referenced or corrupted. This rule discourages the re-use of variables for multiple, incompatible purposes that can complicate fault diagnosis.

## Rule No. 7 – Check Parameters and Return Value
The return value(s) of a non-void functions should be checked by each calling function, and the validity of parameters should be checked inside each function.
In its strictest form, the rule means even the return value of printf statements and file close statements should be checked.

Reason: If the response to an error would rightfully be no different than the response to success, there is a point in explicitly checking a return value. This is usually the case with calls to close and printf. It is acceptable to explicitly cast the function return value to void – indicating that the coder explicitly (not accidentally) decides to ignore a return value.

Read: [Best Programming Software | for writing code](http://www.rankred.com/best-programming-software-for-writing-codes/)
## Rule No. 8 – Limited Use of Preprocessor
The use of the preprocessor should be limited to the inclusion of header files and macro definitions. Recursive macro calls, token pasting, and variable argument lists are not allowed. There should be justification for more than one or two conditional compilation directives even in large application development efforts, beyond the standard boilerplate, which avoids multiple inclusion of the same header file. Each such use must be flagged by a tool based checker and justified in the code.

Reason: The C preprocessor is a powerful and ambiguous tool that can destroy the code clarity and confuse many text based checkers. The effect of constructs in unbounded preprocessor code could be exceptionally hard to decipher, even with a formal language definition in hand.
The caution against conditional compilation is equally important – with just 10 conditional compilation directives, there could be 1024 possible versions (2^10) of the code, which would increase the required testing effort.
## Rule No. 9 – Limited Use of Pointers
The use of pointers must be restricted. No more than one level of dereferencing is permitted. Pointer dereference operations should not be hidden inside typedef declaration or macro definitions. Also, function pointers are not allowed.

Reason: Pointers are easily misused, even by experts. They make it hard to follow or analyze the flow of data in a program, especially by tool-based static analyzers. Function pointers also restrict the type of checks performed by static analyzers, and should only be used if there is a strong justification for their use. If function pointers are used, it becomes almost impossible for a tool to prove absence of recursion, so alternative methods would have to be provided to make up for this loss in analytical capabilities.

Read: [Amazing NASA Inventions That We Use In Our Daily Life](http://www.rankred.com/amazing-nasa-inventions/)
## Rule No. 10 – Compile all Code
All code must be compiled from the first day of development. The compiler warning must be enabled at the compiler’s most punctilious setting. The code must compile with these settings without any warning.
All code should be checked daily with at least one (preferably more than one) state-of-the-art static source code analyzer and should pass the analyses process with zero warning.

Reason: There are plenty of effective source code analyzers available in the market; a few of them are freeware tool as well. There is absolutely no excuse for any software development effort not to make use of this readily available technology.
If the compiler or static analyzer gets confused, the code causing the confusion/error should be rewritten so that it becomes more trivially valid.
