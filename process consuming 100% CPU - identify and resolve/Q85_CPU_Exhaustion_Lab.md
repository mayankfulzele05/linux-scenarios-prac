# 📂 DevOps Practice Lab: Debugging Runaway 100% CPU Processes (Q85)

This repository contains a safe, self-contained sandbox lab to replicate, diagnose, and resolve a **Runaway 100% CPU Production Outage** (`Q85` from the Linux DevOps guide). You will learn how to catch a misbehaving application loop, inspect its system calls without killing it, and safely lower its impact on a live cluster.

---

### 📋 Prerequisites

* A Linux environment (Ubuntu/Debian preferred).
* Standard administrative tools installed (`top`, `htop`, `strace`).
* Sudo privileges for deep process introspection.

---

### 🚀 Lab Setup & Simulation (Replicating the Issue)

#### 🔹 Step 1: Run the Runaway CPU Daemon
We will run a multi-threaded Python daemon script in the background. One of its worker threads will instantly spin up a tight, unoptimized infinite loop, locking an entire CPU core at **100% utilization**.

```bash
# Copy and execute this background python process block
python3 -c '
import threading, os, time

def normal_worker():
    while True:
        # A well-behaved thread that sleeps to prevent CPU exhaustion
        time.sleep(1)

def rogue_worker():
    print(f"🔥 Rogue Loop Active. Tracking Target System PID: {os.getpid()}")
    while True:
        # A tight infinite mathematical loop with ZERO pause states
        x = 100 * 100

t1 = threading.Thread(target=normal_worker, daemon=True)
t2 = threading.Thread(target=rogue_worker, daemon=True)

t1.start()
t2.start()

# Keep main execution loop awake for testing duration
time.sleep(600)
' &
```

*Note down the printed **PID** from the console output block for the diagnostics segment below.*

---

### 🔍 Diagnostic Workflow (The CPU Incident Triage)

Imagine your infrastructure monitoring service throws a firing high-load alert. Follow the professional investigation tree:

#### **1. Confirm the Culprit Process ID**
Initialize an interactive or real-time process list tree to sort tracking processes by active processor consumption.
```bash
# Standard real-time inspector (Press Shift+P to force sorting by active CPU)
top

# Interactive visually indexed equivalent (Highly recommended)
htop
```
*   **Observation:** Note the **PID** of the target `python3` process holding onto **99.9% to 100% CPU**.

#### **2. Inspect Active Kernel Activity (`strace`)**
Before blindly terminating an application, you must determine *what* the application is doing. If an application is writing critical data blocks to a database, forcing a hard kill (`kill -9`) can corrupt systemic database nodes.

Attach a system-call tracer to the live running process ID:
```bash
# Syntactical usage: sudo strace -p <PID_FROM_TOP_COMMAND>
# If your application PID is 12345, run:
sudo strace -p 12345
```
*   **Analysis of the Output Matrix:**
    *   *Scenario A:* The terminal screens flash an un-ending waterfall of rapid repetitive read/write loops. This indicates a network or disk polling blockage.
    *   *Scenario B (Our Sandbox Case):* The terminal window completely freezes and outputs absolutely nothing. This means the thread is caught inside a **purely internal CPU-bound computation loop** (user-space logic), never dropping back down to touch kernel system calls (`sys_calls`).

---

### 🧯 Incident Remediation (Live Production Mitigations)

In enterprise cloud infrastructures, completely stopping a core application can cause a user outage. We will reduce its computational impact safely while developers work on a hotfix patch.

#### **1. Throttle CPU Access Priority (`renice`)**
Linux process scheduling ranges from `-20` (highest priority) to `19` (lowest priority). By default, applications run at priority `0`. Let's lower the rogue application priority to the lowest possible tier so other critical system services get processed first:

```bash
# Syntactical structure: renice -n <PRIORITY_LEVEL> -p <PID>
sudo renice -n 19 -p 12345
```
*   **Verification Check:** Look back at your `top` or `htop` interface metrics. You will see the process yield execution times to other applications, immediately stabilizing the host server's load levels.

#### **2. Graceful Termination Drop**
If the application loop continues to degrade adjacent infrastructure nodes, initiate a polite termination signal (`SIGTERM`), allowing the process to drop active socket handlers cleanly:
```bash
sudo kill 12345
```

#### **3. Absolute Hard Kill (The Final Circuit-Breaker)**
If the process is stuck inside a hard un-interruptible thread loop and ignores standard signals, drop the core kernel bypass hammer (`SIGKILL`):
```bash
sudo kill -9 12345
```

---

### 🧼 Lab Environment Teardown

To cleanly wipe the testing loop from your local development workstation environment:

```bash
# Identify and terminate any remaining background python simulation containers
kill $(pgrep -f "rogue_worker") 2>/dev/null
```





<img width="1918" height="1077" alt="Screenshot (81)" src="https://github.com/user-attachments/assets/7c62c056-4957-431c-9df9-00d66bbe1efe" />
