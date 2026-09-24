# Lab 04 - Understanding Linux File Systems and Management

## Estimated Time

Approximately **60–90 minutes**.

---

## Learning Objectives

By the end of this lab, you will be able to:

- Explain how Linux represents disks and partitions.
- Identify block devices and partitions under `/dev`.
- Display partition information using `fdisk`.
- Explain the difference between MBR and GPT partitioning.
- Use `fdisk` and `gdisk` in a safe, read-only way.
- Explain how file systems are created using `mkfs`.
- Mount a file system to a mount point.
- Identify inode numbers and explain what an inode stores.
- Use `df` and `du` to analyse disk usage.
- Explain the role of `fsck` in checking file-system consistency.
- Distinguish between safe inspection commands and commands that can modify or destroy data.

---

## Scenario

You are working as a junior Linux system administrator.

Your task is to inspect how disks, partitions, file systems, mount points, inode information, and disk usage are represented in Linux.

You will use standard Linux commands to examine the storage configuration of the lab machine.

You will also review partitioning and file-system management tools.

> [!alert]
> Some storage commands can permanently destroy data if used incorrectly.
>
> In this lab, do **not** create, delete, format, or repair partitions unless your instructor explicitly provides a separate practice disk.
>
> The main system disk, such as `/dev/sda`, must be treated as **read-only** for inspection.

---

## Before You Begin

You need:

- a Linux virtual machine;
- a terminal;
- a normal user account with `sudo` privileges.

Open a terminal before starting.

---

# Part A - Explore Linux Storage Devices

## Task 1 - Confirm Your User Account

Run:

```bash
whoami
```

Then:

```bash
pwd
```

Expected result:

```text
Your username and current working directory should display.
```

---

## Task 2 - List Block Devices

Instead of immediately entering the `/dev` directory, first use:

```bash
lsblk
```

Expected output may look similar to:

```text
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   20G  0 disk
├─sda1   8:1    0   19G  0 part /
└─sda2   8:2    0    1G  0 part [SWAP]
```

Your output will be different.

> [!note]
> `lsblk` is a convenient and safe way to display disks and partitions.

Record:

| Device | Type | Size | Mount point |
|---|---|---:|---|
|  |  |  |  |
|  |  |  |  |

---

## Task 3 - Inspect Device Files Under `/dev`

Run:

```bash
cd /dev
```

Then list common SATA/SCSI disk device files:

```bash
ls sd*
```

Example:

```text
sda
sda1
sda2
```

Return to your home directory:

```bash
cd ~
```

> [!note]
> Linux represents devices as special files, commonly stored under `/dev`.
>
> For example:
>
> ```text
> /dev/sda
> ```
>
> may represent a whole disk, while:
>
> ```text
> /dev/sda1
> ```
>
> represents a partition on that disk.

---

# Part B - Display Partition Information

## Task 4 - Display All Partition Tables

Run:

```bash
sudo fdisk -l
```

This displays partition information without entering interactive editing mode.

Look for:

- disk name;
- disk size;
- sector size;
- partition names;
- start and end sectors;
- partition size;
- partition type.

> [!note]
> Modern versions of `fdisk` normally use:
>
> ```bash
> sudo fdisk -l
> ```
>
> rather than older combinations such as `fdisk -cul`.

---

## Task 5 - Interpret the Partition Table

A typical `fdisk -l` output includes fields such as:

| Field | Meaning |
|---|---|
| Device | Partition device name, such as `/dev/sda1` |
| Start | Starting sector |
| End | Ending sector |
| Sectors | Number of sectors |
| Size | Approximate partition size |
| Type | Partition type |

Record one partition from your machine:

| Device | Start | End | Size | Type |
|---|---:|---:|---:|---|
|  |  |  |  |  |

---

## Task 6 - Display Information for One Disk

From your `lsblk` output, identify the main disk.

For example:

```text
/dev/sda
```

Then run:

```bash
sudo fdisk -l /dev/sda
```

> [!alert]
> Replace `/dev/sda` only with the correct disk shown on your own lab VM.
>
> This command is read-only because of the `-l` option.

---

# Part C - Understand Interactive Partitioning

## Task 7 - Review `fdisk` Interactive Commands Safely

The `fdisk` utility can also edit partition tables.

Important commands include:

| Command | Purpose |
|---|---|
| `p` | Print current partition table |
| `n` | Create a new partition |
| `d` | Delete a partition |
| `t` | Change partition type |
| `m` | Display help |
| `q` | Quit without saving |
| `w` | Write changes to disk |

> [!alert]
> Do **not** use `n`, `d`, `t`, or `w` on your system disk during this lab.
>
> Writing partition-table changes to the wrong disk can make the system unusable or destroy data.

If your instructor provides a dedicated empty practice disk, they may demonstrate interactive mode separately.

---

## Task 8 - View `fdisk` Help

Run:

```bash
fdisk --help
```

This is a safe way to inspect available command-line options.

Expected result:

```text
A list of fdisk options should display.
```

---

# Part D - MBR and GPT

## Task 9 - Understand MBR and GPT

Two common partition-table formats are:

### MBR

**Master Boot Record (MBR)** is an older partition-table format.

Traditionally, MBR supports up to:

```text
4 primary partitions
```

without using an extended partition.

### GPT

**GUID Partition Table (GPT)** is a newer partitioning format.

GPT supports many more partitions and is commonly used on modern systems.

A typical GPT implementation supports up to:

```text
128 partitions
```

---

## Task 10 - Check the Partition Table Type

Run:

```bash
sudo fdisk -l
```

Look for information such as:

```text
Disklabel type: gpt
```

or:

```text
Disklabel type: dos
```

Record your result:

```text
Partition table type:
```

> [!note]
> In `fdisk`, `dos` usually refers to an MBR-style partition table.

---

## Task 11 - Check Whether `gdisk` Is Available

Run:

```bash
gdisk --version
```

If it is not installed on an Ubuntu/Debian system:

```bash
sudo apt update
sudo apt install gdisk -y
```

Then run:

```bash
gdisk --version
```

> [!note]
> `gdisk` is designed primarily for GPT partition tables.

---

## Task 12 - Review `gdisk` Commands Safely

Run:

```bash
gdisk
```

If it displays usage information, review it and exit.

You may also use:

```bash
gdisk --help
```

Important interactive `gdisk` commands include:

| Command | Purpose |
|---|---|
| `p` | Print partition table |
| `n` | Add a partition |
| `d` | Delete a partition |
| `i` | Show partition details |
| `l` | List partition types |
| `q` | Quit without saving |
| `w` | Write changes |

> [!alert]
> Do not open your active system disk in write mode unless your instructor explicitly provides a disposable practice disk.

---

# Part E - File Systems

## Task 13 - Identify Existing File-System Types

Run:

```bash
lsblk -f
```

Expected output includes columns such as:

```text
NAME
FSTYPE
FSVER
LABEL
UUID
MOUNTPOINTS
```

Record your observations:

| Device | File-system type | Mount point |
|---|---|---|
|  |  |  |
|  |  |  |

---

## Task 14 - Understand `mkfs`

The `mkfs` command creates a file system on a partition or block device.

For example, the original lab demonstrates a command in this form:

```bash
mkfs -t vfat /dev/sdb1
```

This would create a VFAT file system.

> [!alert]
> Do **not** run this command against an existing system partition.
>
> Formatting a partition destroys the existing file system and its data.

For this lab, display help only:

```bash
mkfs --help
```

You can also inspect available helpers:

```bash
ls /sbin/mkfs.*
```

---

# Part F - Mounting File Systems

## Task 15 - Display Currently Mounted File Systems

Run:

```bash
mount | head
```

Then:

```bash
findmnt
```

Review:

- source device;
- mount point;
- file-system type;
- mount options.

---

## Task 16 - Understand Mount Points

A mount point is a directory where a file system becomes accessible.

For example, a mount point can be created with:

```bash
sudo mkdir -p /mnt/mydisk
```

A formatted partition could then be mounted using a command such as:

```bash
sudo mount /dev/sdb1 /mnt/mydisk
```

> [!alert]
> Do not mount unknown or instructor-unapproved partitions.
>
> Only perform an actual mount if your instructor provides a designated practice partition.

---

## Task 17 - Inspect Mount Information

Run:

```bash
findmnt /
```

This displays the file system currently mounted as the Linux root directory.

Record:

| Item | Result |
|---|---|
| Source device | |
| File-system type | |
| Mount point | `/` |
| Mount options | |

---

# Part G - Inodes

## Task 18 - Understand Inodes

An **inode** stores metadata about a file-system object.

Typical inode information includes:

- file type;
- permissions;
- user and group ownership;
- file size;
- timestamps;
- pointers to data blocks.

The file name itself is associated through a directory entry rather than stored as the core identity of the inode.

---

## Task 19 - Create a Practice File

Return to your home directory:

```bash
cd ~
```

Create a practice directory:

```bash
mkdir -p lab04
```

Move into it:

```bash
cd lab04
```

Create a file:

```bash
echo "Linux file system lab" > example.txt
```

---

## Task 20 - Display the Inode Number

Run:

```bash
ls -i example.txt
```

Expected output looks similar to:

```text
123456 example.txt
```

The number at the beginning is the inode number.

Record:

```text
Inode number:
```

---

## Task 21 - Display Detailed Inode-Related Metadata

Run:

```bash
stat example.txt
```

Observe:

- size;
- blocks;
- inode;
- access permissions;
- UID;
- GID;
- access time;
- modification time;
- change time.

Complete:

| Item | Observed value |
|---|---|
| Size | |
| Inode | |
| Permissions | |
| Owner UID | |
| Group GID | |

---

# Part H - Disk Usage Analysis

## Task 22 - Check File-System Disk Usage with `df`

Run:

```bash
df -h
```

The `-h` option displays human-readable sizes.

Look for:

- total size;
- used space;
- available space;
- utilisation percentage;
- mount point.

Record the row containing:

```text
/
```

---

## Task 23 - Check Disk Usage for the Root File System

Run:

```bash
df -h /
```

Complete:

| Total size | Used | Available | Use % |
|---:|---:|---:|---:|
|  |  |  |  |

---

## Task 24 - Analyse Directory Usage with `du`

From your home directory:

```bash
cd ~
```

Run:

```bash
du -sh .
```

This displays the total disk space used by your home directory.

Now inspect the practice directory:

```bash
du -sh ~/lab04
```

---

## Task 25 - Compare `df` and `du`

Answer:

```text
What does df measure?
```

```text
What does du measure?
```

A useful distinction is:

- `df` reports file-system space usage.
- `du` reports space used by files and directories.

---

# Part I - File-System Checking

## Task 26 - Understand `fsck`

The `fsck` command checks file systems for consistency problems.

The original lab shows an example in the form:

```bash
fsck -f /dev/sdb1
```

> [!alert]
> Do not run `fsck` on a mounted production or system file system.
>
> File-system checks and repairs should normally be performed on an unmounted file system, or according to the procedure recommended for that file-system type.

For this lab, display the help information:

```bash
fsck --help
```

---

## Task 27 - Identify the Root File-System Type

Run:

```bash
findmnt -no FSTYPE /
```

Example:

```text
ext4
```

Record:

```text
Root file-system type:
```

---

# Part J - Validate Your Work

## Task 28 - Complete the Final Checklist

Confirm that you have:

- [ ] Identified the machine's block devices using `lsblk`.
- [ ] Inspected device files under `/dev`.
- [ ] Displayed partition information using `fdisk -l`.
- [ ] Identified the partition-table type.
- [ ] Explained the difference between MBR and GPT.
- [ ] Checked file-system types with `lsblk -f`.
- [ ] Examined mount information using `findmnt`.
- [ ] Created a practice file.
- [ ] Displayed its inode number.
- [ ] Used `stat` to inspect file metadata.
- [ ] Used `df` to inspect file-system space.
- [ ] Used `du` to inspect directory usage.
- [ ] Reviewed the purpose of `mkfs`.
- [ ] Reviewed the purpose of `fsck`.
- [ ] Avoided making destructive changes to the system disk.

If all items are complete, Lab 04 is complete.

---

# Troubleshooting

## Problem - `ls sd*` Returns No Files

Some systems use different device names, such as:

```text
nvme0n1
```

or:

```text
vda
```

Use:

```bash
lsblk
```

to identify the actual device names.

---

## Problem - `fdisk` Requires Permission

Use:

```bash
sudo fdisk -l
```

---

## Problem - `gdisk: command not found`

Run:

```bash
sudo apt update
sudo apt install gdisk -y
```

---

## Problem - You Are Unsure Which Disk Is Safe

Do not proceed with any modifying command.

Use read-only commands:

```bash
lsblk
```

```bash
lsblk -f
```

```bash
sudo fdisk -l
```

and ask your instructor before making any change.

---

## Problem - A Partition Is Mounted

Do not run destructive commands such as `mkfs` or repair operations on it.

Check mounted file systems using:

```bash
findmnt
```

---

# Knowledge Check

Answer the following questions:

1. What directory normally contains Linux device files?
2. What is the difference between `/dev/sda` and `/dev/sda1`?
3. What does `lsblk` display?
4. What is the purpose of `fdisk -l`?
5. What is the difference between MBR and GPT?
6. Why is `w` a dangerous command inside `fdisk`?
7. What does `mkfs` do?
8. What is a mount point?
9. What information is stored in an inode?
10. Which command displays an inode number?
11. What is the difference between `df` and `du`?
12. What is the purpose of `fsck`?
13. Why should `fsck` normally not be run against an actively mounted file system?
14. Why should a system administrator confirm the correct block device before formatting or partitioning?

---

# Challenge

Using **read-only commands only**, produce a short storage inventory of your Linux VM.

Include:

1. all disks;
2. their sizes;
3. partition names;
4. file-system types;
5. mount points;
6. root file-system utilisation;
7. root file-system type.

Suggested commands:

```bash
lsblk
```

```bash
lsblk -f
```

```bash
df -h /
```

```bash
findmnt /
```

Then write:

```text
Main disk:
Partition table type:
Root partition:
Root file-system type:
Root mount point:
Root file-system utilisation:
```

---

## Summary

In this lab, you:

- explored Linux block devices;
- inspected disk and partition information;
- reviewed MBR and GPT partitioning;
- learned how `fdisk` and `gdisk` are used;
- examined file-system types;
- reviewed how `mkfs` creates a file system;
- investigated mount points and mounted file systems;
- explored inode metadata;
- analysed storage usage using `df` and `du`;
- reviewed file-system checking with `fsck`;
- practised safe storage administration using read-only inspection commands.

You are now ready to work with more advanced Linux file-system and storage-management concepts.
