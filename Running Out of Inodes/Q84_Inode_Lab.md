# 📂 DevOps Practice Lab: Running Out of Inodes (Q84)

This repository contains a self-contained sandbox environment to safely replicate and solve an **Inode Exhaustion Production Outage** (`Q84` from the Linux DevOps guide). You will learn how a disk can throw a "No space left on device" error while showing gigabytes of free data space.

---

### 📋 Prerequisites
* A Linux environment (Ubuntu/Debian preferred).
* Sudo administrative privileges.

---

### 🚀 Lab Setup & Simulation (Replicating the Issue)

#### 🔹 Step 1: Initialize the Virtual Sandbox
Instead of creating millions of files on your real hard drive (which can slow down your OS), we will format a tiny **20MB virtual loop disk**. This sets a small, easy-to-hit limit on the filesystem's inode table.

```bash
# 1. Allocate a blank 20 Megabyte virtual disk image file
dd if=/dev/zero of=/tmp/inode-disk.img bs=1M count=20

# 2. Format it with a native ext4 filesystem
mkfs.ext4 /tmp/inode-disk.img

# 3. Create the targeted mount point directory
sudo mkdir -p /mnt/inode-lab

# 4. Attach the virtual disk container to our mount point
sudo mount /tmp/inode-disk.img /mnt/inode-lab

# 5. Take operational ownership of the directory workspace
sudo chown -R $(whoami):$(whoami) /mnt/inode-lab
```

---

#### 🔹 Step 2: Sabotage the Lab (Trigger Inode Exhaustion)
We will run a script that creates thousands of tiny, empty text files inside a mock application cache folder. This will completely fill up the inode index numbers while using almost zero actual disk space.

```bash
# 1. Shift context into your storage lab environment
cd /mnt/inode-lab

# 2. Replicate a standard application metadata structure
mkdir -p app/configs var/app_cache

# 3. Flood the disk to trigger the targeted platform failure
i=1
while true; do
    touch var/app_cache/$i.txt 2>/dev/null
    if [ $? -ne 0 ]; then
        echo "💥 TOUCH COMMAND FAILED! No space left on device error triggered successfully."
        break
    fi
    ((i++))
done
```

---

### 🔍 Diagnostic Workflow (The Inode Hunt)

#### **1. Confirm Table Depletion**
```bash
df -h /mnt/inode-lab
df -i /mnt/inode-lab
```
* **The Reveal:** The data blocks show that the **IUse% is sitting at 100%**, with `0` remaining free inodes available, while physical space shows nearly empty.

#### **2. Locate the Target Directory**
```bash
find /mnt/inode-lab -xdev -printf '%h\n' | sort | uniq -c | sort -rn | head -10
```

---

### 🧯 Incident Remediation
```bash
find /mnt/inode-lab/var/app_cache -type f -delete
```

---

### 🧼 Lab Environment Teardown
```bash
cd ~
sudo umount /mnt/inode-lab
sudo rm -rf /mnt/inode-lab /tmp/inode-disk.img
```
