# 📂 Incident Response Playbook: Diagnosing OOM Killer Drops (Q86)

This playbook outlines standard operational procedures (SOPs) for identifying, auditing, and mitigating Out-Of-Memory (OOM) kernel interventions where critical production daemons crash unexpectedly without dropping application log traces.

---

### 🧠 The Core Architecture: The Kernel Selection Matrix

When system memory resources hit absolute exhaustion levels, the Linux kernel triggers a protective sub-routine known as the **OOM Killer**. To prevent an unrecoverable system-wide kernel panic, it loops through running processes, computes an internal vulnerability algorithm score (`oom_score`), and instantly fires a forceful `SIGKILL (9)` at the highest offender.

The vulnerability calculation is based on:
1.  **Percentage of RAM consumed:** The higher the physical footprint, the more vulnerable the process becomes.
2.  **Process Niceness value:** Background priority settings modify the baseline risks.
3.  **Manual adjustments:** Values written explicitly inside `/proc/<PID>/oom_score_adj`.

---

### 🚀 Lab Simulation Protocol

#### **1. Deploy the Allocation Loop**
Execute the targeted background memory allocation framework to force a kernel-level termination event:
```bash
python3 -c '
import time, os
print(f"🔥 Memory Eater Started on PID: {os.getpid()}")

leak_pool = []
while True:
    # Continuously stack 10MB memory blocks inside a global array allocation matrix
    leak_pool.append("X" * 10 * 1024 * 1024)
    time.sleep(0.01)
' &
```

---

### 🔍 Diagnostic Workflow (The Autopsy Plan)

#### **1. Check the Kernel Ring Trace**
Query the native kernel logging matrix to capture automated system interventions:
```bash
sudo dmesg -T | grep -i -E 'oom|killed'
```

#### **2. Inspect Systemd Journal Frameworks**
If hardware tracing ring buffers are rotated under high load, parse systemd structures:
```bash
sudo journalctl -k --since "10 minutes ago" | grep -i oom
```
*   **Target Output String:** Look for explicit `Out of memory: Killed process <PID> (<NAME>)` log blocks indicating precise timestamp allocations.

#### **3. Evaluate Exit Code Metrics**
*   **Code 137:** Indicates standard `128 + 9 (SIGKILL)` system behavior. If a service container terminates with exit code 137, it means the host operating system forcefully dropped the application due to RAM limitations.

---

### 🧯 Incident Prevention & Remediation

#### **1. Configure Systemd Immunity Rules**
For critical platform processes managed via systemd configurations, ensure the unit file includes an absolute reduction parameter inside the `[Service]` block to protect it during low-resource emergencies:
```ini
[Service]
OOMScoreAdjust=-1000
```

#### **2. Run Manual Process Throttling**
To dynamically immunize a running application daemon process on the fly without restarting services:
```bash
echo -1000 | sudo tee /proc/<PID>/oom_score_adj
```



<img width="1914" height="1068" alt="Screenshot (82)" src="https://github.com/user-attachments/assets/aed638c7-7529-4798-82f2-df39864c32e9" />


