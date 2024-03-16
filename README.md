# Processes and Threads 

## Process

In computer, there are programs that basically consists a set of instructions that are written to handle specific tasks. 

These instructions/codes are stored in the disk or memory and they need to be executed by the processor/CPU to give us what we want. 

When these set of instructions are executed, we can call the process of the execution of these instructions as **process**. Process is an abstraction of running program. 

And sometimes a process may create new process(es) to be able to finish its tasks. In those cases, we can represent the processes in a process tree:

```
      A
     / \
    /   \
   B     C
  / \
 D   F
```

In the process tree above, the process A created two child processes: process B and process C. And process B created two child processes: process D and process F.

In addition, a process typically includes information like:

- variables of the program
- code of the program
- program counter (it is a hardware resource that keeps track off the next instruction to be executed)
- registers (this is another hardware resource that can hold any kind of data that is necessary for the execution)
- process state (this state can be "Created", "Running", "Ready", etc. and it represents the current state of the process)

Okay these are good to know but where are these information stored and how the process can access to these information ?

These information are basically stored in physical memory. Each of these information has a physical address. Hardware components such as CPU, for example, use these physical addresses directly when fetching/storing data in the physical memory. But processes don't load/store information through physical addresses. For processes to access to the information in memory, we first virtualize the addresses in the memory, in other words, we translates the physical addresses into virtual addresses. These virtual addresses are represented as strings and they basically point to the physical addresses. Processes interact with these virtual addresses directly and the virtual address the process wants to access is translated into physical address and the information that is located at that physical address in the memory is retrieved to the process.  

And we call a range of these virtual addresses **address space**. 

### Address Space 

Address spaces can be generally either flat or segmented. In flat address space, we store the virtual addresses consecutively just like how we store data in array. In segmented address space, we group the virtual addresses and divide them into segments based on their groups, and store these virtual addresses in segments. When we make groups in the address space, the operationg system can effectively treat these segments as a region that has the same properties. 

Segmented address space also provides protection of each segment. We can assign different access permissions (e.g. read only, reaad write, execute only) to different segments and this can prevent unauthorized access to the critical memory regions. 

Also one note is that if we have two same applications, their address space will be exactly the same. 

Okay we talked about the processes, how do they look like, what kind of components they include, how they access to the information in the memory. Let's now talk about the lifecycle of a process from process creation to process implementation and process termination.

So, maybe the first question we should ask is: when processes are created ? 

## Process Creation 

- When the system is initialized.
      - When a computer initializes the operationg system and loads it to the memory, initialize hardware devices, etc. And during these times, new processes can be created.
      - During system initialization, some processes are created to run in the foreground to allow the users to interact with them. 
      - And similarly during system initialization, some processes are created to run in the background to perform tasks (e.g. managing resources, without interacting with the user. 
- When processes are running, these processes typically create new processes as well.
- The user can execute some codes or open applications and these actions create processes as well.
- The operationg system can create processes that will to provide services.
- When the user enters into a system interactively (e.g through terminal, remote shell, or a graphical login screen), a new process is created to be able to handle the user's session and allow the user to interact with the system.

Now the question is: how the processes are implemented ? 

## Process Implementation

Once the processes are created, there might be too many of them. And in each one of these processes, there might be too many information that we have to deal with. Therefore, to be able to manage all the processes properly, we can try to store the important information about these processes (e.g. state of the process, its priority, program counter, pointers to memory, IO status information, etc.) in a data structure, and manage the processes through this data structure more easily. 

We call this data structure **Process Control Block (PCB)**. The process control block is basically created and managed by the operating system. This means that there is no reference other than the process ID to access to that object from the user space. 

All the information in the process control block must be saved when the process is switched from one state to another. When the process transitions from one state to another, the operating system must update the information in the process control block. And when the processors switches from executing one process to another process, the state of the current process and the contents of its registers must be saved. This ensures that when the process continues to be executed later, it can continue execution from where it stopped without losing any important data or progress.

Since each process will have a distinct process control block, to be able to access these process control blocks properly, we can assign a unique ID, which we call **Process ID (PID)**, to each process, and store the process ID and process control block in a new data structure. And we call this data structure **Process Table**. 

When we pass the process ID, the kernel will basically look that process ID in the process table and verify that you have the rights to access the process control block.

```
---------
Memory    -------------------> Memory Table
---------
Devices   -------------------> IO Table
---------
Files     -------------------> File Table
---------
Processes -------------------> Process Table
----------                     -------------    
                               PID | PCB
                                1  |  x 
                                2  |  x 
                               ... | ...
                                n  |  x ⁻⁻⁻⁻⁻⁻|
                                              |
                                              |
                                              V
                                          ---------------
                                          Process State
                                          ---------------
                                          Priority
                                          ---------------
                                          Program Counter
                                          ---------------
                                          Memory Pointers
                                          ---------------
                                          IO Status Info
                                          ---------------
                                          ...
                                          ---------------

```

One of the benefits of using process control block is, for example, by storing the state information of the process in the process control block, it becomes possible to interrupt a running process and then resume its execution later. So, it allows us to support/manage multiple processes.

And the operating system maintains multiple tables (e.g. Memory Table, IO Table, File Table, Process Table) to be able to manage the resources when the processes are being executed. 

**!Slides between 17 and 28 are skipped!**

Okay, we have mentioned that the state of the process is stored in the process control block. But what do we mean by a "state" ? 

## Process State

State basically means the current activity of the process. It is specified with strings and stored in the process control block in that way.

A process can have 5 different states depending on its current activity:

- When the process is created, its state will be **New**.
- When the process are being executed, its state becomes **Running**.
- Sometimes processes might wait for an event and expereince delay because of this. When this happens, the state of the process becomes **Waiting/Blocked**.
- And when the process becomes ready to be executed by the CPU, its state becomes **Ready**.
- Lastly, when the process is terminated, the state becomes **Terminated**.

As can be seen from above, a state can go through different stages until it is terminated. The representation of the all the stages a process can go through is called process state model. 

## Process State Model 

There can be several possible state models depending on our implementation. 


If it is okay for us to assign two states (**Not Running**, or **Running**) to each process, for example, we can implement 2-State Model

### Two-State Model

```
      |-----------|
      | Dispatch* |
      |           | 
      |           V
Not Running     Running 
      Ʌ           |
      |           | 
      |   Pause   |
      |-----------|

```

**Dispatch is the action of assigning the process to the running state. When a process is dispatched, this means that it is selected by the CPU to be executed.*

In a system that we developed according to the 2-State Model, a process can be assigned either **Running** or **Not Running** state.

But the problem is that if a process is not in the running state, there might be many reasons why it is not in the running state: maybe it is waiting an IO operation, or maybe it is ready to and waiting for the CPU to be executed. In two state model, we ignore all of these possibilities and simply assign not running state if the process is not running. 

And two state model is efficient only if all the processes that are in not running state are ready for execution. But this may not be the case sometimes. Sometimes, a process that is in not running state might be waiting for some IO operation or another event. In those cases, the dispatcher cannot simply select the process from the front of the queue. It would have to scan the queue to search for the process that can be executed at that moment. The best way to solve this problem is to split the not running state into separate groups. 

So, we can try to introduce these factors in a new state model. 

### Three-State Model

```
             |--------------|     
             V              |
     |----Running----|      |
     |               |      |
     |               |      |  
     V               V      |
  Blocked ------> Ready -----    
```

When we use three-state model, now the process can be moved to blocked state when it starts waiting for an event such as an IO operation. And when that event happens, this means that the process is ready to be executed again so it is moved from the blocked state into the ready state. 

And if another process that has a higher priority than the currently running process arrives, or if the currently running process has run long enough, we can move the currently running process from running state into the ready state. Then we can execute another process that is waiting to be executed by moving it from ready state into running state. 

Okay these are good but we still ignore the fact that some process has to be created in the first step. Not having a state for the process creation specifically may cause the system to not being able to allocate resources to the newly created processes until they are ready to be executed. Similarly, not having a state for the process termination specifically may cause the system to not being able to manage the resources efficiently after a process finishes its execution. 

Thereofre, it may be better to develop another state model that can take these details into account. 

## Five State Model 

                     Timeout
              -----------------------    
              |                     | 
              |                     |
     Admit    V     Dispatch        |     Release
New -------> Ready -----------> Running ----------> Exit
              Ʌ                 /
              |                / 
              |               /
              |              /    
              |             /
              |            /
              |           /
              |          /
              |         /
              |        /
              |       /
              |      /
              |     /
              |    /
              |   /
              |  V
           Blocked
```



## Process Termination 

- When a process completes its execution without any problem, it exits **voluntarily**. We can call this **normal exit**
- When a process encounters an error/exception during its execution, it exits **voluntarily**. We can call this **error exit**.
- When a process encounters a severe error that will prevent the operating system to operate safely (e.g. by potentially damaging the system or causing data loss), the operating system terminates the process immediately. When this kind of severe error happens, the process is not given a chance to take any action, that's why the exit is **involuntary** and the user may end up with losing the changes he made before the error. We can call this **fatal error**.
- Sometimes, a prcoess may explicitly terminate another process. One example of this might be terminating a process that has not been responding for a very long time. Because the process is not given any chance to take action, this is an **involuntary** termination as well.
- When a process has run longer than expected, they may be terminated as well. We can call this **time limit exceeded error**






## Thread

And in these processes, there might be smaller units of execution as well and we call these units **threads**. If there are multiple threads in a process, these threads need to work together and share the same address space and many other resources. 

Okay we have defined the processes and threads but what is exactly processor/CPU ? 

CPU is basically a chip that is made up from electronic components that are called transistors. Its primary job is execute the instructions and perform operations on the data. 

Imagine that you ran a program in your computer. This program includes some set of instructions. And these instructions are sent to the CPU. Then CPU follows these instructions step by step to do the required tasks specified in the program. And while doing that, it can retrieve data from the memory, manipulate that data according to the instructions, and then store the results back in the memory or maybe send them to monitor, or printer for instance. 

A CPU generally has one or more cores. These cores are physical processing units that are responsible from executing instructions and performing computations in general. 

And we can divide the core into three groups: 

- Memory: In core, a memory unit is built to store & transfer information fastly/efficiently. This memory unit consists of registers and cache.
- Control Unit: These units fetches the instructions that are represented as bits from the memory unit, and translates those instructions into electricity or maybe light so that they can be sent to other parts of the computer as signal. 
- Arithmetic Logic Unit: This unit includes electronic circuits. And these circuits are responsible from arithmetic and logic operations. 

The operating system can schedule a process or thread onto each hardware thread. And a hardware thread consists of 

- Program Counter: Keeps track of the next instruction to be executed
- Registers: These are used to store information about the thread (e.g. status information, and addresses of the thread)
- Cache Resources: Each hardware thread owns some part of core's cache to store its own data and instructions.
- Execution Resources: These are the resources it needs for executing the instructions.

Note that the instructions and registers are not virtualized by their nature. They are the real physical instructions and registers. That's why a processor, or core, or hardware thread can only run one process/thread (these can also be called as unit of execution) at a time. 

And because processor can only run one process/thread at a time, it is switched among multiple processors/threads to improve efficiency. This is called **context switching**. As a result of this switching, the units of executon will all look like progressing but at a slower speed. 

The benefit of context switching is that when a process performs IO operation, instead of forcing the CPU to wait for that process to finish its IO operation, we can just direct that CPU to another process during that time. And as result, we can finish more tasks. 

# Reminder 

**Concurrency**: Executing multiple instructions in an interleaved manner. 

**Parallelism**: Distributing multiple instructions into different resources and running them at the same time in parallel. 

# Inter-Process Communication

In computer, there are programs that basically consists a set of instructions that are written to handle specific tasks. 

These instructions/codes are stored in the disk or memory and they need to be executed by the CPU to give us what we want. 

When these set of instructions are executed, we can call the process of the execution of these instructions as **process**.

Sometimes there might be multiple processes that need to work with each other and share resources to be able to finish their executions successfully and efficiently.

In addition, before we handle the communication between the processes, it is important to stop and think about the 3 key points:

1) How these processes will pass information to each others ? 
2) Ensuring that two processes don't interfere with each other.
3) Ordering the processes properly if there is a dependency: *If process A produces data and process B prints these data, process B has to wait until process A produce data to print.*

We have mentioned that sometimes multiple processes can share the same resource. And one of the resources they may share might be storage (e.g. main memory, shared file). 

Print spooler is a perfect example to this. A print stooler is basically a program/software that puts the print jobs that are sent from an application to a spooler directory (which can be seen as temporary data storage). When a process wants to print a file, for example, it enters the name of the file into the spooler directory which is where the information about the file is stored. And another separate process (printer daemon) periodically checks if there is any file in spooler directory to be printed. If there is a file to be printed, it's printed and then it's file name is removed from the spooler directory.

With this procedure, the print jobs do not need to be sent to the printer directly. The benefit of applying this procedure is that sending print jobs to the printer directly could cause two print jobs to interfere with each other if multiple users would try to use the printer simultaneously. And by storing the print jobs in a temporary place this way, extracting a print job from that temporary place, and giving the resource to it whenever the resource becomes available, we can prevent this kind of interference between processes. 

Also, if we wouldn't use spooler directory, processes wouldn't be aware of each other at all.

Okay, using a shared storage like this is cool but it actually brings an important issue: **race condition**. 

**Race Condition**: It is a situation in which two or more processes attempts to do their operatings at the same time. When two or more processes read from the same resource and also write into the same resource at the same time, a race condition occurs. Because processes are kind of racing with each other to do their tasks.

# Intra-Process Communication

Until this point, we have mentioned about the processes as well as the inter-process communication. And one thing to note is that in processes, there might be multiple smaller units of execution and we call these units **threads**. If there are multiple threads in a process, these threads work together and share the same address space and other resources. Therefore, there should be some kind of communication between them as well. 

And before we handle the communication between the threads, the 3 key points that need to be considered and that we emphasized in inter-process communication are valid in here as well:

1) How these threads will pass information to each others ? 
2) Ensuring that two threads don't interfere with each other.
3) Ordering the threads properly if there is a dependency.

So, the ability of the multiple processes/threads to access/modify the shared resources simultaneously is cool but it is important to note that this situation may (and will) end up with many problems if it is not handled properly.

For instance, when multiple processes or threads are accessing/sharing/modifying the shared data, a specific sequence of operations must be performed in order to ensure data consistency and integrity. This sequence of operations is what we call **Read-Modify-Write Cycle**. The overall goal is for the code to access to the consistent and same view of the data. 

### Read-Modify-Write Cycles

- **Read**: Reading the current value of a memory location from the memory is called **read**.
- **Modify**: Some operation is performed on the value that was read from a memory location and this is called **modify**. 
- **Write**: The modified value is written back to the same memory location from where the value was read in the first step.

One note is that some parts of this cycle should be atomic. Otherwise, we may end up with race conditions and data inconsistency. 

# Preventing Race Conditions

One simple way to prevent race conditions is to prevent more than one process or thread from reading and modifying the shared resource at the same time. In other words, if one process or thread accesses to a shared resource, all the other processes or threads should be excluded from doing the same thing and we call this **mutual exclusion**.

And the section of the program/codes in which shared resources are accessed is called **critical region**. 

If we can develop a system in which two or more processes would never be within their critical regions at the same time, we can prevent race conditions. 

But the issue is that if we develop only this rule (in other words if we only try to prevent multiple processes from being in their critical regions at the same time) we can prevent race conditions but this also prevents parallel processes from cooperating correctly and to using shared resources efficiently. 

For example, preventing multiple processes to be within their critical regions at the same time does not prevent a process that is **running outside of its critical region** to **block another process** that is running in its **critical region**. This kind of block is unnecessary because it may cause delays and does not bring any benefit. Therefore, we should add additional rules to make the system more efficient and scalable: 

1) We shouldn't take the speed or the number of CPUs into account. (Not relying on assumptions about the speed of processors or the number of them helps us to develop methods that can be used in different systems)
2) If there is a process running outside its critical region, it shouldn't block any process. (This will ensure that processes do not block each other unnecessarily when they are **not** accessing shared resources)
3) There should be no process that is waiting to enter its critical region forever. (This will prevent starvation)

in addition to the first condition that we defined previously 

4) Two processes should not be in their critical regions at the same time.

Now let's try to find a method that can meet all of these four rules. 

# Mutual Exclusion with Busy Waiting

Let's say that there is a process that is currently in it's critical region. And let's call it process A. One of the conditions of preventing race conditions was to prevent all other processes from entering their critical regions if process A is in it's critical region. 

The simplest way to do this is using **interrupts**. 

Interrupt is basically a signal/notification that is sent by hardware or software to the CPU. It indicates that an event happened that requires attention. The interrupts can be generated by external devices such as keyboard, mouse, etc. or internal system components such as disk controller, timer, etc. One way to represent interrupts is using a vector of numerical values associated with a specific interrupt type. 

When an interrupt happens, the information of the currently executed process (e.g. registers, program counter, etc.) is saved and the control is transferred to the interrupt handler, which is a software that is responsible from handling interrupts. And the intrrupt handler performs the necessary actions. 

So, if process A disables all the interrupts just after it enters it's critical region and then re-enable these interrupts just before leaving the critical region, only one process can be in its critical region and therefore process A can access/modify the shared memory exclusively without the fear of intervention. 

But using only this approach is not the best option because it is not a good practice to give the process in the user space the ability to turn on/off the interrupts since the process may forget to turn on the interrupt after leaving its critical region and that would cause many problems. In addition, disabling interrupts from the user space will affect only the CPU that executed the process A. In this case, we actually don't prevent the processes **in other CPUs** intervening the process A and entering their own critical regions while the process A is in its critical region. 

So, using interrupts is not the best way to provide mutual exclusion. Let's look at another way to provide mutual exclusion.

**Lock Variables:** Another way to apply mutual exclusion is using **lock variables**. Instead of interrupts, we can use a variable that takes a value of 0 or 1. And we call this lock variable. 

For example, if the process A enter its critical region, it can set this lock variable to 1, which basically means that the critical section is occupied at this moment. And if other processes try to enter their critical regions after the lock variable is set to 1 by the process A, they will see that lock variable is 1 and they will have to wait until the lock variable is set to 0 by the process A (which means that process A stopped accessing to the shared resource, left its critical region and one process can now enter its own critical region and access to the shared resource).

But the thing is: when we use just lock variables, we might encounter with the same problem that we encounter with print spooler: what if two processes see the lock variable as 0 and enter their critical regions simultaneously ? 

Now, let's say that there are two processes: 

```
Process A:

while (TRUE) {
  while (turn != 0) {} // Wait until turn = 0
  critical_region();   // If turn = 0, enter to critical region.
  turn = 1;        
  noncritical_region();
}
```

```
Process B:

while (TRUE) {
  while (turn != 1) {} // Wait until turn = 1
  critical_region();   // If turn = 1, enter to critical region.
  turn = 0;
  noncritical_region();
}
```

In the example above, when process A leaves the critical region, it sets the turn variable to 1 to allow the process B to enter its critical region. Suppose that process B finishes its critical region quickly and set the turn variable to 0. At that moment, both process A and process B will be in their non critical regions. 

Later, if process A executes its loop again, enters its critical region, sets the turn variable to 1, and then enters its non critical region, both process A and B will be in their non critical regions and turn variable will be equal to 1. 

At that time, if process A finishes its non critical region and goes back to the top of it's loop, it won't be permitted to enter its critical region because turn = 1. 

That's why, this kind of procedure is not a good solution because it violates the 2nd condition that we defined previously: 

1) We shouldn't take the speed or the number of CPUs into account. (Not relying on assumptions about the speed of processors or the number of them helps us to develop a system that can be used in different configurations)
2) **If there is a process running outside its critical region, it shouldn't block any process. (This will ensure that processes do not block each other unnecessarily when they are not accessing shared resources)**
3) There should be no process that is waiting to enter its critical region forever. (This will prevent starvation)
4) Two processes should not be in their critical regions at the same time.

So we can conclude that using lock variables solely is not the best method to achieve mutual exclusion. So, let's take a look at other methods.

## Peterson's Solution for Achieving Mutual Exclusion

```
#define FALSE 0
#define TRUE 1
#define N 2

int turn;          
int interested[N];
  // interested = [. , .]

void enter_region(int process) {
  int other;

  other = 1 - process;
    // Other process. If the current process is 0, other process will be 1.
    // If current process is 1, other process will be 0.

  turn = process;
    // Which process' turn to enter its critical region ?

  while (turn == process && interested[other] == TRUE) {}
    // If its current process turn and and the other process is in its critical region,
    // keep waiting until the other process leaves its critical region,
    // in other words until interested[other] == FALSE.
}

void leave_region(int process) {
  interested[process] = FALSE;  
}

```

The other methods that are used to do ensure mutual exclusion are called **Test and Set Lock** and **XCHG**.

## TSL (Test and Set Lock)

```
enter_region:

  TSL REGISTER, LOCK
    | Copies the value in the lock into the register and sets the lock to 1.

  CMP REGISTER, #0
    | Compares the value in the register with 0.

  JNE enter_region
    | If the value in the register is not 0,
    | this means that critical region is occupied.
    | So keep waiting until the value in the register is set to 0, in other words,
    | until the critical region becomes vacant.
    | If the value in the register is 0,
    | this means that the critical region is vacant.
    | In that case, jump into the RET.

  RET
    | Returns, allowing the process to acquire the lock and enter its critical region.

leave_region:

  MOVE_LOCK, #0
    | Releases the lock, and stores the value of 0 in the lock variable.

  RET
    | Returns, allowing other threads to attempt to acquire the lock.
```  
  
## XCHG

```
enter_region:

  MOVE REGISTER, #1
    | Load the value of 1 into the register.

  XCHG REGISTER, LOCK
    | Swap the value of the register with the lock.
    | Let's say that the current value of the lock is 0.
    | This means that the lock is available and there is no process
    | that is in its critical region right now. 
    | So, the processor reads this value in the lock.
    | And then it writes the current value of the register (1) to lock,
    | and writes the value of the lock (0) into the register simultaneously.
    | After this, the register is now equal to 0 (which means the lock is acquired)
    | and the lock is now equal to 1 (which represents a locked state)
    | So, by storing the initial value of the lock (0) in the register
    | and using the swap operation,
    | we can check the initial value of the lock and set the lock variable
    | to a desired state in an atomic operation.
    | And this atomic operation prevents the situation in which
    | one process (e.g. process A) checks the lock variable and
    | another process (e.g. process B) acquires the lock
    | before the process A sets the lock variable.

  CMP REGISTER, #0
    | Compares the value in the register with 0.

  JNE enter_region
    | If the value in the register is not 0,
    | this means that critical region is occupied.
    | So keep waiting until the value in the register is set to 0, in other words,
    | until the critical region becomes vacant.
    | If the value in the register is 0,
    | this means that the critical region is vacant.
    | In that case, jump into the RET.
    
  RET
    | Returns, allowing the process to acquire the lock and enter its critical region.

leave_region:

  MOVE_LOCK, #0
    | Releases the lock, and stores the value of 0 in the lock variable.

  RET
    | Returns, allowing other threads to attempt to acquire the lock.
```  

So, the solutions we tried until now, **Peterson's solution**, **TSL**, and **XCHG**, to achieve mutual exclusion were correct. But the thing is they had some issues. 

For instance, they had the defect of **busy waiting**. In other words, when a process wanted to enter its critical region, it checked to see whether it is allowed to enter. If this was not the case, the process just waited until the its entry was allowed. And the problem is that this waiting process wastes the CPU time. That's why it should be avoided.

In addition, suppose  that there are two processes: process A and process B and the priority of process A is higher than the priority of process B. When we use methods like **Peterson's solution**, **TSL**, or **XCHG**, there is a chance that process A can be prevented from entering its critical region when process B is in its critical region and holding the lock variable. And this is called **priority inversion**. 

So in the next steps, we will try to find a way to eliminate the busy waiting and priority inversion problems as much as possible. 

Let's try to do the lock implementation by reducing the busy waiting. 

# Lock Implementation with Semi-Busy Waiting 

```
mutex_lock: 
  TSL REGISTER, MUTEX
    | Copies the value in the mutex (mutual exclusion) lock into the register and sets the mutex lock to 1.

  CMP REGISTER, #0
    | Compares the value in the register with 0.

  JZE ok
    | If the value in the register is 0, this means that critical region is vacant.
    | In that case, jump into RET.
    | If the value of the register is not 0,
    | this means that the critical region is occupied.
    | In that case, go to the next code.

  CALL thread_yield
    | Instead of checking the status of the lock repeatedly (busy-waiting),
    | and wasting CPU time, we can allow another process/thread to run.
    | So, if critical region is currently occupied,
    | schedule another process/thread and go to the next code.

  JMP mutex_lock
    | Runs the mutex_lock again with a new process/thread.

  RET
    | Returns, allowing the process to acquire the lock and enter its critical region.

mutex_unlock:

  MOVE_MUTEX, #0
    | Releases the lock, and stores the value of 0 in the mutex lock variable.

  RET
    | Returns, allowing other threads to attempt to acquire the lock.
```

Okay with this new method, we now reduced the **busy waiting** by scheduling another process/thread instead when the lock is not available for a process.

Note that when a process/thread tries to acquire a lock, if the lock is not available for that process, we call this **lock contention**

# Lock Contention

Lock contention depends on 

1) the frequency of attempts to acquire the lock      *(as the number of attempts to acquire the lock increases, the probability of lock contention increases)*
2) the amount of time a process/thread holds the lock *(if a process hold the lock very short time, the probability of lock contention decreases)*
3) number of processes/threads that acquired the lock *(as the number of processes that want to acquire the lock increases, the probability of lock contention increases)*

If the lock contention is low, this means that the length of time a process/thread spends to wait for the lock variable to be available is low. In other words, processes/threads don't wait too much and in that kind of scenario, TSL might be a good solution.

Until now, we used lock variable to ensure that only one process can occupy the critical region at a time. We can do this with methods other than a simple lock variable as well. 

Let's explain this in a new problem that is called **producer-consumer problem**.

# Producer-Consumer Problem

In producer-consumer problem, there are two processes. And they share a common buffer (temporary data storage) which it's size is fixed. 

One of these two processes produces information and puts that information into this buffer. We can call this process **producer**. 

Another process takes this information from the buffer and uses it. We can call this process **consumer**. 

Now imagine that the buffer is full and there is no empty space. In that case, if the producer wants to put a new information into this buffer, that would cause a problem. One solution for the producer might be being forced to sleep until being awakened. And when the consumer removes one or more items from the buffer and buffer has empty slot(s), it can wake up the producer as well so that it can put its information into the buffer. 

Similarly, if the buffer is completely empty, and if the consumer wants to remove an item from that buffer, that's a problem too since there is nothing to remove. And again one solution for the consumer might be being forced to sleep until being awakened. And when the producer puts information to the buffer and buffer has a non-empty slot, it can wake up the consumer as well so that it can extract the information from the buffer and use (consume) it. 

But there are some issues in these solution.

If we look at from the producer's perspective: 

1) How can the producer know if a slot is free or not ?
2) If the slot is not free, how can the producer know if it/when it will become free ?

And if we look at from the consumer's perspective: 

1) How can the consumer know if an information is available in the buffer ?
2) If there is nothing available to consume in the buffer, how can the consumer know if/when an information will become available ?

To answer these questions, we should first introduce a new concept named **Pipe**. 

## Pipe 

Pipe is basically an inter-process communication mechanism that provides temporary storage between processes. A pipe is basically implemented with a buffer data structure in the kernel. And in the produer-consumer problem, the output of the producer is passed to this buffer and the consumer takes the information from the same buffer. 

When the buffer is full, pipe blocks the producer to put a new information to it. Similarly, when the buffer is empty, it blocks the consumer to prevent it to attempt consuming an information from an empty buffer. 

And when a space becomes available in the buffer, the pipe unblocks the producer. Similarly, when some data becomes available in the buffer, the pipe unblocks the consumer. 

So, in summary pipe is a unidirectional data channel that can be used for inter-process communication. You can create pipes between two processes or threads to ensure smooth data flow between processes/threads. 

So, now let's turn back to the producer-consumer problem. You can see a smiple application of this problem in below:

```
#define N 100
  // The length of buffer

int count = 0;
  // Number of items in the buffer

void producer(void) {

  int item = produce_item();
    // Produce the next item

  if (count == N) {
    sleep(producer);
      // If the buffer is full, don't put the item into the buffer
      // and sleep until being awakened by the consumer.
  }

  insert_item();
    // Now put the item into the buffer

  count++;
    // Increase the number of items in the buffer by 1.

  if (count == 1) {
    wakeup(consumer);
      // If an item is available in the buffer,
      // wake up the consumer and let it know about this
  }
}

void consumer(void) {
  if (count == 0) {      
    sleep(consumer);
      // If the buffer is empty, don't attempt to extract item from the buffer and
      // sleep until being awakened.
  }

  int item = remove_item();
    // Now extract an item from the buffer

  count--;
    // Decrease the number of items in the buffer by 1

  if (count == N-1) {

    wakeup(producer);
      // If the buffer was full before extracting an item,
      // this means that an empty slot became available in the buffer.
      // In that case, wake up the producer and let it know about this so that
      // it can put a new item into the buffer if it is waiting. 
  }

  consume_item(item);
}
    
```

Okay, we explained the producer-consumer problem, brought a new method (sleep() and wakeup() operations) to prevent multiple processes to access to their critical regions at the same time. But there is still some issue. 

In the producer-consumer problem above, the access to the count variable is not constrained. And as a result of this, we may encounter with a race condition. 

## Fatal Race Condition

For instance, let's assume that the buffer is empty and the consumer reads the count variable as 0. At that moment, consumer starts sleeping. Then let's say that a scheduler starts running the producer. Producer produces an item and inserts it into the buffer, increments the value of the count from 0 to 1, and lastly **wakes up the consumer** which is technically **not sleeping**. 

In another case, let's say that count value equals to 1 and consumer begins running its codes. It will first remove an item from the buffer since count is not 0. Then it will decrease the value of count from 1 to 0, and consume the item without waking up the producer since the buffer was not full before extracting the item from it. 

So, after consuming the item, the consumer will start the loop again. At that moment, count value equals to 0. That's why the consumer will sleep. Sooner or later, the producer will produce items and fill the whole buffer. At that moment, consumer has already been sleeping and since the buffer is full, the producer will sleep as well and they will both sleep forever. 

The main issue in these two scenarios is that the wakeup() call that is sent to a process that is not sleeping is lost. 

So we can develop a new mechanism in that allows multiple processes to access to the shared resources simultaneously on the basis shared resource. We call this mechanism **semaphores**.

# Semaphores 

A semaphore is an integer value just like mutex. But the only way to access it is through two separate operations that are called **wait()** and **signal()**.

- **wait():**  Decrements the value of the semaphore by 1 which means that the resource is acquired. Before this decrementation, if the value of the semaphore was 0 or negative, this means that resource(s) controlled by the semaphore was/were already being used. And after wait() operation decreases the value of the semaphore by 1, the value of the semaphore will be lower than 0. In that case, we add the current thread into the wait queue, because of the lack of available resources, block it (since it cannot access to the shared resource at this moment), and schedule another thread and run the wait() operation again to avoid spinning and wasting CPU time.

- **signal():** Increments the value of the semaphore by 1 which means that the process/thread that was using a resource left its critical region and stopped accessing to that resource. If the semaphore value was negative before this increment, this means that there were waiting processes/threads in the wait queue. And after we release the resource and increment the value of semaphore by 1, one of these waiting processes/threads can now start using that resource. So in that condition, we pick one thread from the wait queue and add it to the scheduler.

You can see the implementation of Semaphore below:

```
class Semaphore {

  int value;
    // The value of semaphore

  queue<Thread*> waitQ;
    // Queue that holds threads that are blocked (waiting) because of the lack of resources.

  void Init(int v);

  void wait();
    // This operation is called by a thread/process when it wants to acquire a resource.

  void signal();
    // This operation is called by a thread/process when it wants to release a resource.
}

void Semaphore::Init(int v) {

  value = v;

  waitQ.init();
    // Initialize an empty queue to hold threads
    // that are blocked (waiting) because of the lack of resources.                         
}

void Semaphore::wait() {                  
  value--;
    // Decrement the value of semaphore by 1.

  if (value < 0) {

    waitQ.add(current_thread);
      // If the new value of the semaphore is negative,
      // this means that resource(s) controlled by the semaphore was already being used.
      // In that case, add current thread into the wait queue because of the lack of available resources.

    current_thread->status = blocked;
      // Update the status of the thread as "blocked"
      // because it cannot access to the shared resource at this moment

    schedule();
      // Schedule another thread to avoid spinning and wasting CPU time.
  }
}

void Semaphore::signal() {                   
  value++;
    // Increment the value of the semaphore by 1.
    // This means that the process/thread that was using a resource left its critical region
    // and stopped accessing to that resource.

  if (value <= 0) {

    Thread *thd= waitQ.getNextThread();
      // If the semaphore value was negative before this increment,
      // this means that there were waiting processes/threads in the wait queue.
      // In that case, after the shared resource is being released in the previous steps,
      // one of these waiting processes/threads can now start using that resource.
      // In that condition, we pick one thread from the wait queue.

    scheduler->add(thd);
      // And then add the tread to the scheduler.
  }
}
```

Let's give another implementation of semaphores. Imagine that there are two processes: process A and process B: 

```
process A             process B 
----------            ----------
A1                    B1
A2                    B2
signal(s)
A3        \           B3
           \
            \
             \
              \
               \
                \
                 \
                  \
                   \
                     wait(s)
A4                   B4
A5                   B5 
```

And assume that we want the statement A2 in process A to be completed before statement B4 begins in process B. How we can do this ? 

We can put the **signal()** call after A2, and **wait()** call before B4. Through that way, after B1, B2, and B3 are executed, we wait the signal coming from the signal() operation. And that signal only comes after A2 is completed. Through this way, we can ensure that statement A2 is completed before statement B4 begins.

So if we want to summarize: semaphores are elegant solutions for sycnhronization of the processes/threads. But a **race condition** can **occur** like in the other methods if multiple processes/threads try to **execute the wait() and signal() operations simultaneously**. 

For instance, if one thread is in the middle of decrementing the semaphore value by using wait() operation and another thread tries to increment the same value in signal() operation at the same time, that will cause race condition. 

Therefore, we must **impement the wait() and signal()** operations in **atomic** way so that they should be executed exclusively.

We can make the wait() and signal() operations atomic by using **lock variable** 


```
class Semaphore {
  int lockvar;
    // The lock variable

  int value;
    // The value of semaphore

  queue<Thread*> waitQ;
    // Queue that holds threads that are blocked (waiting) because of the lack of resources.

  void Init(int v);

  void wait();
    // This operation is called by a thread/process when it wants to acquire a resource.

  void signal();
    // This operation is called by a thread/process when it wants to release a resource.
}

void Semaphore::Init(int v) {

  value = v;

  waitQ.init();
    // Initialize an empty queue to hold threads
    // that are blocked (waiting) because of the lack of resources.                         
}

void Semaphore::wait() {
  lock(&lockvar);
    // Lock the lock variable so that when a process/thread acquires resources,
    // no other process/threads can interfere with this.

  value--;
    // Decrement the value of semaphore by 1.

  if (value < 0) {

    waitQ.add(current_thread);
      // If the new value of the semaphore is negative,
      // this means that resource(s) controlled by the semaphore was already being used.
      // In that case, add current thread into the wait queue because of the lack of available resources.

    current_thread->status = blocked;
      // Update the status of the thread as "blocked"
      // because it cannot access to the shared resource at this moment

    unlock(&lockvar);
      // Unlock the lock variable so that another process/thread can start it's operations.

    schedule();
      // Schedule another thread to avoid spinning and wasting CPU time.
  }
  else {
    unlock(&lockvar);
      // Unlock the lock variable so that another process/thread can start it's operations.
  }
}

void Semaphore::signal() {

  lock(&lockvar);
    // Lock the lock variable so that when a process/thread release resources,
    // no other process/threads can interfere with this.

  value++;
    // Increment the value of the semaphore by 1.
    // This means that the process/thread that was using a resource left its critical region
    // and stopped accessing to that resource.

  if (value <= 0) {

    Thread *thd= waitQ.getNextThread();
      // If the semaphore value was negative before this increment,
      // this means that there were waiting processes/threads in the wait queue.
      // In that case, after the shared resource is being released in the previous steps,
      // one of these waiting processes/threads can now start using that resource.
      // In that condition, we pick one thread from the wait queue.

    scheduler->add(thd);
      // And then add the tread to the scheduler.
  }
  unlock(&lockvar);
    // Unlock the lock variable so that another process/thread can start it's operations.
}
```

In the examples above, we used **binary/mutex semaphores**. These semaphores can be seen as lock. They are ideal for mutual exclusion problems. In these problems, when the semaphore value is initialized to 1, this means that the lock is available. 

There is also another type of semaphores that is called **counting semaphores**. This is a type of semaphore that represents the number of processes/threads that are allowed to be in their critical regions at the same time. In cases when we want multiple processes/threads to enter their critical regions simultaenously, we can use counting semaphores. But in these cases, mutual exclusion is not guaranteed. 

Let's review the use case of both binary/mutex semaphores and counting semaphores in the producer-consumer problem. 

```
#define N 100               // Define the size of the buffer

Semaphore empty = N;        // Define a semaphore to manage the number of emtpy slots in the buffer
Semaphore full = 0;         // Define a semaphore to manage the number of full slots in the buffer
Semaphore mutex = 1;        // Define a semaphore to manage the lock/unlock process and to make operations atomic

T buffer[N];                // Define an empty buffer that has N elements
int widx = 0,               // Define an integer value to write items into the buffer
int ridx = 0;               // Define an integer value to read items from the buffer

#define N 100 // Define the size of the buffer

Semaphore empty = N; // Define a semaphore to manage the number of empty slots in the buffer (initially all slots are empty)
Semaphore full = 0; // Define a semaphore to manage the number of full slots in the buffer (initially all slots are empty)
Semaphore mutex = 1; // Define a semaphore to manage the lock/unlock process and to make operations atomic (initially unlocked)

T buffer[N]; // Define an empty buffer that has N elements
int widx = 0; // Define an integer value to write items into the buffer (initially at index 0)
int ridx = 0; // Define an integer value to read items from the buffer (initially at index 0)

Producer(T item) {
    wait(&empty); // Decrement the value of the empty semaphore by one. If the value of the empty semaphore was 0 before this decrementation, this means that there is no empty slot in the buffer. In this case, the producer process will be blocked and added to the empty semaphore's wait queue.

    wait(&mutex); // Lock the mutex to ensure exclusive access to the shared buffer before putting a new item to it.

    buffer[widx] = item; // Put an item into the next available empty slot in the buffer
    widx = (widx + 1) % N; // Update the index of the next available empty slot (using modulo to wrap around)

    signal(&mutex); // Unlock the mutex after putting an item into the buffer.

    signal(&full); // Increment the value of the full semaphore by one. If the value of the full semaphore was larger than or equal to N before this incrementation, this means that the buffer was full, and a consumer waiting on the full semaphore will be unblocked and added to the ready queue.
}

Consumer(T &item) {
    wait(&full); // Decrement the value of the full semaphore by one. If the value of the full semaphore was 0 before this decrementation, this means that there are no full slots in the buffer. In this case, the consumer process will be blocked and added to the full semaphore's wait queue.

    wait(&mutex); // Lock the mutex to ensure exclusive access to the shared buffer.

    item = buffer[ridx]; // Read an item from the next available full slot in the buffer
    ridx = (ridx + 1) % N; // Update the index of the next available full slot (using modulo to wrap around)

    signal(&mutex); // Unlock the mutex after reading an item from the buffer.

    signal(&empty); // Increment the value of the empty semaphore by one. If the value of the empty semaphore was larger than or equal to N before this incrementation, this means that the buffer was empty, and a producer process waiting on the empty semaphore will be unblocked and added to the ready queue.
}
```

So, semaphores are great but like all the other methods, they have some downsides as well:

- It is not always easy to write the codes with semaphores.
- If a thread dies while holding a semaphore, the permit to access to a shared resource is basically lost. And this can prevent other threads being blocked from accessing shared resource even. That's why we need to be extra careful when constructing the semaphores.

In addition, they may cause a situation in which the processes had to wait for resources(s) forever. To avoid this situation, we should

- acquire the multiple locks in the same order.
- release the locks in reverse order ideally.

And we can show how these solutions work in an example. In below, we can see two very similar but different codes: 

**1st Scenario:**
```
typedef int semaphore;   
  semaphore resource_1;
  semaphore resource_2;

  void process_A(void) {
    down(&resource_1);
    down(&resource_2);

    use_both_resources();

    up(&resource_2);
    up(&resource_1);
  }

  void process_B(void) {
    down(&resource_1);
    down(&resource_2);

    use_both_resources();

    up(&resource_2);
    up(&resource_1);
  }

```

**2nd Scenario:**
```
typedef int semaphore;
  semaphore resource_1;
  semaphore resource_2;

  void process_A(void) {
    down(&resource_1);
    down(&resource_2);

    use_both_resources();

    up(&resource_2);
    up(&resource_1);
  }

  void process_B(void) {
    down(&resource_2);
    down(&resource_1);

    use_both_resources();

    up(&resource_1);
    up(&resource_2);
  }

```

In the second case, process A can acquire the resource 1, and process B can acquire the resource 2. Then process A will attempt to acquire the resource 2 but it won't be able to because the resource 2 is hold by the process B. 

Similarly, process B will attempt to acquire the resource 1 but it won't be able to because the resource 1 is hold by the process A. This is called **deadlock** and both processes will wait to hold a resource forever.

But by acquiring locks in the same order and releasing them in the reverse order we can avoid deadlock.

Okay we have mentioned about the deadlock above but one note is that sometimes people can get confused about **deadlock** and **starvation**. That's why it may be useful to define both of them to see the differences between them.

**!!Lecture 4 - Slides between 50 and 72 are passed!!**

# Deadlock vs Starvation

- **Deadlock**: This occurs when the processes wait for a resource that will never be available. 
- **Starvation** The resource can become available at some point but if a process cannot get access to that resource, that process will experience starvation. In other words, the process waits for its turn but its turn never comes even if there is an available resource.

Deadlock occurs when **none** of the processes are able to move ahead while starvation occurs when a process waits for indefinite period of time to get the resource it needs to move forward. 

If we want to give an example, let's say that you submit a large document to printer. But if other people keep submitting small documents continuously, this means that when one small document is printed, another one starts and your document won't get a chance to be printed. Because the resources for printing are continuously being used by some other processes. This is where we see starvation. There is no deadlock in here because we don't see dependency between processes to move forward. Deadlock occurs among processes that are waiting to acquire resources in order to progress forever.

Okay we compared the deadlock and starvation and explained their differences but we also talked about the resources. What are the things that we call **resource** in general ? 

# Resources

If we want to explain simply, we call anything that needs to be 

1) acquired
2) used
3) released

 **resource**. 

So, a printer, lock variable, or semaphore, for instance, can be seen as resources because they all need to be acquired, used, and then released.

The resource can be **preemptable** or **nonpreemptable** as well. In other words, sometimes, you can take away some resources from the process while the process is actively using that resource. These resources are preemptable. But not all the resources are like this. There are also other resources that cannot/should not be taken away from the process if it is actively using it. 

We can also divide the resources into two categories: **Reusable**, and **Consumable**. 

**Reusable Resources**: We call a resource reusable if it is/can be used by **ONLY ONE** process at a time **over and over again**. **CPU, I/O channels, memory, data structures such as files, databases, semaphores can be given as an example of reusable resources**. 

**Consumable Resources**: These are the resources that can be created and then destroyed/consumed and once we consume these resources, they are not going to be available to be used anymore. **Interrupts, signals, messages, information in I/O buffers can be given as an example of consumable resources**. 

We have explained the interrupts before but if you don't know what the signal is: signal is a kind of mechanism that is used for **inter-process communication**. It is sent from one process to another, or from kernel to process to notify the process about an event. Signals can be **represented** in various ways **(e.g. constants, enumerated values, signal numbers that are basically integer values associated with specific signal types, etc.)**

One thing to note is that when we explained the deadlocks, we have made some **assumptions**. And these are: 

- If a process requests a resource and that resource is unavailable, the process is suspended and put into waiting (sleep) state until the requested resource becomes available for it.
- Once the process is put into the waiting state because of the lack of resources, no interrupt can wake up that process from the sleeping/waiting state. If you can send the signal externally, you can break the deadlock but that is not the definition of the deadlock. That is the definition of how we can recover from a deadlock.

And now, let's explain the **conditions of a deadlock**

- Each resource is either available or assigned to exactly one process
- Only one process can use a resource at a time. In other words, the resources cannot be used by multiple processes at the same time.
- Processes that are holding resources can request new resources.
- Granted resources cannot be taken away from a process. They must be explicitly released by the process that is holding them.
- There must be a circular chain of 2 or more processes. And each of these processes should be waiting a resource that will be released by the next member (process) of the chain

Well, it is good to know the conditions of a deadlock but another important question that we should answer is: when the deadlock happens, how can we deal with them ? 

# How to Deal with Deadlocks ? 

- You can ignore them.
- Let the deadlock occurs, detect it, and take an action.
  -  We can dynamically manage the resources to avoid deadlock. **(Deadlock Avoidance)**
  -  We can also break one of the conditions of the deadlock and prevent the deadlock. **(Deadlock Prevention)**

# Deadlock Detection and Recovery

The system actually does not attempt to prevent the deadlock. It just detects the deadlock as it happens. And once it detects the deadlock, it takes actions to recover.

But the question is: how we can detect the deadlock ? 
 
## Deadlock Detection with One Resource of Each Type

If there is only 1 resource for each type, constructing a resource graph is a good way to detect the deadlock. Because if we detect a cycle in that graph, this means that there is a deadlock.

We can give a system that has one scanner, one plotter, one tape-drive as an example of a system in which there is one resource of eacy type. Having two printers, however, would break the rule. 

Okay now the question is: how to detect cycles ? We can detect it by simply looking at it but how a computer can detect a cycle in a graph ? 

## Formal Algorithm to Detect Cycles

For each node in the graph: 

1) Initialize an empty list L.
2) Add current node to the end of L.
   - If the node appears twice, this means that there is a deadlock and we terminate the algorithm
3) Search for an unmarked outgoing and arc.
   - If there is an unmarked and outgoing arc, jump into the step 4.
   - If there is no unmarked and outgoing arc, jump into the step 5.
4) Pick an unmarked and outgoing arc randomly and mark it. Then follow it to the new node and jump into step 2.
5) If the current node is visited first time, this means that there is no cycle and we terminate the algorithm. Otherwise, we are in dead end and we remove the node and go back to the previous node. Then we jump into the 2nd step.

Okay we have mentioned about the deadlocks, conditions of the deadlock, and how to teach the computer to detect them but one another question is: when to check for deadlocks ? 

# When to Check Deadlocks ? 

- We can check when a resource is requested but this might be very expensive to run.
- We can check periodically (e.g. every x minutes)
- We can also check when CPU utilization drops to a level that is lower than a specific threshold. Because if the CPU utilization is low, this means that the amount of time spent for idle tasks is high. And this means that some processes or threads are not making enough progress and it may be because of the deadlock.

Okay let's say that we checked the deadlocks, and detected one. Now the question is: how we can recover from the deadlock ? 

## 1) Recovering from Deadlock Through Preemption

When a deadlock happens, we can take the resource away from its owner and give that resource to another process temporarily. And this requires manual intervention. We can recover from the deadlock throug this way but it is important to note that the decision of whether we should take away the resource from the process or not is highly dependent on the nature of resource. 

Also, this method is often not possible because taking the resource away from the process would cause unpredictable and nonoptimal behavior.  

## 2) Recovering from Deadlock Through Rollback 

We can also save the information about the processes periodically (e.g. every x minutes). And when a deadlock happens, we can roll the process back to the previous checkpoint. 

But this method may cause significant delays. 

## 3) Recovering from Deadlock Through Killing Process 

Another simple way to recover from deadlock is killing process(es). And we can continue killing processes until the deadlock is resolved. 

One note is that killing a process outside of the cycle can release resources that can be used to fix the deadlock. So we can kill process(es) outside of the cycle as well. 

So, until now, we covered deadlocks, its conditions, how to detect them, and how we can recover from them, but we didn't mention about how to avoid them before they happen. 

# Deadlock Avoidance

Most of the times, the resources are requested one at a time and we don't see many cases in which process(es) request(s) mutliple resources at the same time.

One main way to avoid deadlock is granting the resource **only if** it it is **safe** to grant the resource. But what do we mean by **safe** ? 

## Safe and Unsafe States

If there is a sequence of order in which each process in the sequence can run and can be completed, that state is called **safe**. Even if every process requests the maximum number of resources immediately, as long as each process can run and finish without a problem, then the state is safe. 

And an **unsafe** state is a state that will **potentially** result in deadlock in some period. 

In the example below, we see 3 processes: A, B, and C. Process A has 3 resources, B has 2 resources and C has 2 resources. All together, they have 7 resources in total. 

And process A needs 9 resources to be finished while prcoess B needs 4 resources, and process C needs 7 resources. And let's assume that there are 10 instances of resources available in total. 

```
A | 3 | 9
B | 2 | 4
C | 2 | 7

Free: 3
```

So in this example, since we have 10 instances of resources available and all the processes has 7 resources at this moment, there are 3 resources available. We can give 2 of these resources to B and in that case process B will be completed. So we can call this state as **safe** state. 

Once we give 2 of the resources to process B 

```
A | 3 | 9
B | 4 | 4
C | 2 | 7

Free: 1
```

process B will be completed and release all of the resources it was using back.

```
A | 3 | 9
B | 0 | -
C | 2 | 7

Free: 5
```

So now, we have 5 resources. If we give these 5 resources to process A, it won't be enough for it because it needs 6 resources to complete. In that case, we end up with **unsafe** state. So we give these 5 resources to process C. 

```
A | 3 | 9
B | 0 | -
C | 7 | 7

Free: 0
```

and once process C is completed, it will release its resources

```
A | 3 | 9
B | 0 | -
C | 0 | -

Free: 7
```

and we will have 7 resources available. Now we can use 6 resources in process A

```
A | 9 | 9
B | 0 | -
C | 0 | -

Free: 1
```

and complete all the processes.

```
A | 0 | -
B | 0 | -
C | 0 | -

Free: 9
```

In this example, all of these states were **safe** because in each of them, there was a scheduling order in which each process can run and can be completed. 

In another example, let's say that we have a state like this: 

```
A | 3 | 9
B | 2 | 4
C | 2 | 7

Free: 3
```

This is a **safe** safe. And now let's say that process A acquires one resource. 

```
A | 4 | 9
B | 2 | 4
C | 2 | 7

Free: 2
```

Now, this state is **unsafe** because the system cannot guarantee that all processes will finish unlike a **safe** state. Because at this step, we can give the available 2 resources only to process B,

```
A | 3 | 9
B | 4 | 4
C | 2 | 7

Free: 0
```

and after process B is completed and it releases all the resources, the total number of resources that is available won't be enough for neither process A or process B.

```
A | 3 | 9
B | 0 | -
C | 2 | 7

Free: 4
```

So in these examples, we tried to avoid the deadlock by using the concepts of **safe** and **unsafe** states. And avoiding the deadlock through this way is called **Banker's Algorithm**. 

# Banker's Algorithm

The main idea behind this algorithm is checking if granting a resource will lead to an **unsafe** state. If it won't, we just release the resource.

## Applying Banker Algorithm to Single Resource

```
A | 0 | 6
B | 0 | 5
C | 0 | 4
D | 0 | 7

Free: 10

(Safe)
```

```
A | 1 | 6
B | 1 | 5
C | 2 | 4
D | 4 | 7

Free: 2

(Safe)
```

```
A | 1 | 6
B | 2 | 5
C | 2 | 4
D | 4 | 7

Free: 1

(Unsafe)
```

## Applying Banker Algorithm to Multiple Resources

As in the single-resource case, processes must state their total resource needs before executing.


```
Process | Tape Drives | Plotters | Printers | CD ROMs
A       | 3           | 0        | 1        | 1
B       | 0           | 1        | 0        | 0
C       | 1           | 1        | 1        | 0
D       | 1           | 1        | 0        | 1
E       | 0           | 0        | 0        | 0

(How many resources are currently assigned to each process)

```

```
Process | Tape Drives | Plotters | Printers | CD ROMs
A       | 1           | 1        | 0        | 0
B       | 0           | 1        | 1        | 2
C       | 3           | 1        | 0        | 0
D       | 0           | 0        | 1        | 0
E       | 2           | 1        | 1        | 0

(How many resources each process still needs in order to be finished)

Existing Resources  = Tape Drivers: 6, Plotters: 3, Printers: 4, CD ROMs: 2
Processed Resources = Tape Drivers: 1, Plotters: 0, Printers: 2, CD ROMs: 0
Available Resources = Tape Drivers: 5, Plotters: 3, Printers: 2, CD ROMs: 2

```

To find the deadlock, we can

1) look for a row whose unmet resource needs are all smaller than or equal to the values in available resources. If there is no that kind of row, this means that there is a deadlock and terminate.
2) assume the process of the chosen row requests all resources it needs and then finishes. Mark that process as terminated and release its resources back to the available resources.
3) repeat steps 1 and 2 until either all the processes are terminated or no process is left.

The Banker Algorithm looks nice in theory but it is practically **not useful** because 

- prcoesses don't know the maximum number of resources they will need in advance.
- the number of processes is not fixed. We can start with process A, B, and C and then other processes might come or some of the existing processes might vanish. In those scenarios, Banker Algorithm is not a good solution.
- resources can vanish.

We talked about deadlock avoidance which basically means analyzing the resources dynamically and trying to make decisions that won't end up with deadlock. We can see deadlock avoidance as a more indirect way to prevent deadlocks before they happen.

But we can also try to prevent the system entering into deadlock directly by violating at least one of the conditions of deadlocks. And this is called **deadlock prevention**

# Deadlock Prevention 

We have mentioned about the conditions of the deadlock. For a deadlock to happen, all of those conditions were suppposed to be met simultaneously. And by violating at least one of these conditions, we can directly prevent the deadlock. 

## 1) Deadlock Prevention: Attacking the Mutual Exclusion

One of the conditions of the deadlock was the mutual exclusion. In other words, if one process is accessing to the resource, no other process is allowed to use that resource until it is released. If we violate this rule, this means that the resources will become sharable among multiple processes simultaneously and these processes can access to the shared resource concurrently without waiting another process to release it. 

Spooling can be a way to violate the mutual exclusion rule. A spooler is a program/software that puts the tasks into a queue temporarily. If multiple processes arrives to the spooler simultaneously, instead of giving the resource to one of them and excluding the others, it simply puts all of these processes into the queue in the first come first served manner. And when the resource becomes available, a job is extracted from the queue and the resource is given to that job. In this kind of example, it is impossible to observe deadlock because we don't give the resource to one process and restrict the other processes using that resource until it becomes available. We just put all of the processes into a queue. 

So attacking the mutual exclusion condition prevents the deadlock directly. But we cannot apply this procedure in all cases. Some resources should be exclusively accessed by one process at a time and in those cases, this solution cannot be used for those cases. 

## 2) Deadlock Prevention: Attacking the Hold and Wait Condition

Another condition of the deadlock was having process(es) that hold(s) a resource and that wait(s) a new resource to make progress. If we prevent this, in other words, if we prevent processes from waiting for new resources if they are currently holding resources, we can prevent the deadlock as well. 

One way to do this is to make processes to request all of the resources at the same time before starting the execution.

Imagine that there is a process named process A. If all the resources that process A needs are available, they can be allocated to the process A directly and process A can begin execution. But even if one the requested resources is not available, process A is not granted any of the resources and must wait until all the resources become available. And once the resources it needs are available to use, it can start. Through this way, we prevent incremental acquisition of the resources which is the potential cause of the deadlocks.

Another way is to make the process to release all the resources it holds when it wants to acquire new resources. After releasing all the resources, now it can try to acquire all the resources it needs at the same time. 

## 3) Deadlock Prevention: Attacking No Preemption Condition

The third condition of the deadlock was not allowing preemption. So we can prevent the deadlock by preempting a resource from a process when the same resource is required by another process. 

A good strategy to do this is virtualization. 

Virtualization allows multiple virtual machines/containers/etc. to share the same resources. Spooling, for instance, can be seen as virtualization. We can create an abstract layer that separates the logical view of the printer from the physical printer istelf. And after that, processes can interact with spooler instead of directly interacting with the physical printer. 

If a process has been assigned the printer and it is in the middle of printing its output, taking away the printer would be tricky and sometimes impossible without this abstract layer. By using spooler, in other words virtualizing the resources, spooling printer outputs to disk and allowing only printer daemon to access to the real printer, we can now preempt the resources and as a result deadlocks. 

However, one thing to note is that not all resources are eligible to be virtualized. 

## 4) Deadlock Prevention: Attacking The Circular Wait Condition 

**Method 1:** For us to observe the circular chain of dependencies, a process should hold one resource and wait another resource. So, if we have a rule forcing the processes to be entitled to a **single** resource at a moment, we can violate the circular wait condition and as a result prevent the deadlock. According to this rule, if a process needs a second source while holding another resource, it must first release the first resource. 

**Method 2:** Another way to violate the circular wait condition is having a rule forcing the processes to request resources in numerical order. They can request resources whenever they want as long as they request in numerical order. 

Let's say that we have these resources: 

1) Imagesetter
2) Printer
3) Plotter
4) Tape Drive
5) Blu-ray Drive

So if we apply this method, a process may request printer first and then a tape driver second but it can never request a plotter first and then printer second. 

If we have the process A and process B, and the resources are i and j, for instance, 

```
A   B
|   | 
i   j
```

at every instant, one of the assigned resources (i or j) will have the highest number. The process that is holding that resource will never ask for another resource that was already assigned. It will either be completed (if it doesn't require any other source) or request even higher numbered resources (and all of them will be available). This way, circular wait condition won't occur and as a result we won't see deadlock.

# Conclusion 

In summary, deadlocks can occur in hardware resources or software resources.

And the operating system needs to 
- try to avoid the as much as possible before they happen,
- detect these deadblocks when they happen,
- take actions and deal with them when they are detected.













  












