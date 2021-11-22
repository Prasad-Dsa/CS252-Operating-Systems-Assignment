# CS252 Assignment: Scheduling Algorithms
 (Operating System Concepts, 10th Edition, Chapter 5)

---

## Description
This project involves implementing several different process scheduling algorithms. The scheduler will be assigned a predefined set of tasks and will
schedule the tasks based on the selected scheduling algorithm. Each task is
assigned a priority and CPU burst. The following scheduling algorithms are implemented

* First-come, first-served ( FCFS ), which schedules tasks in the order in which they request the CPU
* Shortest-job-first ( SJF ), which schedules tasks in order of the length of the tasks’ next CPU burst
* Round-robin ( RR ) scheduling, where each task is run for a time quantum (or for the remainder of its CPU burst)
* Priority scheduling, which schedules tasks based on priority
* Priority with round-robin, which schedules tasks in order of priority and
uses round-robin scheduling for tasks with equal priority

Priorities range from 1 to 10, where a higher numeric value indicates a higher
relative priority. For round-robin scheduling, the length of a time quantum is
10 milliseconds.

## Implementation 
The project is implemented in C and the supporting program files were provided in the source code downloaded from the [companion website](https://codex.cs.yale.edu/avi/os-book/OS10/index.html ) for the text. These supporting files read in the schedule of tasks, insert the tasks into a list, and invoke the scheduler.
The schedule of tasks has the form **[task name], [priority], [CPU burst]**, with the following example format:
```
T1, 4, 20
T2, 2, 25
T3, 3, 25
T4, 3, 15
T5, 10, 10

```
