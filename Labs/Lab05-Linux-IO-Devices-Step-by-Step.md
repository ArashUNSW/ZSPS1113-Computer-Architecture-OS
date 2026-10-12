# Lab 05 - Exploring Linux I/O Devices with Python and CLI

## Estimated Time

Approximately **90–120 minutes** for the guided lab.

Allow additional time for the graded submission tasks at the end.

---

## Learning Objectives

By the end of this lab, you will be able to:

- Explain how Linux represents block and character I/O devices.
- Create a file-backed virtual disk safely using `dd`.
- Map a virtual disk file to a loop device using `losetup`.
- Identify storage devices and character devices.
- Use Python and `psutil` to inspect disk partitions and disk usage.
- Inspect I/O controllers with `lspci`, `dmesg`, `hdparm`, and `lshw`.
- Explain the difference between memory-mapped I/O and port-mapped I/O.
- Explain the difference between polling and interrupt-driven I/O.
- Inspect loaded kernel modules and device-driver information.
- Compare storage access and transfer behaviour.
- Compare buffered and lower-level file I/O.
- Monitor I/O activity using `iostat`, `dstat`, and Python.
- Safely simulate storage pressure and I/O errors using a disposable file-backed device.

---

## Scenario

You are continuing your Linux systems administration work.

In the previous lab, you explored Linux file systems and disk management. In this lab, you will extend that knowledge by examining Linux I/O behaviour.

You will work with file-backed virtual disks, loop devices, block and character devices, storage controllers, device drivers, polling and interrupt-like behaviour, disk latency, buffered and unbuffered I/O, and performance-monitoring tools.

> [!alert]
> Some commands in the original lab use the system disk directly.
>
> In this converted lab, all potentially destructive storage experiments should be performed only on a **disposable file-backed virtual disk or instructor-provided practice device**.
>
> Do not write to the active system disk such as `/dev/sda` unless explicitly instructed.

---

# Part A - Prepare the Lab Environment

## Task 1 - Confirm Your Linux Environment

Run:

```bash
whoami
```

Then:

```bash
pwd
```

Check Python:

```bash
python3 --version
```

Check whether `psutil` is available:

```bash
python3 -c "import psutil; print(psutil.__version__)"
```

If it is missing:

```bash
sudo apt update
sudo apt install python3-psutil -y
```

---

# Part B - Create a File-Backed Virtual Disk

## Task 2 - Create a Safe Working Directory

Create a directory for the lab:

```bash
mkdir -p ~/lab05
```

Move into it:

```bash
cd ~/lab05
```

Confirm:

```bash
pwd
```

---

## Task 3 - Create a 200 MB Virtual Disk File

Run:

```bash
dd if=/dev/zero of=virtualdisk.img bs=1M count=200 status=progress
```

Check it:

```bash
ls -lh virtualdisk.img
```

Expected result:

```text
A file approximately 200 MB in size.
```

### Command Breakdown

| Element | Meaning |
|---|---|
| `if=/dev/zero` | Input source that continuously provides zero bytes |
| `of=virtualdisk.img` | Output file |
| `bs=1M` | Block size of 1 MB |
| `count=200` | Copy 200 blocks |
| `status=progress` | Show progress while running |

---

## Task 4 - Map the File to a Loop Device

Run:

```bash
sudo losetup -fP ~/lab05/virtualdisk.img
```

List active loop devices:

```bash
sudo losetup -a
```

You should see an entry similar to:

```text
/dev/loop9: (...) (/home/student/lab05/virtualdisk.img)
```

Your loop-device number will probably be different.

Record:

```text
Assigned loop device:
```

---

## Task 5 - Confirm the Loop Device

Run:

```bash
lsblk
```

Look for the loop device associated with `virtualdisk.img`.

> [!note]
> A loop device allows Linux to treat a regular file as if it were a block device.

---

# Part C - Identify and Classify I/O Devices

## Task 6 - List Block Devices

Run:

```bash
lsblk -o NAME,MAJ:MIN,RM,SIZE,RO,TYPE,MOUNTPOINTS
```

Record examples of block devices.

| Device | Type | Size | Mounted? |
|---|---|---:|---|
| | | | |
| | | | |

---

## Task 7 - List Character Devices

Run:

```bash
ls /dev | grep tty
```

You may see entries such as:

```text
tty
tty0
tty1
ttyS0
```

These are examples of character-device interfaces.

### Question

What is the difference between a block device and a character device?

Record one example of each from your own system.

---

# Part D - Inspect Disk Information with Python

## Task 8 - Create a Python Disk Information Script

Create:

```bash
nano disk_info.py
```

Enter:

```python
import psutil

print("Disk Partitions:")
for p in psutil.disk_partitions():
    print(p)

print("\nDisk Usage:")
for p in psutil.disk_partitions():
    try:
        print(f"{p.device} =>", psutil.disk_usage(p.mountpoint))
    except PermissionError:
        print(f"{p.device} => Permission denied")
```

Run:

```bash
python3 disk_info.py
```

Observe the device name, mount point, file-system type, mount options, and total/used/free space.

---

# Part E - Inspect I/O Controllers

## Task 9 - Inspect PCI Devices

Run:

```bash
lspci
```

To search for USB controllers:

```bash
lspci | grep -i usb
```

If nothing is displayed, record:

```text
No matching USB controller entry observed.
```

---

## Task 10 - Inspect Kernel Messages for USB Devices

Run:

```bash
dmesg | grep -i usb | head -20
```

If permission is denied, try:

```bash
sudo dmesg | grep -i usb | head -20
```

Record any controller or device information you observe.

---

# Part F - Understand Memory-Mapped and Port-Mapped I/O

## Task 11 - Review MMIO and PMIO

### Memory-Mapped I/O

In **memory-mapped I/O (MMIO)**, device registers are exposed within the processor's memory address space. The CPU accesses them using ordinary memory-style load/store instructions.

### Port-Mapped I/O

In **port-mapped I/O (PMIO)**, devices use a separate I/O address space. On x86 systems, privileged code can use instructions such as `IN` and `OUT`.

> [!note]
> User applications normally do not access hardware registers directly. Linux exposes devices through kernel drivers for safety and portability.

---

## Task 12 - Inspect Storage Hardware Information

First identify the actual disk name:

```bash
lsblk
```

If your main disk is `/dev/sda`, run:

```bash
sudo hdparm -I /dev/sda | head -20
```

If your VM uses a different device name, substitute it appropriately.

You can also run:

```bash
sudo lshw -class disk -short
```

Record:

| Item | Observed value |
|---|---|
| Disk device | |
| Model | |
| Approximate capacity | |

---

# Part G - Polling and Interrupt-Like Behaviour

## Task 13 - Simulate Polling

Create `polling_demo.py`:

```python
import os
import time

print("Polling for trigger.txt. Press Ctrl+C to stop.")

while True:
    if os.path.exists("trigger.txt"):
        print("Detected trigger!")
        os.remove("trigger.txt")
    time.sleep(1)
```

Run:

```bash
python3 polling_demo.py
```

In a second terminal:

```bash
touch trigger.txt
```

Observe the first terminal and then press `Ctrl + C`.

### Question

Why is this considered polling?

---

## Task 14 - Simulate an Interrupt-Like Event

Create `interrupt_demo.py`:

```python
import threading

def interrupt_like_event():
    print("Interrupt received: I/O completed.")

timer = threading.Timer(5.0, interrupt_like_event)
timer.start()

print("Waiting for I/O interrupt...")
timer.join()
```

Run:

```bash
python3 interrupt_demo.py
```

Expected behaviour:

```text
Waiting for I/O interrupt...
```

Then approximately five seconds later:

```text
Interrupt received: I/O completed.
```

### Question

How does this differ conceptually from the polling example?

---

# Part H - Explore Device Drivers

## Task 15 - List Loaded Kernel Modules

Run:

```bash
lsmod | head
```

---

## Task 16 - Inspect a Kernel Module

We use `ext4` as an example:

```bash
modinfo ext4
```

If `ext4` is built into the kernel rather than loaded as a module, choose another module from `lsmod`.

Record:

```text
Module name:
Description:
Filename:
```

---

## Task 17 - Inspect Device Metadata with `udevadm`

Identify your disk:

```bash
lsblk
```

Then inspect it, for example:

```bash
udevadm info --query=all --name=/dev/sda | head -30
```

Use the actual disk device in your VM.

---

# Part I - Storage Structure and Access Latency

## Task 18 - Inspect Disk Model Information

Run:

```bash
sudo hdparm -I /dev/sda | grep -i model
```

Substitute the correct disk if necessary. If the virtual disk does not support this query, record that result rather than guessing.

---

## Task 19 - Perform a Read Timing Test

Create a test file:

```bash
dd if=/dev/zero of=~/lab05/testfile bs=1M count=200 status=progress
```

Measure sequential read time:

```bash
time cat ~/lab05/testfile > /dev/null
```

Record:

```text
real:
user:
sys:
```

> [!note]
> This is a simple observation exercise, not a complete storage benchmark.

---

# Part J - Buffered and Lower-Level File I/O

## Task 20 - Create a Buffered Python Write Test

Create `buffered_io.py`:

```python
with open("logfile.txt", "w") as f:
    for i in range(100000):
        f.write(f"Line {i}\n")

print("Buffered write completed.")
```

Run:

```bash
python3 buffered_io.py
```

Check:

```bash
ls -lh logfile.txt
```

---

## Task 21 - Create a Lower-Level `os.write` Example

Create `oswrite_demo.py`:

```python
import os

fd = os.open(
    "unbuffered_log.txt",
    os.O_WRONLY | os.O_CREAT | os.O_TRUNC,
    0o644
)

os.write(fd, b"Write performed using os.write().\n")
os.close(fd)

print("os.write example completed.")
```

Run:

```bash
python3 oswrite_demo.py
```

Display the file:

```bash
cat unbuffered_log.txt
```

> [!note]
> `os.write()` uses a lower-level file-descriptor interface than Python's normal buffered file object. This should not be described as true hardware direct I/O.

---

# Part K - Monitor I/O Performance

## Task 22 - Install Monitoring Tools

Check `iostat`:

```bash
iostat --version
```

If unavailable:

```bash
sudo apt update
sudo apt install sysstat -y
```

Check `dstat`:

```bash
dstat --version
```

If unavailable:

```bash
sudo apt install dstat -y
```

---

## Task 23 - Monitor Extended Disk Statistics

Run:

```bash
iostat -xm 1
```

Observe read/write throughput, utilisation, request size, and wait time. Press `Ctrl + C` to exit.

---

## Task 24 - Monitor Multiple Resource Types

Run:

```bash
dstat -cdngyt
```

Observe CPU, disk, network, paging, and system activity. Press `Ctrl + C` to exit.

---

## Task 25 - Monitor Disk I/O with Python

Create `io_monitor.py`:

```python
import psutil
import time

for i in range(5):
    print("Disk IO:", psutil.disk_io_counters())
    time.sleep(2)
```

Run:

```bash
python3 io_monitor.py
```

Observe how the counters change over time.

---

# Part L - Safe Storage Pressure and Error Simulation

## Task 26 - Check Available Space Before Testing

Run:

```bash
df -h .
```

Do not create a large test file if insufficient space is available.

---

## Task 27 - Create a Moderate Test Allocation

Instead of immediately allocating 2 GB, create a smaller test file:

```bash
fallocate -l 200M ~/lab05/junkfile
```

Check:

```bash
ls -lh ~/lab05/junkfile
```

Then remove it:

```bash
rm ~/lab05/junkfile
```

---

## Task 28 - Safely Demonstrate an Out-of-Range Read

Do **not** run the original out-of-range test against the physical system disk.

Find your loop device:

```bash
sudo losetup -a
```

Suppose it is `/dev/loop9`. Run:

```bash
sudo dd if=/dev/loop9 of=/dev/null bs=512 count=1 skip=999999999
```

Use the actual loop device assigned to your virtual disk.

Record:

```text
Error or result observed:
```

### Question

Why does the read fail or return no data?

---

# Part M - Clean Up

## Task 29 - Detach the Loop Device

Find it:

```bash
sudo losetup -a
```

Detach it:

```bash
sudo losetup -d /dev/loopX
```

Replace `/dev/loopX` with the actual device.

---

## Task 30 - Remove Temporary Files

```bash
rm -f ~/lab05/virtualdisk.img
rm -f ~/lab05/testfile
rm -f ~/lab05/logfile.txt
rm -f ~/lab05/unbuffered_log.txt
```

# Final Validation

Confirm that you have:

- [ ] Created a file-backed virtual disk.
- [ ] Mapped it to a loop device.
- [ ] Distinguished block and character devices.
- [ ] Used Python `psutil` to inspect disk information.
- [ ] Inspected I/O controllers.
- [ ] Explained MMIO and PMIO.
- [ ] Demonstrated polling.
- [ ] Demonstrated an interrupt-like event.
- [ ] Inspected loaded kernel modules.
- [ ] Inspected storage device information.
- [ ] Compared file I/O methods.
- [ ] Monitored I/O activity using `iostat`, `dstat`, and Python.
- [ ] Performed safe storage-error experiments on a loop device.

---

# Troubleshooting

## Problem - `losetup` Does Not Show the Virtual Disk

```bash
ls -lh ~/lab05/virtualdisk.img
sudo losetup -fP ~/lab05/virtualdisk.img
```

## Problem - `hdparm` Is Missing

```bash
sudo apt update
sudo apt install hdparm -y
```

## Problem - `lshw` Is Missing

```bash
sudo apt install lshw -y
```

## Problem - `iostat` Is Missing

```bash
sudo apt install sysstat -y
```

## Problem - `dstat` Is Missing

```bash
sudo apt install dstat -y
```

## Problem - LVM Commands Are Missing

```bash
sudo apt install lvm2 -y
```

## Problem - NTFS Formatting Tool Is Missing

```bash
sudo apt install ntfs-3g -y
```

## Problem - FAT Formatting Tool Is Missing

```bash
sudo apt install dosfstools -y
```

---

# Knowledge Check

1. What is a file-backed virtual disk?
2. What does `losetup` do?
3. What is the difference between a block device and a character device?
4. What information does `lsblk` provide?
5. What information can `psutil.disk_partitions()` provide?
6. What is MMIO?
7. What is PMIO?
8. Why does Linux normally prevent user applications from directly accessing hardware registers?
9. What is polling?
10. What is interrupt-driven I/O?
11. What is the role of a device driver?
12. What does `lsmod` display?
13. What does `modinfo` display?
14. What information can `iostat` provide?
15. What is the difference between Python's normal buffered file writes and `os.write()`?
16. Why should destructive disk experiments be performed on a loop device instead of the system disk?
17. What is an LVM Physical Volume?
18. What is a Volume Group?
19. What is a Logical Volume?
20. Why should persistent mounts use the actual UUID values from the local system?

---

# Optional Challenge

Using the safe 200 MB file-backed disk created earlier:

1. attach it to a loop device;
2. identify the loop device;
3. inspect it with `lsblk`;
4. monitor I/O with `iostat`;
5. detach it cleanly;
6. write three statements:

```text
Observed I/O behaviour:
Linux abstraction involved:
Safety control used:
```

---

## Summary

In this lab, you:

- created a file-backed virtual disk;
- mapped it to a loop device;
- identified block and character devices;
- inspected disk information using Linux commands and Python;
- reviewed memory-mapped and port-mapped I/O;
- simulated polling and interrupt-like events;
- inspected kernel modules and driver metadata;
- explored storage timing and I/O behaviour;
- compared buffered and lower-level user-space I/O;
- monitored system I/O activity;
- practised safe storage-pressure and error testing;
- investigated process file descriptors and inodes through `/proc`;
- created an LVM structure on a file-backed loop device;
- created NTFS and FAT logical volumes;
- configured mount points and persistent mounts.

You are now ready to work with more advanced Linux I/O, storage, and device-management concepts.
