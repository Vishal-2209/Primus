---
created: 2026-08-30
purpose: Operating Systems concepts for technical interview
---
# Operating Systems Cheatsheet

## 1. Process vs Thread

| Aspect | Process | Thread |
|--------|---------|--------|
| Definition | Independent program in execution | Lightweight process within a process |
| Memory | Separate address space | Shared address space |
| Creation | Expensive (copy entire address space) | Cheap (share parent's memory) |
| Communication | IPC (pipes, sockets, shared memory) | Direct read/write to shared variables |
| Crash Impact | Isolated - doesn't affect others | Can crash entire process |
| Example | Chrome browser vs VS Code | Chrome tabs (each tab = thread) |

### Python Context
```python
import threading
import multiprocessing

# Thread - good for I/O bound tasks
t = threading.Thread(target=func)
t.start()

# Process - good for CPU bound tasks
p = multiprocessing.Process(target=func)
p.start()
```

---

## 2. Process States

```
New -> Ready -> Running -> Terminated
         ^        |
         |  Waiting (I/O)
         +--------+
```

- **New**: Process being created
- **Ready**: Waiting to be assigned to CPU
- **Running**: Instructions being executed
- **Waiting**: Waiting for I/O to complete
- **Terminated**: Finished execution

---

## 3. CPU Scheduling Algorithms

### First Come First Serve (FCFS)
- Non-preemptive
- Simple, but convoy effect (short processes wait behind long ones)

### Shortest Job First (SJF)
- Non-preemptive version of SRTF
- Optimal average waiting time
- Problem: hard to predict burst time

### Round Robin (RR)
- Preemptive, time quantum based
- Fair, no starvation
- Quantum too small = high context switch overhead
- Quantum too large = becomes FCFS

### Priority Scheduling
- Each process has a priority
- Problem: starvation of low-priority processes
- Solution: aging (gradually increase priority of waiting processes)

---

## 4. Deadlock

### Four Conditions (All must hold simultaneously)
1. **Mutual Exclusion**: Resource cannot be shared
2. **Hold and Wait**: Process holds resource while waiting for another
3. **No Preemption**: Resources cannot be forcibly taken
4. **Circular Wait**: Circular chain of processes waiting for each other

### Prevention Strategies
| Strategy | Breaks Condition | Trade-off |
|----------|-----------------|-----------|
| Lock ordering | Circular Wait | Must maintain consistent order |
| Try-lock / timeout | Hold and Wait | May need retry logic |
| Resource hierarchy | Circular Wait | Design complexity |
| Deadlock detection | Let it happen, then fix | Recovery cost |

### Banker's Algorithm (Avoidance)
- Before granting resource, check if system remains in **safe state**
- Safe state = there exists a sequence where all processes can finish
- Used in: Database systems, OS resource allocation

---

## 5. Memory Management

### Virtual Memory
- Each process gets its own virtual address space
- Maps to physical RAM via page tables
- Allows programs to use more memory than physically available

### Paging
- Memory divided into fixed-size blocks: **frames** (physical) and **pages** (logical)
- Page table maps virtual pages to physical frames
- **Page fault**: Page not in RAM -> OS fetches from disk (slow)

### TLB (Translation Lookaside Buffer)
- Cache for page table entries
- Speeds up virtual -> physical address translation
- TLB hit: ~1 cycle | TLB miss: ~10-100 cycles

---

## 6. Synchronization

### Race Condition
Two threads access shared data concurrently, and result depends on execution order.

```python
import threading

counter = 0
lock = threading.Lock()

def safe_increment():
    with lock:
        global counter
        counter += 1
```

### Mutex vs Semaphore
| Aspect | Mutex | Semaphore |
|--------|-------|-----------|
| Ownership | Thread-specific | No ownership |
| Value | Binary (0 or 1) | Integer count |
| Use Case | Critical section protection | Resource pooling, signaling |

---

## 7. I/O Models

| Model | Description | Use Case |
|-------|-------------|----------|
| Blocking I/O | Thread waits until I/O complete | Simple apps |
| Non-blocking I/O | Returns immediately with data or EAGAIN | Polling-based |
| I/O Multiplexing | select/poll/epoll monitors multiple FDs | Web servers |
| Asynchronous I/O | OS notifies when I/O complete | High-performance |

### select vs poll vs epoll (Linux)
- **select**: O(n) scan, limited to 1024 FDs
- **poll**: O(n) scan, no FD limit
- **epoll**: O(1) event-driven, scales to millions of FDs

---

## 8. System Calls

| Category | Examples |
|----------|----------|
| Process | fork(), exec(), wait(), exit() |
| File | open(), read(), write(), close() |
| Memory | mmap(), brk(), munmap() |
| Network | socket(), bind(), listen(), accept() |

### fork() Creates Child Process
- Child gets copy of parent's address space
- Returns 0 to child, child PID to parent
- exec() replaces process image with new program

---

## 9. Linux Commands You Should Know

```bash
ps aux                    # List all processes
top / htop                # Real-time process monitor
free -h                   # RAM usage
df -h                     # Disk space usage
netstat -tlnp             # Listening ports
tail -f /var/log/syslog   # Follow log file
```

---

## Interview Quick Answers

**Q: Difference between process and thread?**
A: Process has its own memory space; threads share memory within a process. Thread creation is cheaper. Process isolation provides safety; thread sharing provides efficiency.

**Q: What is a deadlock? How to prevent it?**
A: Four conditions: mutual exclusion, hold-and-wait, no preemption, circular wait. Prevent by breaking one: use lock ordering (breaks circular wait), try-lock with timeout (breaks hold-and-wait).

**Q: What is virtual memory?**
A: Mechanism that gives each process its own address space, mapped to physical RAM via page tables. Allows programs to use more memory than physically available through paging to disk.

**Q: Mutex vs Semaphore?**
A: Mutex = binary (locked/unlocked), owned by thread. Semaphore = counting, no ownership. Use mutex for protecting critical section; use semaphore for signaling/resource pooling.

---

> **Key for Vishal**: You use threads in your Celery workers (LawPrix) and process-level isolation in Docker containers (PGPulse). Connect OS concepts to your real experience.
