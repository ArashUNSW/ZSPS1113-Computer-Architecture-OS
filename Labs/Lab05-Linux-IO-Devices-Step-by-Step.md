# Lab 05 – Exploring Linux I/O Devices with Python and CLI

**Estimated time:** 90–120 minutes  
**Environment:** Skillable Linux lab VM (Ubuntu/Debian-based) or equivalent Linux VM  
**Tools:** Linux terminal, Python 3, `psutil`, and standard Linux utilities

> **Important:** Run all commands inside the **Linux lab VM**, not on your personal computer. GitHub displays this Markdown guide; commands run in the Skillable terminal. Some hardware details and device names vary between virtual machines.

## Learning Objectives

By the end of this lab, you should be able to:

- Identify block and character devices and explain the role of device drivers.
- Create and inspect a safe file-backed virtual disk.
- Inspect Linux storage and I/O devices using command-line tools and Python.
- Distinguish memory-mapped I/O from port-mapped I/O, and polling from interrupt-driven I/O.
- Compare basic file-writing interfaces and observe disk read time.
- Monitor I/O activity using `iostat` and Python.

## Scenario

You are investigating how a Linux operating system communicates with I/O devices. You will create a small **disposable virtual disk**, inspect devices and drivers, explore two I/O-handling approaches, and observe simple I/O activity. The goal is to understand the concepts, not to administer real storage hardware.

## Safety and Setup

- **Do not format, overwrite, or modify the VM's system disk** (such as `/dev/sda`, `/dev/vda`, or `/dev/nvme0n1`).
- Only attach and detach the loop device created from **your own** `~/lab05/virtualdisk.img` file.
- Commands requiring `sudo` may be unavailable in some Skillable environments. If a command is blocked, **record the result and continue**; do not attempt to bypass restrictions.
- Do not copy example device names blindly. Check the actual device name in your VM.
- Command outputs will differ between VMs. These differences are normal.

---

# Part A – Prepare the Environment

## Task 1 – Confirm Your Linux Environment

Open the terminal and run:

```bash
whoami
pwd
python3 --version
```

**Check:** Which user are you logged in as? What version of Python is installed?

## Task 2 – Create Your Working Directory

```bash
mkdir -p ~/lab05
cd ~/lab05
pwd
```

Use this directory for the remaining activities unless a command specifies another path.

---

# Part B – Create and Inspect a Virtual Disk

## Task 3 – Create a 200 MiB Virtual Disk File

```bash
dd if=/dev/zero of=virtualdisk.img bs=1M count=200 status=progress
ls -lh virtualdisk.img
```

`dd` copies data from an input to an output. Here, `/dev/zero` provides zeros and `virtualdisk.img` stores the result. **Do not change the output path to a physical device.**

**Check:** Approximately how large is the file?

## Task 4 – Attach the File to a Loop Device

A *loop device* lets Linux access an ordinary file through a block-device interface.

```bash
sudo losetup --find --show ~/lab05/virtualdisk.img
```

An output such as `/dev/loop9` means the file was attached. Your device number will differ. Record it here:

**My loop device:** `________________`

If `sudo` or loop devices are not available, record this and continue with the remaining tasks. You can still inspect the virtual disk **file**.

## Task 5 – Identify the Virtual Disk

```bash
lsblk -o NAME,SIZE,TYPE,MOUNTPOINTS
sudo losetup -a
```

Find the loop device linked to your file (if the previous task succeeded). **Do not confuse it with the VM's system disk.**

**Question:** What is the difference between `virtualdisk.img` and its `/dev/loopX` representation?

---

# Part C – Identify I/O Devices

## Task 6 – Explore Block Devices

```bash
lsblk -o NAME,SIZE,TYPE,MOUNTPOINTS
```

Identify one disk and, if available, one loop device. Record their types and approximate sizes.

**Question:** Why are disks usually accessed as block devices?

## Task 7 – Explore Character Devices

```bash
ls -l /dev/null /dev/zero /dev/tty
```

Look at the first character in each listing (`c` indicates a character device). If `/dev/tty` is unavailable in your session, use `/dev/null` and `/dev/zero`.

**Question:** In general terms, how is a character device different from a block device?

## Task 8 – Inspect Disk Information with Python

Check for `psutil`:

```bash
python3 -c "import psutil; print(psutil.__version__)"
```

If it is not installed and package installation is allowed, run:

```bash
sudo apt-get update
sudo apt-get install -y python3-psutil
```

Create `disk_info.py`:

```bash
nano disk_info.py
```

Enter the following code:

```python
import psutil

print("Mounted disk partitions:")
for partition in psutil.disk_partitions():
    print(partition.device, partition.mountpoint, partition.fstype)
    try:
        usage = psutil.disk_usage(partition.mountpoint)
        print("  Used:", usage.percent, "%")
    except (PermissionError, OSError):
        print("  Usage not available")
```

Save and run:

```bash
python3 disk_info.py
```

**Question:** What do the device name, mount point, file-system type, and percentage used tell you? If `psutil` cannot be installed, use `df -h` for a basic comparison and record the limitation.

---

# Part D – Controllers and I/O Concepts

## Task 9 – Inspect I/O Controllers

Try:

```bash
lspci
```

To filter for storage or USB controllers:

```bash
lspci | grep -Ei 'storage|sata|nvme|usb|scsi'
```

If `lspci` is missing and installation is permitted, install `pciutils` using `sudo apt-get install -y pciutils`.

**Question:** What does an I/O controller do? If the VM does not expose controller details, explain why a virtual machine might show different hardware from a physical PC.

## Task 10 – Compare Memory-Mapped and Port-Mapped I/O

Read the descriptions:

- **Memory-mapped I/O (MMIO):** Device registers occupy addresses in the processor's memory address space.
- **Port-mapped I/O (PMIO):** Some processors, notably x86, provide a separate I/O address space with dedicated I/O instructions.

Applications usually interact with hardware through operating-system drivers rather than accessing registers directly.

**Question:** Describe one difference between MMIO and PMIO. Why does an operating system control access to device registers?

---

# Part E – Polling and Events

## Task 11 – Demonstrate Polling with Python

Create `polling_demo.py`:

```bash
nano polling_demo.py
```

```python
import os
import time

print("Checking for trigger.txt; press Ctrl+C to stop")
try:
    while True:
        if os.path.exists("trigger.txt"):
            print("Trigger detected")
            os.remove("trigger.txt")
            break
        time.sleep(1)
except KeyboardInterrupt:
    print("Stopped")
```

Run it in the **first** terminal:

```bash
python3 polling_demo.py
```

Open a **second terminal**, go to the same directory, and create the trigger:

```bash
cd ~/lab05
touch trigger.txt
```

**Question:** Why is the program an example of polling? What happens if you increase the sleep interval?

## Task 12 – Demonstrate an Interrupt-Like Callback

Create `event_demo.py`:

```bash
nano event_demo.py
```

```python
import threading

def notify():
    print("Event notification received")

timer = threading.Timer(3.0, notify)
print("Waiting for an event...")
timer.start()
timer.join()
```

Run:

```bash
python3 event_demo.py
```

**Question:** How does this callback differ from repeatedly checking for a file? **Note:** This is a software event demonstration, **not a real hardware interrupt**.

---

# Part F – Device Drivers and Storage

## Task 13 – List Loaded Kernel Modules

```bash
lsmod | head -15
```

Kernel modules can provide drivers and other kernel functionality.

**Question:** Why might Linux load some drivers as modules? If `lsmod` is unavailable, inspect `/proc/modules` using `head -15 /proc/modules`.

## Task 14 – Inspect Module Information

Choose a module name from the first column of the `lsmod` output. Replace `MODULE_NAME` with that name:

```bash
modinfo MODULE_NAME | head -20
```

**Record:** Module name, description (if shown), and filename (if shown). Some modules have limited metadata, and built-in drivers may not appear in `lsmod`.

## Task 15 – Inspect Disk Characteristics

```bash
lsblk -o NAME,SIZE,ROTA,TYPE,MODEL
```

If available, check:

```bash
lsblk -d -o NAME,SIZE,ROTA,MODEL
```

`ROTA` is a device-reported rotational indicator (`1` usually indicates rotational storage, `0` non-rotational). In a VM, these details can reflect **virtualised hardware**, not the host's physical disk.

**Question:** What can you infer from the device size, type, and rotational indicator? What information might be hidden by virtualisation?

## Task 16 – Observe Basic Read Time

Create a test file in your lab directory:

```bash
dd if=/dev/zero of=testfile bs=1M count=100 status=progress
```

Read it and measure elapsed time:

```bash
time cat testfile > /dev/null
```

Run the timed command twice.

**Record:** The `real` time from each run. Were the times identical?

> This is a simple observation activity, **not** a reliable disk benchmark: OS caching and VM conditions affect the results.

---

# Part G – File I/O with Python

## Task 17 – Write a File Using Python

Create `buffered_io.py`:

```bash
nano buffered_io.py
```

```python
with open("buffered.txt", "w") as file:
    for i in range(10000):
        file.write(f"Line {i}\n")

print("File created using Python's file object")
```

Run:

```bash
python3 buffered_io.py
ls -lh buffered.txt
```

**Question:** What does the `with open(...)` statement do? Why is buffering useful?

## Task 18 – Compare with the `os.write()` Interface

Create `oswrite_demo.py`:

```bash
nano oswrite_demo.py
```

```python
import os

fd = os.open("lower_level.txt", os.O_WRONLY | os.O_CREAT | os.O_TRUNC, 0o644)
try:
    os.write(fd, b"Written through a file descriptor\n")
finally:
    os.close(fd)

print("File created using os.write")
```

Run:

```bash
python3 oswrite_demo.py
cat lower_level.txt
```

**Question:** How does this code differ from Task 17? **Note:** `os.write()` provides a lower-level system-call interface; it does **not** mean the data bypasses the OS page cache or uses hardware direct I/O.

---

# Part H – Monitor I/O Activity

## Task 19 – Observe Disk Statistics with `iostat`

Check whether `iostat` is installed:

```bash
command -v iostat
```

If not found and installation is permitted:

```bash
sudo apt-get install -y sysstat
```

Run:

```bash
iostat -d 1 5
```

While it runs, you can create some file activity in another terminal:

```bash
cd ~/lab05
dd if=/dev/zero of=monitor_test.img bs=1M count=50 status=none
```

**Question:** What do `tps`, `kB_read/s`, or `kB_wrtn/s` represent? Did you observe any activity? Results can be low or absent in a shared VM. If `iostat` is unavailable, record the limitation and continue.

## Task 20 – Observe I/O Counters with Python

Create `io_monitor.py`:

```bash
nano io_monitor.py
```

```python
import time
import psutil

previous = psutil.disk_io_counters()
if previous is None:
    print("Disk I/O counters are unavailable in this environment")
else:
    print("Watching I/O counters for 5 seconds...")
    time.sleep(5)
    current = psutil.disk_io_counters()
    if current is not None:
        print("Read count change:", current.read_count - previous.read_count)
        print("Write count change:", current.write_count - previous.write_count)
        print("Bytes written change:", current.write_bytes - previous.write_bytes)
```

Run:

```bash
python3 io_monitor.py
```

**Question:** What does a change in the counters mean? Why might the reported activity include operations from other processes or the VM itself? If `psutil` is unavailable, explain what this script would measure.

---

# Cleanup – Complete After Task 20

**Only detach the loop device associated with your own file.** First inspect its association:

```bash
sudo losetup -j ~/lab05/virtualdisk.img
```

If this lists a loop device, substitute that exact device for `/dev/loopX` below:

```bash
sudo losetup -d /dev/loopX
```

**Never detach an unrelated loop device.** If your file was never attached, skip detachment. After detaching, remove only the lab-generated files:

```bash
cd ~/lab05
rm -f virtualdisk.img testfile buffered.txt lower_level.txt monitor_test.img trigger.txt
```

You may keep the Python scripts as learning resources. Check the directory:

```bash
ls -lh
```

If privileged commands are blocked, inform your instructor rather than attempting to force cleanup.

---

# Knowledge Check

1. What is the difference between a block device and a character device?
2. What does a loop device allow Linux to do?
3. How does Linux use device drivers to communicate with hardware?
4. What is one difference between MMIO and PMIO?
5. How is polling conceptually different from an event notification or interrupt?
6. Why might device details in a virtual machine differ from physical hardware?
7. Why might two runs of a file read test produce different timings?
8. How does Python's `open()` interface differ from `os.write()`?
9. What types of information do `iostat` and `psutil.disk_io_counters()` show?
10. Why is it important to detach only the loop device you created?

# Final Validation

Before completing the lab, confirm that you have:

- [ ] Created and inspected a virtual disk file (and a loop device if permitted).
- [ ] Identified block and character devices.
- [ ] Used Linux tools and Python to inspect I/O information.
- [ ] Explained MMIO/PMIO and polling/event-driven behaviour.
- [ ] Inspected a kernel module or recorded why the environment prevented it.
- [ ] Tested simple file read and write operations.
- [ ] Observed I/O statistics or recorded tool limitations.
- [ ] Safely cleaned up your temporary files.

## Summary

You explored how Linux exposes I/O devices, virtual disks, kernel modules, and drivers. You also compared I/O approaches and used Linux and Python tools to observe file access and system I/O activity. The examples illustrate important operating-system concepts while avoiding changes to real storage devices.
