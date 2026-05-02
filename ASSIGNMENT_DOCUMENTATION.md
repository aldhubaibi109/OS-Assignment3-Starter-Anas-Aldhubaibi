# Assignment 3 - Complete Documentation

**Student Name**: [Anas Aldhubaibi]  
**Student ID**: [445052785D]  
**Date Submitted**: [1/5/2026]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: https://drive.google.com/file/d/1AAlHMf91JgZYG64RMutzIsSmQS20BqBn/view?usp=drivesdk

**Video filename**: `Assignment3_Synchronization_445052785`

**Verification**:
- [ ] Link is accessible (tested in incognito mode)
- [ ] Video is 3-5 minutes long
- [ ] Video shows code walkthrough and commits
- [ ] Video has clear audio
- [ ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [30/4, 10:00 PM]
**What I implemented**: Forked the starter repository, cloned it to my local machine, configured Git, and set my student ID in the simulation file.

**Challenges encountered**: Git configuration issues (author identity unknown) when trying to make the first commit.

**How I solved it**: Used git config --global to set my university email and name properly before committing.

**Testing approach**: Checked GitHub repository online to ensure the commit and student ID changes were successfully pushed.

**Time spent**: 20 minutes.

---

### Entry 2 - [1/5, 2:00 PM]
**What I implemented**: Task 1 and 2. Added a ReentrantLock to protect shared counters (contextSwitchCount, completedProcessCount, totalWaitingTime) and the executionLog ArrayList.

**Challenges encountered**: Ensuring that the lock is always released to prevent deadlocks, even if an exception occurs during counter updates.

**How I solved it**: Wrapped all critical sections inside a try block and placed the lock.unlock() strictly inside a finally block.

**Testing approach**: Compiled the code to ensure no syntax errors and reviewed the logic to confirm mutual exclusion.

**Time spent**: 45 minutes.

---

### Entry 3 - [1/5, 3:30PM]
**What I implemented**: 
Task 3. Added a binary Semaphore (1 permit) to control concurrent CPU access for the processes.
**Challenges encountered**: 
Figuring out exactly where to acquire and release the semaphore in the process lifecycle.
**How I solved it**: 
Added acquire() at the very beginning of the run() and runToCompletion() methods, and placed release() in the finally block at the end.
**Testing approach**: 
Ran the simulation to ensure only one process executes its time quantum at any given moment without overlapping outputs.
**Time spent**: 
50 minutes
---

### Entry 4 - [Date, Time]
**What I implemented**: 

**Challenges encountered**: 

**How I solved it**: 

**Testing approach**: 

**Time spent**: 

---

### Entry 5 - [Date, Time]
**What I implemented**: 

**Challenges encountered**: 

**How I solved it**: 

**Testing approach**: 

**Time spent**: 

---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

[Counters (contextSwitchCount): The ++ operation is not atomic. If two threads increment it at the exact same time, one update might get lost, resulting in incorrect final totals.

Execution Log (ArrayList): ArrayList is not thread-safe. Multiple threads adding messages simultaneously can crash the program with a ConcurrentModificationException or corrupt the list order.]

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

[A ReentrantLock provides mutual exclusion (only one thread can access a resource at a time). A Semaphore limits how many threads can access a resource simultaneously.
I used a ReentrantLock to protect shared variables (counters and log) from data corruption. I used a Semaphore(1) to control the CPU, ensuring only one process executes its time quantum at a time.]

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

[Deadlock happens when threads are permanently stuck waiting for each other to release resources. Two ways to prevent it: 1) Always release locks in a finally block, and 2) Acquire locks in a fixed order.
In my code, I prevented deadlocks by wrapping all critical sections in try-finally blocks. This guarantees unlock() and release() always execute, even if an exception occurs.]

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

[I used a coarse-grained approach (one lock for all counters). I chose this because the operations are just simple increments, making the code simpler and avoiding deadlocks.
The trade-off is lower concurrency (a thread updating one counter blocks others from updating different counters). Since the counters are independent, fine-grained locking (one lock per counter) would give better performance, but it is too complex for this simple simulation.]

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: 
contextSwitchCount, completedProcessCount, and totalWaitingTime.
**Why they need protection**: 
Multiple threads updating these counters simultaneously can cause race conditions (lost updates) because operations like count++ are not atomic.
**Synchronization mechanism used**: 
ReentrantLock
**Code snippet**:
```java
public static void incrementContextSwitch() {
        lock.lock();
        try {
            contextSwitchCount++;
        } finally {
            lock.unlock();
        }
    }
```

**Justification**: 
Using a lock ensures that only one thread can read, increment, and write back the counter value at a time. The try-finally block guarantees the lock is always released to prevent deadlocks.
---

### Critical Section #2: Execution Log

**What resource**: 
executionLog (which is an ArrayList).
**Why it needs protection**: 
ArrayList is not a thread-safe data structure. Concurrent additions can corrupt its internal array or throw a ConcurrentModificationException
**Synchronization mechanism used**: 
ReentrantLock
**Code snippet**:
```java
public static void logExecution(String message) {
        lock.lock();
        try {
            executionLog.add(message);
        } finally {
            lock.unlock();
        }
    }
```

**Justification**: 
Applying the lock ensures that structural modifications to the array list happen sequentially, maintaining data integrity and preventing runtime exceptions.
---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: 
To act as a gatekeeper for the simulated CPU, controlling how many process threads can execute their processing logic at the same time.
**Number of permits and why**: 
1 permit. This simulates a single-core processor where only one process can execute its time quantum at any given moment.
**Where implemented**: 
Inside the Process class, specifically at the beginning and end of the run() and runToCompletion() methods.
**Code snippet**:
```java
    public void run() {
        try {
            SharedResources.cpuSemaphore.acquire();
        } catch (InterruptedException e) {
            System.out.println(Colors.RED + "  ✗ " + name + " interrupted while waiting for CPU." + Colors.RESET);
            return;
        }
        
        try {
           
        } finally {
            
            SharedResources.cpuSemaphore.release();
        }
    }
```

**Effect on program behavior**: 
It forces threads that are in the "Ready Queue" to wait (block) until the current executing thread finishes its quantum and releases the permit, creating an organized, non-overlapping terminal output.
---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: 
```bash
javac SchedulerSimulationSync.java
java SchedulerSimulationSync
# (Repeated this process 5 times)
```

**Results**: 
(In all 5 runs, the total number of completed processes always matched the exact number of initially generated processes. The execution output in the terminal was organized without any overlapping text from different threads.)

**Why synchronization is necessary**: 
(Without synchronization, multiple process threads would try to write to the terminal and update shared counters at the same millisecond. This WOULD lead to lost updates (e.g., 15 processes finish, but the counter only shows 13) and a completely garbled terminal output.)

**Conclusion**: 
The applied synchronization mechanisms successfully guarantee program consistency across multiple executions.
---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException
in the executionLog.
**Testing procedure**: 
Ran the simulation with the maximum randomized number of processes to force high concurrency and frequent log writing.
**Results**: 
The simulation completed successfully from start to finish without throwing any runtime exceptions.
**What this proves**: 
This proves that the ReentrantLock correctly serializes access to the ArrayList. By allowing only one thread to call executionLog.add() at a time, we maintain the structural integrity of the list.
---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)

**Expected values**: 
Total Completed Processes must equal the number of processes shown at the start of the simulation.
**Actual values**: 
If the simulation started with 15 processes, the final statistics exactly reported Total Completed Processes: 15.
**Analysis**: 
The 100% match between started and completed processes confirms that the completedProcessCount counter was perfectly protected by the lock, missing zero increments despite being accessed concurrently.
---

### Test 4: Different Scenarios
**Scenario tested**: [e.g., different time quantum, more processes, etc.]
High contention (Many processes yielding CPU frequently).
**Purpose**: 
To observe if the CPU Semaphore properly handles a large queue of threads waiting for the processor.
**Results**: 
The semaphore correctly blocked all waiting threads. Only the thread that successfully acquired the 1 permit was allowed to print its progress bar. The CPU never executed two processes simultaneously.
**What I learned**: 
Semaphores are highly effective at managing access to a limited resource (like a single-core CPU), acting as a strict turnstile regardless of how many threads are waiting in line.
---

## Part 5: Reflection and Learning

### What I learned about synchronization:

[I learned that simple operations like count++ can cause big problems in multithreading if we don't protect them. I now understand how ReentrantLock protects shared variables so only one thread can change them at a time. Also, I learned that Semaphore is useful for controlling access to things like the CPU. The most important lesson was to always put unlock() and release() in a finally block so the program doesn't freeze in a deadlock.]

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: Bank ATMs. Synchronization stops two people from withdrawing the same money from a joint account at the exact same time. 

**Example 2**: Online Ticket Booking. It prevents the system from selling the exact same cinema or flight seat to two different customers.

---

### How I would explain synchronization to others:

[Imagine three students trying to write on the same small whiteboard at the same time. The board will be a mess. Synchronization is like having only one marker. If you have the marker (the lock), you can write. When you finish, you give the marker to the next student. This keeps everything organized.]

---

## Part 6: GitHub Repository Information

**https://github.com/aldhubaibi109/OS-Assignment3-Starter-Anas-Aldhubaibi.git**: 

**Number of commits**: 
4
**Commit messages**: 
1. Set my student ID: 445052785 
2. Task 1 complete: Protected counters using ReentrantLock and try-finally blocks
3. Task 3 complete: Added CPU Semaphore to control concurrent process execution
4. Task 4 complete: Added reflection and testing documentation

---

## Summary

**Total time spent on assignment**: 
5 hours
**Key takeaways**: 
1. We must protect shared variables using locks to avoid data loss.
2. Always use try-finally to prevent deadlocks.
3. Semaphores are great for controlling CPU access.

**Most challenging aspect**: 
Knowing exactly where to put the acquire() and release() methods for the Semaphore without making the program freeze.
**What I'm most proud of**: 
Fixing the race conditions and seeing the terminal output print cleanly without any overlapping text.
---

**End of Documentation**
