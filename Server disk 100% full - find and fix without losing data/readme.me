# 📂 DevOps Practice Lab: Linux Disk 100% Full 

This repository contains a self-contained sandbox environment to safely replicate and solve a **100% Full Disk Production Outage** from the Linux DevOps engineering guide) without affecting your host machine's filesystem.

---

### 📋 Prerequisites

*   A Linux environment (Ubuntu/Debian preferred).
*   Sudo administrative privileges.

---

### 🚀 Lab Setup & Execution

#### 🔹 Step 1: Initialize the Virtual Sandbox
Instead of filling your real hard drive, we will allocate a **50MB virtual disk image file**, format it with a native Linux filesystem (`ext4`), and mount it locally.

```bash
# 1. Allocate a blank 50 Megabyte virtual disk image
dd if=/dev/zero of=/tmp/disk.img bs=1M count=50

# 2. Format the file container with a standard ext4 filesystem
mkfs.ext4 /tmp/disk.img

# 3. Create the targeted mount point directory
sudo mkdir -p /mnt/production-lab

# 4. Attach the virtual disk container to our mount point
sudo mount /tmp/disk.img /mnt/production-lab

# 5. Take operational ownership of the directory workspace
sudo chown -R $(whoami):$(whoami) /mnt/production-lab
```

---

#### 🔹 Step 2: Sabotage the Lab (Trigger 100% Utilization)
We will now establish standard operational folders alongside an active application framework, then fill the storage boundary completely using a runaway logging script.

```bash
# 1. Shift context into your storage lab environment
cd /mnt/production-lab

# 2. Replicate standard Linux log and cache trees
mkdir -p app/configs app/database var/logs var/cache

# 3. Write basic operational configuration files
echo "config_data" > app/configs/settings.conf
echo "db_index" > app/database/cluster.db

# 4. Flood the disk to trigger the targeted platform failure
# (Expect a 'No space left on device' exception output string)
cat /dev/zero > var/logs/runaway_application.log
```

---

### 🔍 Diagnostic Workflow (The Incident Response Plan)

#### **1. Confirm System-Level Saturation**
Execute a storage system block query to evaluate space constraints:
```bash
df -h /mnt/production-lab
```
*   **Expected Metric:** `/mnt/production-lab` must report **100% Use** with `0` remaining memory space.

#### **2. Map Directory Capacity Sizing**
Run the sizing aggregation pipeline directly inside your operational mount boundary to isolate the highest consumption path:
```bash
du -sh /mnt/production-lab/* | sort -rh | head -10
```
*   **Analysis:** The trace matrix safely implicates the `/mnt/production-lab/var` directory tree.

#### **3. Isolate the Rogue Entity**
Drill deeper down into the compromised path folder structure to pinpoint the rogue file entry:
```bash
du -sh /mnt/production-lab/var/* | sort -rh | head -10
```
*   **Analysis:** The data stream directly identifies `var/logs/runaway_application.log` as the primary space consumption source.

---

### 🧯 Incident Remediation

> ⚠️ **CRITICAL DEVOP SYSTEM WARNING:** Never blindly delete active logs using `rm`! If an active background daemon maintains an open file handle, the storage blocks remain allocated but completely invisible to standard directory queries (`du`), leading directly to the dreaded **Q82 scenario**.

#### **Safe Truncation Operation**
Zero out the log length while maintaining its underlying active file descriptor configuration:
```bash
> /mnt/production-lab/var/logs/runaway_application.log
```

#### **Post-Incident Validation Check**
Verify storage recovery across your active pool tree:
```bash
df -h /mnt/production-lab
```
*   **Expected Result:** System usage safely returns to normal baselines (approx. 1% to 2% configuration utilization) without data structural integrity loss.

---

### 🧼 Lab Environment Teardown

To cleanly wipe the testing loop from your local development environment, run:

```bash
# Move back to safety directory framework
cd ~

# Detach the isolated lab architecture volume pool
sudo umount /mnt/production-lab

# Destroy the mock workspace mount structures and disk file
sudo rm -rf /mnt/production-lab /tmp/disk.img
```





<img width="1920" height="1080" alt="Screenshot (75)" src="https://github.com/user-attachments/assets/d72837f9-4466-4a2e-9fc4-41504942adf2" />

