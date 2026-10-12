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
- Perform a basic file-write activity, observe disk read time, and compare HDDs with SSDs.
- Monitor I/O activity using `iostat` and Python.

## Scenario

You are investigating how a Linux operating system communicates with I/O devices. You will create a small **disposable virtual disk**, inspect devices and drivers, explore two I/O-handling approaches, and observe simple I/O activity. The goal is to understand and observe the concepts introduced in the Week 5 I/O lecture, not to administer real storage hardware.

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

**Lecture connection (block devices and disk partitioning):** Linux normally treats disks as *block devices*, meaning data is accessed in blocks. In this lab, you will create a regular file and let Linux present it as a pretend block device through a **loop device**. This gives you a safe way to explore the block-device interface without changing the VM's real system disk. A loop device is a software representation; it is **not a new physical disk**.

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

**Lecture connection (device drivers and controllers):** A **device controller** handles communication with the hardware. A **device driver** is operating-system software that communicates with the controller on behalf of the kernel. In Linux, some drivers are compiled into the kernel and others are provided as **loadable kernel modules**. Not every kernel module is a device driver; modules can also provide file systems and other kernel features.

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

# Part G – File I/O and Storage Characteristics

## Task 17 – Write and Read a File Using Python

When a program writes data to a file, it normally uses operating-system services rather than communicating directly with a storage controller. Python's `open()` gives us a simple way to perform file I/O. Python may **buffer** writes, which means it temporarily groups data before passing it to the operating system.

Create `file_io.py`:

```bash
nano file_io.py
```

Enter:

```python
with open("notes.txt", "w") as file:
    file.write("Linux uses drivers to communicate with devices.\n")
    file.write("Disks are examples of block devices.\n")

with open("notes.txt", "r") as file:
    print(file.read())
```

Run:

```bash
python3 file_io.py
ls -lh notes.txt
```

**Questions:** What did the program write to the file? Which statement reads the file? Why does an application use the operating system for file I/O rather than accessing hardware registers itself?

> **Note:** `with` automatically closes the file, even if an error occurs. You do not need to know the internal details of Python buffering to complete this task.

## Task 18 – Compare HDD and SSD Characteristics

In the lecture, you saw that a **hard disk drive (HDD)** uses spinning platters and a moving read/write head, while a **solid-state drive (SSD)** stores data electronically and has no moving mechanical parts.

Look again at the device information you obtained in Task 15:

```bash
lsblk -d -o NAME,SIZE,ROTA,MODEL
```

Complete the comparison using **your lecture slides** (not assumptions about your VM):

| Characteristic | HDD | SSD |
|---|---|---|
| Moving mechanical parts? | | |
| Why might random access be slower or faster? | | |
| Typical source of physical access delay | | |

**Questions:** Why do HDDs have *seek time* and *rotational delay*, while SSDs do not? Can the `ROTA` value in a virtual machine prove the type of physical storage in the host computer? Explain briefly.

> **Note:** A virtual machine may report a virtual disk as rotational or non-rotational independently of the actual storage hardware. No physical HDD or SSD is required for this comparison.

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
rm -f virtualdisk.img testfile notes.txt monitor_test.img trigger.txt
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
8. Why do programs use operating-system services to read and write files?
9. Why do HDDs have seek time and rotational delay, but SSDs do not?
10. What types of information can `iostat` and `psutil.disk_io_counters()` show? Why must you detach only the loop device you created?

# Final Validation

Before completing the lab, confirm that you have:

- [ ] Created and inspected a virtual disk file (and a loop device if permitted).
- [ ] Identified block and character devices.
- [ ] Used Linux tools and Python to inspect I/O information.
- [ ] Explained MMIO/PMIO and polling/event-driven behaviour.
- [ ] Inspected a kernel module or recorded why the environment prevented it.
- [ ] Read and wrote a text file using Python and compared HDD/SSD characteristics.
- [ ] Observed I/O statistics or recorded tool limitations.
- [ ] Safely cleaned up your temporary files.

## Summary

You explored how Linux exposes I/O devices, virtual disks, kernel modules, and drivers. You also compared polling with event notifications, read and wrote a text file, compared HDDs and SSDs, and used Linux and Python tools to observe system I/O activity. The examples illustrate important operating-system concepts while avoiding changes to real storage devices.
