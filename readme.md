# CS252 Assignment: Scheduling Algorithms
 (Operating System Concepts, 10th Edition, Chapter 5)

---

## Description
Code credits: https://github.com/forestLoop/Learning-EI338
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
Thus, task T1 has priority 4 and a CPU burst of 20 milliseconds, and so forth. It
is assumed that all tasks arrive at the same time, so the scheduler algorithms
do not have to support higher-priority processes preempting processes with
lower priorities. In addition, tasks do not have to be placed into a queue or list
in any particular order.

## Details
The file `driver.c`reads in the schedule of tasks, inserts each task into a linked
list, and invokes the process scheduler by calling the `schedule()` function. The
`schedule()` function executes each task according to the specified scheduling
algorithm. Tasks selected for execution on the CPU are determined by the `pickNextTask()` function and are executed by invoking the `run()` function defined
in the `CPU.c` file. A `Makefile` is used to determine the specific scheduling algorithm that will be invoked by driver . For example, to build the FCFS scheduler,
enter
```
make fcfs
```
and to execute the scheduler using the schedule of tasks `example_tasks.txt` enter
```
./fcfs example_tasks.txt
```

### FCFS
First-come, first-served (FCFS) schedules tasks in the order in which they request the CPU. Therefore, to implement it, we just need to pick the first task in the queue, i.e., the last element in the linked list.

```
Task *pickNextTask() {
    struct node *lastNode = taskList;
    while(lastNode->next) {
        lastNode = lastNode->next;
    }
    return lastNode->task;
}

void schedule() {
    while(taskList) {
        Task *t = pickNextTask();
        run(t, t->burst);
        delete(&taskList, t);
    }
}
```
### SJF
Shortest-job-first (SJF) schedules tasks in order of the length of the tasks’ next CPU burst. To implement it, we iterate over the list and pick the one with the shortest burst.
```
Task *pickNextTask() {
    Task *shortest_job = taskList->task;
    struct node *n = taskList;
    while(n){
        if(n->task->burst <= shortest_job->burst){
            shortest_job = n->task;
        }
        n = n->next;
    }
    return shortest_job;
}

void schedule() {
    while(taskList) {
        Task *t = pickNextTask();
        run(t, t->burst);
        delete(&taskList, t);
    }
}
```

### RR
Round-robin (RR) scheduling, where each task is run for a time quantum (or for the remainder of its CPU burst). To implement it, we maintain a variable `remaining_burst` for each task, which gets reduced each time the task is executed. When `remaining_burst` drops to zero, remove the task from the task list.
```
Task *pickNextTask() {
    Task *ret = next_node->task;
    next_node = (next_node -> next) ? next_node->next : taskList;
    return ret;
}

void schedule() {
    next_node = taskList;
    while(taskList) {
        Task *t = pickNextTask();
        int slice = QUANTUM < t->remaining_burst ? QUANTUM : t->remaining_burst;
        run(t, slice);
        t->remaining_burst -= slice;
        if(!t->remaining_burst) {
            delete(&taskList, t);
        }
    }
}
```
### Priority
Priority scheduling schedules tasks based on priority. Its implementation is quite similar to that of SJF, and the only difference is that we pick the highest-priority task.
```
Task *pickNextTask() {
    Task *highest_priority_job = taskList->task;
    struct node *n = taskList;
    while(n){
        if(n->task->priority >= highest_priority_job->priority){
            highest_priority_job = n->task;
        }
        n = n->next;
    }
    return highest_priority_job;
}

void schedule() {
    while(taskList) {
        Task *t = pickNextTask();
        run(t, t->burst);
        delete(&taskList, t);
    }
}
```
### Priority with RR
Priority with round-robin schedules tasks in order of priority and uses round-robin scheduling for tasks with equal priority. To implement it, rather than only one task list, multiple task lists are used for tasks with different priorities.
```
struct node *taskList[MAX_PRIORITY + 1];

void add(char *name, int priority, int burst) {
    Task *t = malloc(sizeof(Task));
    // allocate memory and then copy the name
    t->name = malloc(sizeof(char) * (strlen(name) + 1));
    strcpy(t->name, name);
    // priority and burst
    t->priority = priority;
    t->burst = t->remaining_burst = burst;
    // insert into task list
    insert(&taskList[priority], t);
}
```
Then when scheduling, perform the RR algorithm in the task list that has the highest priority and is not empty.
```
Task *pickNextTask(struct node *tl) {
    Task *ret = next_node->task;
    next_node = (next_node -> next) ? next_node->next : tl;
    return ret;
}

// invoke the scheduler
void schedule() {
    // from higher priority to lower priority
    for(size_t p = MAX_PRIORITY; p >= MIN_PRIORITY; --p) {
        next_node = taskList[p];
        while(taskList[p]) {
            Task *t = pickNextTask(taskList[p]);
            int slice = QUANTUM < t->remaining_burst ? QUANTUM : t->remaining_burst;
            run(t, slice);
            t->remaining_burst -= slice;
            if(!t->remaining_burst) {
                delete(&taskList[p], t);
            }
        }
    }
}
```
## Result
With the example tasks:
```
$ cat example_tasks.txt
T1, 4, 20
T2, 3, 25
T3, 3, 25
T4, 5, 15
T5, 5, 20
T6, 1, 10
T7, 3, 30
T8, 10, 25
```
The output of each algorithm is:
```
$ ./fcfs example_tasks.txt
Running task = [T1], priority = [4], burst = [20] for 20 units.
Running task = [T2], priority = [3], burst = [25] for 25 units.
Running task = [T3], priority = [3], burst = [25] for 25 units.
Running task = [T4], priority = [5], burst = [15] for 15 units.
Running task = [T5], priority = [5], burst = [20] for 20 units.
Running task = [T6], priority = [1], burst = [10] for 10 units.
Running task = [T7], priority = [3], burst = [30] for 30 units.
Running task = [T8], priority = [10], burst = [25] for 25 units.

$ ./sjf example_tasks.txt
Running task = [T6], priority = [1], burst = [10] for 10 units.
Running task = [T4], priority = [5], burst = [15] for 15 units.
Running task = [T1], priority = [4], burst = [20] for 20 units.
Running task = [T5], priority = [5], burst = [20] for 20 units.
Running task = [T2], priority = [3], burst = [25] for 25 units.
Running task = [T3], priority = [3], burst = [25] for 25 units.
Running task = [T8], priority = [10], burst = [25] for 25 units.
Running task = [T7], priority = [3], burst = [30] for 30 units.

$ ./priority example_tasks.txt
Running task = [T8], priority = [10], burst = [25] for 25 units.
Running task = [T4], priority = [5], burst = [15] for 15 units.
Running task = [T5], priority = [5], burst = [20] for 20 units.
Running task = [T1], priority = [4], burst = [20] for 20 units.
Running task = [T2], priority = [3], burst = [25] for 25 units.
Running task = [T3], priority = [3], burst = [25] for 25 units.
Running task = [T7], priority = [3], burst = [30] for 30 units.
Running task = [T6], priority = [1], burst = [10] for 10 units.

$ ./rr example_tasks.txt
Running task = [T8], priority = [10], burst = [25] for 10 units.
Running task = [T7], priority = [3], burst = [30] for 10 units.
Running task = [T6], priority = [1], burst = [10] for 10 units.
Running task = [T5], priority = [5], burst = [20] for 10 units.
Running task = [T4], priority = [5], burst = [15] for 10 units.
Running task = [T3], priority = [3], burst = [25] for 10 units.
Running task = [T2], priority = [3], burst = [25] for 10 units.
Running task = [T1], priority = [4], burst = [20] for 10 units.
Running task = [T8], priority = [10], burst = [25] for 10 units.
Running task = [T7], priority = [3], burst = [30] for 10 units.
Running task = [T5], priority = [5], burst = [20] for 10 units.
Running task = [T4], priority = [5], burst = [15] for 5 units.
Running task = [T3], priority = [3], burst = [25] for 10 units.
Running task = [T2], priority = [3], burst = [25] for 10 units.
Running task = [T1], priority = [4], burst = [20] for 10 units.
Running task = [T8], priority = [10], burst = [25] for 5 units.
Running task = [T7], priority = [3], burst = [30] for 10 units.
Running task = [T3], priority = [3], burst = [25] for 5 units.
Running task = [T2], priority = [3], burst = [25] for 5 units.

$ ./priority_rr example_tasks.txt
Running task = [T8], priority = [10], burst = [25] for 10 units.
Running task = [T8], priority = [10], burst = [25] for 10 units.
Running task = [T8], priority = [10], burst = [25] for 5 units.
Running task = [T5], priority = [5], burst = [20] for 10 units.
Running task = [T4], priority = [5], burst = [15] for 10 units.
Running task = [T5], priority = [5], burst = [20] for 10 units.
Running task = [T4], priority = [5], burst = [15] for 5 units.
Running task = [T1], priority = [4], burst = [20] for 10 units.
Running task = [T1], priority = [4], burst = [20] for 10 units.
Running task = [T7], priority = [3], burst = [30] for 10 units.
Running task = [T3], priority = [3], burst = [25] for 10 units.
Running task = [T2], priority = [3], burst = [25] for 10 units.
Running task = [T7], priority = [3], burst = [30] for 10 units.
Running task = [T3], priority = [3], burst = [25] for 10 units.
Running task = [T2], priority = [3], burst = [25] for 10 units.
Running task = [T7], priority = [3], burst = [30] for 10 units.
Running task = [T3], priority = [3], burst = [25] for 5 units.
Running task = [T2], priority = [3], burst = [25] for 5 units.
Running task = [T6], priority = [1], burst = [10] for 10 units.
```

