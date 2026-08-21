# 📂 DevOps Practice Lab: Deleted Files Held Open (Q82)

This repository contains a clean, self-contained sandbox lab to replicate and resolve the phantom disk space incident (`Q82` from the Linux DevOps guide). You will learn how to locate invisible deleted files and free up disk space on a live system without stopping critical applications.

---

### 📋 Prerequisites

* A Linux environment (Ubuntu/Debian preferred).
* Sudo administrative privileges.

---

### 🚀 Lab Setup & Execution

#### 🔹 Step 1: Initialize the Virtual Sandbox
We will create a clean **50MB virtual disk image file**, format it, and mount it locally.

```bash
# 1. Allocate a blank 50 Megabyte virtual disk image
dd if=/dev/zero of=/tmp/phantom-disk.img bs=1M count=50

# 2. Format the file container with an ext4 filesystem
mkfs.ext4 /tmp/phantom-disk.img

# 3. Create the targeted mount point directory
sudo mkdir -p /mnt/phantom-lab

# 4. Attach the virtual disk container to our mount point
sudo mount /tmp/phantom-disk.img /mnt/phantom-lab

# 5. Take operational ownership of the directory workspace
sudo chown -R $(whoami):$(whoami) /mnt/phantom-lab
```

---

#### 🔹 Step 2: Sabotage the Lab (Create the Phantom Leak)
We will run a background Python process that creates a large 35MB log file (leaving enough room for filesystem metadata blocks), deletes the file immediately to copy a messy `rm` mistake, and stays asleep to hold the file descriptor open.

```bash
python3 -c '
import os, time
path = "/mnt/phantom-lab/live_app.log"
# 1. Open file and write 35 Megabytes immediately (fits safely with disk overhead)
f = open(path, "w")
f.write("X" * 35 * 1024 * 1024)
f.flush()
print("🔥 35MB File Created successfully.")

# 2. Delete the file from the directory tree right now
os.remove(path)
print("🗑️ File DELETED with rm (os.remove). It is gone from du!")

# 3. Freeze the process open so it holds the file descriptor awake
print("💤 Process is now sleeping for 5 minutes. GO TEST YOUR COMMANDS!")
time.sleep(300)
' &
```

---

### 🔍 Diagnostic Workflow (The Phantom Chase)

#### **1. Verify System Panic**
Run a system block storage query to check disk use:
```bash
df -h /mnt/phantom-lab
```
* **Observation:** The disk reports high storage use (**~85% to 95% full**).

#### **2. Run Directory Scans (The Trap)**
Now run a standard folder space aggregation to find out what file is causing the leak:
```bash
du -sh /mnt/phantom-lab/*
```
* **The Mystery:** The output reports that the folder is completely **empty or using 0 bytes**! The log file is gone from directory listings, but `df` still registers the disk space as occupied.

#### **3. Unmask the Phantom Leaking Blocks**
To solve this, use the specialized auditing utility tool `lsof`. The `+L1` flag instructs Linux to look explicitly for files that have **0 hard links** (deleted) but are still held open by an active process ID:
```bash
lsof +L1 | grep deleted
```
* **Expected Metric Output:**
    ```text
    COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF NLINK NAME
    python3 12345   user    3w   REG   8,16 36700160     0 /mnt/phantom-lab/live_app.log (deleted)
    ```
* **Analysis:** Process ID (`PID`) `12345` (our background Python process) is holding file descriptor `3w` (write access) open on a 35MB file that no longer exists on disk!

---

### 🧯 Incident Remediation

In a live production setting, you may not be allowed to kill the app because it could disrupt paying users. We must drain the file space **safely while the app keeps running**.

#### **Safe Live Truncation Strategy**
Every process exposes its file handles inside the virtual `/proc` system framework. We can look inside the specific process file descriptor path and truncate it directly to zero bytes:

```bash
# Syntactical format: > /proc/<PID>/fd/<FD_NUMBER>
# Based on the lsof trace above where PID is 12345 and FD is 3:
> /proc/12345/fd/3
```
*(Note: Replace 12345 with your actual process ID found via the `lsof` command).*

#### **Post-Incident Validation Check**
Verify the storage pool baseline again:
```bash
df -h /mnt/phantom-lab
```
* **Result:** The storage blocks instantly clear out, dropping the disk usage metrics safely back to normal parameters (~1%) without breaking the process execution state.

---

### 🧼 Lab Environment Teardown

To cleanly wipe this loop from your local workstation environment, terminate the running ghost task and drop the virtual partition:

```bash
# 1. Kill the loop app now that we are done testing
kill $!

# 2. Return to host safety environment
cd ~

# 3. Unmount and wipe the experiment structures
sudo umount /mnt/phantom-lab
sudo rm -rf /mnt/phantom-lab /tmp/phantom-disk.img
```


<img width="1920" height="1080" alt="Screenshot (76)" src="https://github.com/user-attachments/assets/4dbfc8dc-9fc1-4cc1-bb02-f179ec1bbfe7" />




<img width="1927" height="1068" alt="Screenshot (77)" src="https://github.com/user-attachments/assets/1da97fed-b00d-48ca-850a-f656ae2e01e8" />


