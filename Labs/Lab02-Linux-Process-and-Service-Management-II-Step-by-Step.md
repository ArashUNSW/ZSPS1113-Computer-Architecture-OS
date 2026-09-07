# Lab 02 - Linux Process and Service Management II

## Estimated Time

60–75 minutes

## Learning Objectives

By the end of this lab, you will be able to:

- Explain how Linux creates a child process using the `fork()` system call.
- Compile and run a simple C program that creates parent and child processes.
- Inspect running processes through the `/proc` virtual filesystem.
- Explain how uncontrolled process creation can exhaust system resources.
- View and apply process limits using `ulimit`.
- Monitor processes using `top` and `strace`.
- Explain Linux process priority using **PR** and **NI** values.
- Start a program with a selected nice value using `nice`.
- Change the priority of a running process using `renice`.
- Run processes in the background and manage shell jobs.
- Create a simple Bash service script.
- Create and manage a basic `systemd` service.

---

## Scenario

You are continuing your Linux system administration training.

In the previous lab, you learned how to list, search, monitor, and terminate processes. In this lab, you will explore more advanced process-management concepts, including process creation, resource limits, process priorities, background jobs, and Linux services.

You will also create a small `systemd` service and practise starting, stopping, restarting, enabling, and disabling it.

> [!note]
> This lab is for **self-assessment and practical learning**. No submission is required unless your instructor advises otherwise.

---

## Before You Begin

You need:

- A Linux virtual machine.
- A normal user account with `sudo` access.
- A terminal application.
- The GNU C compiler (`gcc`).
- The `nano` text editor.
- A Linux distribution using `systemd`.

Open a terminal before starting.

---

# Part A - Process Creation

## Task 1 - Confirm Your Linux Environment

In the terminal, run:

```bash
whoami
```

Then run:

```bash
pwd
```

Expected result:

```text
Your username and current working directory should display.
```

> [!hint]
> You will create the files for this lab inside your home directory.

---

## Task 2 - Check Whether GCC Is Installed

Run:

```bash
gcc --version
```

Expected result:

```text
The installed GCC version should display.
```

If GCC is not installed on an Ubuntu/Debian-based system, run:

```bash
sudo apt update
sudo apt install build-essential -y
```

Then check again:

```bash
gcc --version
```

---

## Task 3 - Create a C Program Using `fork()`

Create a new source-code file:

```bash
nano prfork.c
```

Enter the following code:

```c
#include <stdlib.h>
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

int main(void) {
    pid_t pid;

    pid = fork();

    if (pid == 0) {
        printf("Child process! PID=%d, Parent PID=%d\n",
               getpid(), getppid());
    } else if (pid > 0) {
        printf("Parent process! PID=%d, Child PID=%d\n",
               getpid(), pid);
    } else {
        perror("fork");
        return EXIT_FAILURE;
    }

    return EXIT_SUCCESS;
}
```

Save the file:

```text
Ctrl + O
```

Press:

```text
Enter
```

Exit `nano`:

```text
Ctrl + X
```

> [!note]
> `fork()` creates a new child process. After a successful fork:
>
> - the **child process receives `0`** as the return value;
> - the **parent process receives the PID of the child**;
> - a **negative value** indicates that process creation failed.

---

## Task 4 - Compile the Program

Compile the program:

```bash
gcc prfork.c -o prfork
```

Expected result:

```text
No error messages should appear.
```

Confirm that the executable was created:

```bash
ls -l prfork
```

---

## Task 5 - Run the Program

Run:

```bash
./prfork
```

You should see output similar to:

```text
Parent process! PID=4210, Child PID=4211
Child process! PID=4211, Parent PID=4210
```

Your PID values will be different.

> [!note]
> The order of the parent and child messages can vary because the Linux scheduler decides which process runs first.

Run the program several times:

```bash
./prfork
./prfork
./prfork
```

Observe how the PID values change.

---

## Task 6 - Inspect the Source Process Relationship

Run:

```bash
ps -ef | grep prfork
```

Because the example program exits very quickly, you may not see it in the process list.

This demonstrates an important idea:

```text
Some processes exist for only a very short period of time.
```

---

# Part B - The `/proc` Virtual Filesystem

## Task 7 - Explore `/proc`

Linux exposes information about running processes through the `/proc` virtual filesystem.

Run:

```bash
ls /proc | head
```

Now display your shell PID:

```bash
echo $$
```

Example:

```text
3521
```

Check whether a directory with that PID exists:

```bash
ls /proc/$$
```

Expected result:

```text
A directory containing information about the current shell process should display.
```

---

## Task 8 - Examine Process Information

Display information about your current shell process:

```bash
cat /proc/$$/status
```

Look for fields such as:

```text
Name
State
Pid
PPid
Uid
Threads
VmSize
VmRSS
```

Now display the command line used for the process:

```bash
tr '\0' ' ' < /proc/$$/cmdline
echo
```

> [!note]
> Linux creates a `/proc/<PID>` directory for each active process. The directory disappears after the process terminates.

---

# Part C - Resource Exhaustion and Process Limits

## Task 9 - Understand a Fork-Bomb Attack

A **fork bomb** is a denial-of-service technique in which a program repeatedly creates new processes until available system resources are exhausted.

This can lead to:

- very high CPU usage;
- memory exhaustion;
- inability to start new programs;
- an unresponsive virtual machine;
- forced restart of the operating system.

> [!alert]
> **Do not execute a fork bomb in this lab.**
>
> An unrestricted fork bomb can make the Linux virtual machine unusable and may affect shared laboratory infrastructure. The purpose of this section is to understand the concept and practise defensive controls.

Conceptually, the behaviour is:

```text
Process
  ├── Child
  │    ├── Child
  │    └── Child
  └── Child
       ├── Child
       └── Child
```

The number of processes can grow extremely quickly if there is no stopping condition.

---

## Task 10 - Check Your Current Process Limit

Run:

```bash
ulimit -u
```

This displays the maximum number of processes available to your current user session.

Example:

```text
15472
```

Your value may be different.

---

## Task 11 - View All Current Shell Limits

Run:

```bash
ulimit -a
```

Review the displayed resource limits.

Look for the entry related to:

```text
max user processes
```

> [!note]
> Resource limits are one defensive mechanism that can reduce the effect of uncontrolled process creation.

---

## Task 12 - Apply a Temporary Process Limit

Before changing the value, record the current limit:

```bash
ulimit -u
```

For this lab, you may set a temporary limit for the current shell:

```bash
ulimit -u 100
```

Verify it:

```bash
ulimit -u
```

Expected result:

```text
100
```

> [!alert]
> Do not set the value unnecessarily low. A very low limit can prevent normal applications from starting.
>
> This change applies to the current shell and processes launched from it.

Close the terminal when you have finished this section if you want to return to the normal session configuration.

---

## Task 13 - Review PAM Process Limits

Linux can also enforce process limits through PAM.

View the configuration file:

```bash
sudo nano /etc/security/limits.conf
```

Read the comments in the file.

A configuration can use entries in this general form:

```text
<domain>    <type>    <item>    <value>
```

For example:

```text
student    soft    nproc    50
student    hard    nproc    100
```

> [!alert]
> Do **not** modify the shared lab machine's limits unless your instructor specifically asks you to do so.

Exit without saving:

```text
Ctrl + X
```

---

# Part D - Process Monitoring

## Task 14 - Monitor Processes with `top`

Run:

```bash
top
```

Observe the dynamically updating process list.

Look for the following columns:

```text
PID
USER
PR
NI
VIRT
RES
S
%CPU
%MEM
TIME+
COMMAND
```

Press:

```text
q
```

to exit `top`.

---

## Task 15 - Trace System Calls with `strace`

Check whether `strace` is installed:

```bash
strace --version
```

If required on Ubuntu/Debian:

```bash
sudo apt install strace -y
```

Trace a simple command:

```bash
strace cp --help
```

You will see many system calls.

A shorter example is:

```bash
strace -e trace=openat,read,write ls
```

Expected result:

```text
The terminal should display selected system calls used while executing ls.
```

> [!note]
> `strace` is useful for troubleshooting because it shows how a process interacts with the Linux kernel.

---

# Part E - Process Priority

## Task 16 - Understand `PR` and `NI`

Run:

```bash
top
```

Find the columns:

```text
PR
NI
```

- **PR** represents the process scheduling priority reported by the kernel.
- **NI** represents the user-space **nice value**.

For normal processes, nice values normally range from:

```text
-20 to 19
```

A lower nice value generally gives a process a higher scheduling preference.

Press:

```text
q
```

to exit.

---

## Task 17 - Start a Process with `nice`

Start a simple process with a nice value of `10`:

```bash
nice -n 10 sleep 120 &
```

The shell should display a job number and PID similar to:

```text
[1] 5324
```

Find the process:

```bash
ps -o pid,ppid,ni,stat,cmd -C sleep
```

Expected result:

```text
The sleep process should show a nice value of 10.
```

---

## Task 18 - Change Priority with `renice`

Find the PID:

```bash
pgrep -n sleep
```

Store it in a shell variable:

```bash
SLEEP_PID=$(pgrep -n sleep)
```

Display it:

```bash
echo $SLEEP_PID
```

Change the nice value to `15`:

```bash
renice -n 15 -p "$SLEEP_PID"
```

Verify:

```bash
ps -o pid,ni,cmd -p "$SLEEP_PID"
```

Expected result:

```text
The NI value should now be 15.
```

> [!note]
> Normal users can usually make their own processes **less favourable** to the scheduler by increasing the nice value. Increasing priority by selecting a lower nice value may require elevated privileges.

---

# Part F - Background Jobs

## Task 19 - Start a Background Process

Run:

```bash
sleep 300 &
```

The `&` symbol starts the command in the background.

The shell should immediately return to the prompt.

Display active shell jobs:

```bash
jobs
```

Expected result:

```text
A running sleep job should appear.
```

---

## Task 20 - Stop and Resume a Job

Start:

```bash
sleep 300
```

While it is running, press:

```text
Ctrl + Z
```

The process should become **stopped**.

Display the jobs:

```bash
jobs
```

Expected result:

```text
The sleep process should be shown as Stopped.
```

Resume it in the background:

```bash
bg
```

Check again:

```bash
jobs
```

---

## Task 21 - Bring a Background Job to the Foreground

Display the jobs:

```bash
jobs
```

If the sleep command is job `1`, run:

```bash
fg %1
```

The process returns to the foreground.

Press:

```text
Ctrl + C
```

to terminate it.

> [!hint]
> Your job number may not be `1`. Use the number shown by the `jobs` command.

---

# Part G - Create a Simple Linux Service

## Task 22 - Create the Service Script

Move to your home directory:

```bash
cd ~
```

Create the script:

```bash
nano myservice.sh
```

Enter:

```bash
#!/bin/bash

echo "Hello, this is my service."
sleep 10
echo "Service execution complete."
```

Save and exit.

Make the script executable:

```bash
chmod +x ~/myservice.sh
```

Test it:

```bash
~/myservice.sh
```

Expected result:

```text
Hello, this is my service.
Service execution complete.
```

---

## Task 23 - Find the Full Script Path

Run:

```bash
realpath ~/myservice.sh
```

Example:

```text
/home/student/myservice.sh
```

Copy or note your displayed path.

You will use this path in the service unit file.

---

## Task 24 - Create a `systemd` Service Unit

Create the unit file:

```bash
sudo nano /etc/systemd/system/myservice.service
```

Enter:

```ini
[Unit]
Description=My Custom Service
After=network.target

[Service]
Type=oneshot
ExecStart=/home/student/myservice.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

> [!alert]
> Replace:
>
> ```text
> /home/student/myservice.sh
> ```
>
> with the exact path returned by:
>
> ```bash
> realpath ~/myservice.sh
> ```

Save and exit.

---

## Task 25 - Reload the `systemd` Configuration

Whenever you create or modify a unit file, reload the service manager:

```bash
sudo systemctl daemon-reload
```

Expected result:

```text
No error messages should appear.
```

---

## Task 26 - Start the Service

Run:

```bash
sudo systemctl start myservice
```

Check its status:

```bash
systemctl status myservice --no-pager
```

Look for information including:

```text
Loaded
Active
ExecStart
```

Because this example is a short `oneshot` service, it completes after approximately 10 seconds.

---

## Task 27 - View the Service Log

Run:

```bash
journalctl -u myservice --no-pager -n 20
```

You should find messages associated with the service execution.

---

## Task 28 - Stop the Service

Run:

```bash
sudo systemctl stop myservice
```

Check:

```bash
systemctl status myservice --no-pager
```

---

## Task 29 - Enable the Service at Boot

Enable the unit:

```bash
sudo systemctl enable myservice
```

Check:

```bash
systemctl is-enabled myservice
```

Expected result:

```text
enabled
```

---

## Task 30 - Restart the Service

Run:

```bash
sudo systemctl restart myservice
```

Then check:

```bash
systemctl status myservice --no-pager
```

---

## Task 31 - Disable the Service

Disable automatic startup:

```bash
sudo systemctl disable myservice
```

Verify:

```bash
systemctl is-enabled myservice
```

Expected result:

```text
disabled
```

---

# Part H - Clean Up

## Task 32 - Remove the Practice Service

Stop and disable the service:

```bash
sudo systemctl stop myservice
sudo systemctl disable myservice
```

Remove the unit file:

```bash
sudo rm /etc/systemd/system/myservice.service
```

Reload `systemd`:

```bash
sudo systemctl daemon-reload
```

Remove the practice script if you no longer need it:

```bash
rm ~/myservice.sh
```

Remove the compiled fork example if required:

```bash
rm -f ~/prfork ~/prfork.c
```

---

## Task 33 - Validate Your Work

Before cleanup, you should have successfully demonstrated the following:

- [ ] Compiled a C program with `gcc`.
- [ ] Used `fork()` to create a parent and child process.
- [ ] Examined process information under `/proc`.
- [ ] Checked user process limits with `ulimit`.
- [ ] Used `top` to monitor processes.
- [ ] Used `strace` to observe system calls.
- [ ] Started a process using `nice`.
- [ ] Changed a running process priority using `renice`.
- [ ] Used `&`, `jobs`, `bg`, and `fg`.
- [ ] Created an executable Bash service script.
- [ ] Created a `systemd` unit file.
- [ ] Used `systemctl` to start and inspect the service.
- [ ] Enabled and disabled the service.
- [ ] Viewed service logs using `journalctl`.

If you completed these items, Lab 02 is complete.

---

# Troubleshooting

## Problem: `gcc: command not found`

On Ubuntu/Debian-based systems, run:

```bash
sudo apt update
sudo apt install build-essential -y
```

---

## Problem: `strace: command not found`

Run:

```bash
sudo apt install strace -y
```

---

## Problem: Permission denied when running `myservice.sh`

Make the script executable:

```bash
chmod +x ~/myservice.sh
```

---

## Problem: `systemctl` says the service cannot be found

Reload `systemd`:

```bash
sudo systemctl daemon-reload
```

Then try:

```bash
sudo systemctl start myservice
```

---

## Problem: Service fails to start

Check the status:

```bash
systemctl status myservice --no-pager
```

Then inspect the log:

```bash
journalctl -u myservice --no-pager -n 30
```

Check that the `ExecStart` path in the unit file is correct:

```bash
realpath ~/myservice.sh
```

---

## Problem: `renice` reports permission denied

A normal user may not be allowed to increase a process's scheduling priority.

For this lab, use a **larger** nice value, for example:

```bash
renice -n 15 -p <PID>
```

---

# Knowledge Check

Answer the following questions:

1. What is the purpose of the `fork()` system call?
2. What return value does the child receive from `fork()`?
3. What does the parent receive from a successful `fork()`?
4. What information is stored under `/proc/<PID>`?
5. Why can uncontrolled process creation cause a denial of service?
6. What command displays the maximum number of processes available to a user?
7. What is the purpose of the `nice` command?
8. What is the difference between `nice` and `renice`?
9. What does the `&` symbol do when placed after a command?
10. What command lists the jobs managed by the current shell?
11. What is the purpose of a `systemd` unit file?
12. What command must normally be run after creating or editing a unit file?
13. What is the difference between `systemctl start` and `systemctl enable`?
14. Where can you view logs generated by a `systemd` service?

---

# Challenge

Complete the following without copying the commands from the earlier tasks.

1. Start a `sleep` process for 180 seconds in the background.
2. Find its PID.
3. Display its current nice value.
4. Change its nice value to `12`.
5. Confirm the new value.
6. Bring the process to the foreground.
7. Terminate it safely.

Then answer:

```text
Which commands did you use, and what did each command do?
```

---

## Summary

In this lab, you:

- created parent and child processes using `fork()`;
- compiled and executed a C program;
- explored process information through `/proc`;
- examined user process limits;
- reviewed how uncontrolled process creation can exhaust resources;
- monitored Linux processes with `top` and `strace`;
- worked with Linux process priorities using `nice` and `renice`;
- managed foreground and background jobs;
- created a basic Bash service;
- created a `systemd` service unit;
- started, stopped, restarted, enabled, and disabled the service;
- inspected service status and logs.

You are now ready to work with more advanced Linux process and service-management concepts.
