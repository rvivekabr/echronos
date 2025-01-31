/*| provides |*/
acrux
rtos

/*| requires |*/
docs

/*| doc_header |*/
<!-- %title eChronos RTOS Manual: Acrux Variant -->
<!-- %version 0.1 -->
<!-- %docid T57d49 -->


# Introduction

This document provides the information that system designers and application developers require to successfully create reliable and efficient embedded applications with the eChronos real-time operating system.

The [Concepts] chapter presents the fundamental ideas and functionalities realized by the RTOS and how developers can harness them to successfully construct systems.

The [API Reference] chapter documents the details of the run-time programming interface that applications use to interact with the RTOS.

The [Configuration Reference] chapter details the interface to the build-time configuration of the RTOS that system designers use to tailor the RTOS to their applications.

Throughout this document, *eChronos RTOS* or *the RTOS* will refer specifically to the *Acrux* variant of the eChronos RTOS.

/*| doc_concepts |*/
## Overview

The eChronos RTOS facilitates the rapid development of reliable, high-performance embedded applications.
It allows developers to focus on the application logic by wrapping the complexities of low-level platform and system code in a comprehensive, easy-to-use operating-system API.
Since each application configures the RTOS to its specific requirements, this document refers to the combination of RTOS and application code simply as the *system*.

In terms of its functionality, the RTOS is a task-based operating system that multiplexes the available CPU time between tasks.
Since it is non-preemptive, tasks execute on the CPU until they voluntarily relinquish the CPU by calling an appropriate RTOS API function.
The RTOS API (see [API Reference]) gives tasks access to the objects that the RTOS provides.

A distinctive feature of the RTOS is that these objects, including tasks, are defined and configured at build time (see [Configuration Reference]), not at run time.
This configuration defines, for example, the tasks that exist in a system at compile and run time.
Static system configuration like this is typical for small embedded systems.
It avoids the need for dynamic memory allocation and permits a much higher degree of code optimization.
The [Configuration Reference] chapter describes the available configuration options for each type of object in the RTOS.


## The Acrux Variant

The acrux variant of the RTOS is a bare-bones RTOS with very limited functionality: support for [Task States], [Mutexes], a [round-robin scheduler](#scheduling-algorithm), and [Interrupt Service Routines].
This narrow feature set makes it a good candidate to start experimenting with the RTOS, but is unlikely to provide much benefit to real-world applications.

An application built on top of this RTOS variant can be split into multiple tasks that interact to deliver an application's functionality.
For example, with an interrupt and a task for sensor reads and another task for user interaction, the sensor interrupt service routine may implement time-critical interrupt processing and then activate the sensor task.
The sensor task may then perform non-critical sensor handling and eventually activate the UI task to indicate a particular sensor reading to the user.

For more complex application behavior, the RTOS provides other variants with more features.


## Startup

The RTOS does not start automatically when a system boots.
Instead, the system is expected to start normally, as per the platform's conventions and C runtime environment.
The C runtime environment invokes the canonical `main` function without any involvement of the RTOS.
This allows the user to customize how the system is initialized before starting the RTOS.

The RTOS provides a [<span class="api">start</span>] API that needs to be called to initialize the RTOS and begin its execution.
The [<span class="api">start</span>] API never returns because it transfers control to the RTOS and its [Tasks].
From the application's point of view, calling the [<span class="api">start</span>] API function makes the first task in the system execute its task function (see [Task Functions]).
From this point onwards, tasks need to use the RTOS APIs (such as [<span class="api">yield</span>]) to trigger task switches.

There is no API to shut down or stop the RTOS once it has started.
The result of marking all tasks as blocked is undefined.

/*| doc_api |*/
## Functions vs. Macros

Some compilers do not support function inlining.
For performance or code space considerations, some APIs described in this chapter are implemented as function-like macros.
This is an implementation detail and the use of all APIs must conform to the formal function definitions provided in this chapter.


## Core API

### <span class="api">block</span>

<div class="codebox">void block(void);</div>

The [<span class="api">block</span>] API causes the current task to enter the blocked state (see [Task States]).
As a consequence, the scheduler causes a context switch to the next runnable task based on the [Scheduling Algorithm].
If the current task is the only runnable task in the system, only an interrupt event is able to make a task runnable again.

### <span class="api">start</span>

<div class="codebox">void start(void);</div>

The [<span class="api">start</span>] API initializes the RTOS and starts the first task in its task function (see [Task Functions]).
That first task is the first task defined in the system configuration file (see [Configuration Reference]), which also specifies the task function.

This function must be called from the system's main function.
This function does not return.

### <span class="api">unblock</span>

<div class="codebox">void unblock(TaskId task_id);</div>

The [<span class="api">unblock</span>] API marks the task with the specified task ID as runnable (see [Task States]).
Calling this API function does not cause an immediate context switch.
If the specified task is already in the runnable state when this API function is called, it has no application-visible effect.

### <span class="api">yield</span>

<div class="codebox">void yield(void);</div>

The [<span class="api">yield</span>] API causes a context switch from the current task to a runnable task in the system.
That task is determined by the scheduler based on the [Scheduling Algorithm].
If the current task is the only runnable task in the system, [<span class="api">yield</span>] returns without a context switch and has no application-visible effect.

### <span class="api">yield_to</span>

<div class="codebox">void yield_to(TaskId task_id);</div>

The [<span class="api">yield_to</span>] API causes a context switch to the task with the specified task ID.
If the specified task ID identifies the current task, [<span class="api">yield_to</span>] returns without a context switch and has no application-visible effect.

The result of yielding to a task that is not runnable is undefined.

/*| doc_configuration |*/
## RTOS Configuration

### `prefix`

In some cases the RTOS APIs may conflict with existing symbol or pre-processor macro names used in a system.
Therefore, the RTOS gives system designers the option to prefix all RTOS APIs to help avoid name-space conflicts.
The prefix must be an all lower-case, legal C identifier.
This is an optional configuration item that defaults to the empty string, i.e., no prefix.

The following examples are based on `prefix` having the value `rtos`.

* functions and variables: lower-case version of `prefix` plus an underscore, so [<span class="api">start</span>] becomes `rtos_start` and [<span class="api">task_current</span>] becomes `rtos_task_current`.

* types: CamelCase version of `prefix`, so [<span class="api">TaskId</span>] becomes `RtosTaskId`.

* constants: upper-case version of `prefix` plus an underscore, so [<span class="api">TASK_ID_ZERO</span>] becomes `RTOS_TASK_ID_ZERO`.

/*| doc_footer |*/
