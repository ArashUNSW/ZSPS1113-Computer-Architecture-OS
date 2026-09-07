# Lab 01 - Linux Process Management I

## Learning Objectives

By the end of this lab, you will be able to:

- Identify running processes in a Linux system.
- Use `ps` to list processes and interpret common process fields.
- Recognise common Linux process states.
- Search for processes using `pgrep`.
- Monitor processes in real time using `top` and `watch`.
- Observe system calls using `strace`.
- Display process relationships using `pstree`.
- Terminate processes safely using `kill`, `pkill`, and `killall`.
- Identify common Linux process signals such as `SIGTERM`, `SIGKILL`, and `SIGSTOP`.

---

## Lab Status

This lab is for **self-assessment and practice**.

- The lab is **not graded**.
- **Submission is not required**.
- Complete the knowledge-check questions at the end to confirm your understanding.

---

## Scenario

You are working with a Linux system and need to understand how applications and background services are represented as processes.

In this lab, you will start several applications, inspect their processes, search for them, monitor resource usage, examine the process hierarchy, and practise terminating selected processes using standard Linux commands.

> [!note]
> Process IDs and command output will be different on each Linux machine. Use the values shown on your own system.

---

## Task 1 - Log In and Prepare the Linux Environment

1. Log in to your Linux virtual machine using the username and password provided for the lab.
2. Open the **Calculator** application and leave it running.
3. Open the **Calendar** application and leave it running.
4. Open a web browser.
5. Visit:

```text
www.google.com
```

6. Leave the browser running.
7. Open a **Terminal** window.

You should now have several applications running at the same time.

---

## Task 2 - List All Running Processes

In the terminal, run:

```bash
ps aux
```

The command displays information about running processes.

Look at several rows and identify the following fields:

- `PID` - Process ID
- `USER` - User that owns the process
- `%CPU` - CPU usage
- `%MEM` - Memory usage
- `COMMAND` - Command or program associated with the process

> [!note]
> Depending on the command and Linux distribution, additional fields such as `TTY`, `STAT`, `START`, `TIME`, `VSZ`, and `RSS` may also be displayed.

### Your Task

Choose three running processes and record their details.

| Process | PID | USER | %CPU | %MEM | COMMAND |
|---|---:|---|---:|---:|---|
| 1 |  |  |  |  |  |
| 2 |  |  |  |  |  |
| 3 |  |  |  |  |  |

---

## Task 3 - Use the Basic `ps` Command

Run:

```bash
ps
```

Typical output may look similar to:

```text
PID TTY          TIME CMD
80  ?        00:00:00 bash
94  ?        00:00:00 ps
```

The exact values on your system will be different.

The main columns are:

| Column | Description |
|---|---|
| `PID` | A unique number assigned to each process. |
| `TTY` | The terminal or pseudo-terminal associated with the process. |
| `TIME` | The cumulative CPU time used by the process. |
| `CMD` | The command used to start the process. |

---

## Task 4 - List Processes with `ps x`

Run:

```bash
ps x
```

This displays your processes, including processes that may not be attached to the current terminal.

Typical output contains fields such as:

```text
PID TTY      STAT   TIME COMMAND
```

Locate the `STAT` column.

Common process states include:

| State | Description |
|---|---|
| `D` | Uninterruptible sleep |
| `R` | Running |
| `S` | Interruptible sleep |
| `T` | Stopped |
| `Z` | Zombie |

### Your Task

Find at least one process with an `R` or `S` state and record its PID and command.

```text
PID:
State:
Command:
```

---

## Task 5 - Review `ps aux`

Run the following command again:

```bash
ps aux
```

You should see a larger list of processes belonging to different users and services on the system.

A typical header may look like:

```text
USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND
```

> [!hint]
> The list may be long. You can scroll in the terminal to review earlier output.

### Your Task

Try to locate the Calculator, Calendar, or browser process that you opened earlier.

---

## Task 6 - Search for a Process with `pgrep`

The `pgrep` command searches the active process list for names that match the value you provide.

First, try:

```bash
pgrep -i init
```

If the process exists, one or more process IDs may be displayed.

To display both the PID and process name, use the `-l` option. For example:

```bash
pgrep -li sshd
```

If `sshd` is running, output may be similar to:

```text
1234 sshd
```

If no matching process exists, the command may return no output.

### Your Task

Search for one application that you currently have open. For example, try a browser or calculator process name.

```bash
pgrep -li <process-name>
```

Record the result:

```text
Process searched:
PID found:
```

---

## Task 7 - Monitor Processes in Real Time with `top`

Run:

```bash
top
```

The display updates continuously and shows information such as:

- running processes
- CPU usage
- memory usage
- process state
- process owner
- process command

Look for the Calculator process.

If it is difficult to see, click the Calculator application and perform an action such as entering a number. Then return to `top` and observe whether its CPU usage changes.

To exit `top`, press:

```text
q
```

### Your Task

Record one process with noticeable CPU activity.

```text
Process:
PID:
%CPU:
%MEM:
```

---

## Task 8 - Observe System Calls with `strace`

The `strace` command can show system calls made by a command or process.

Try:

```bash
strace cp
```

You can also use `strace` with a script or command, for example:

```bash
strace <myscript.sh>
```

> [!note]
> The command may produce a large amount of output. The purpose of this task is to observe that normal programs interact with the operating system through system calls.

> [!alert]
> If `strace` is not installed in the lab environment, continue to the next task.

---

## Task 9 - Repeatedly Run a Command with `watch`

The `watch` command repeatedly executes another command and refreshes the result.

Run:

```bash
watch date
```

By default, `watch` refreshes approximately every two seconds.

Observe the displayed date and time changing.

To exit, press:

```text
Ctrl + C
```

---

## Task 10 - Monitor the Process List with `watch`

Run:

```bash
watch ps aux
```

The process list will refresh repeatedly.

1. Keep the command running.
2. Start or close an application.
3. Observe how the process list changes.
4. Press **Ctrl + C** to stop `watch`.

### Your Task

Write down one change you observed in the process list.

```text
Observation:
```

---

## Task 11 - Display the Process Hierarchy

Run:

```bash
pstree
```

The output displays processes as a tree, showing parent-child relationships.

A simplified example may look similar to:

```text
init-+-cron
     |-login---bash-+-pstree
     |              `-top
     |-rsyslogd
     `-sshd
```

To include process IDs and user information for a particular user, the original lab introduces:

```bash
pstree -p -u <username>
```

For example:

```bash
pstree -p -u user01
```

To review usernames configured on the system, run:

```bash
cat /etc/passwd
```

### Your Task

Identify one parent process and one of its child processes.

```text
Parent process:
Child process:
```

---

## Task 12 - Terminate a Process by PID with `kill`

Choose a non-essential process that you started during this lab, such as Calculator or Calendar.

> [!alert]
> Do not terminate system processes or services that you did not start yourself.

First, locate its PID using:

```bash
ps aux
```

or:

```bash
pgrep -li <process-name>
```

Then terminate the process using:

```bash
kill <PID>
```

Example:

```bash
kill 2268
```

Confirm that the process has stopped:

```bash
ps aux
```

or:

```bash
pgrep -li <process-name>
```

Expected result:

```text
The selected process should no longer be running.
```

---

## Task 13 - Review Common Process Signals

Linux uses signals to control processes.

Common signals introduced in this lab are:

| Signal | Value | Purpose |
|---|---:|---|
| `SIGINT` | 2 | Interrupt a process from the keyboard, similar to `Ctrl+C`. |
| `SIGQUIT` | 3 | Quit from the keyboard and may produce a core dump. |
| `SIGKILL` | 9 | Immediately terminates a process. |
| `SIGTERM` | 15 | Requests graceful termination. |
| `SIGCONT` | 18 | Resumes a stopped process. |
| `SIGSTOP` | 19 | Pauses a process. |

To list the signals available on the system, run:

```bash
kill -l
```

You should see a numbered list containing signal names such as:

```text
SIGHUP
SIGINT
SIGQUIT
SIGKILL
SIGTERM
```

---

## Task 14 - Send a Specific Signal with `kill`

To send a signal to a process, use:

```bash
kill <signal> <PID>
```

For example, a forceful termination using signal 9 is:

```bash
kill -9 <PID>
```

> [!alert]
> Use `SIGKILL` only when necessary. For normal termination, first try the standard `kill <PID>` command.

### Your Task

Start Calculator again, identify its PID, and terminate it using the normal `kill <PID>` command.

Do not use `kill -9` unless your instructor specifically asks you to test it.

---

## Task 15 - Terminate Processes by Name with `pkill`

The `pkill` command can send signals to processes based on their names or other criteria.

To terminate processes by name:

```bash
pkill firefox
```

To send `SIGTERM` explicitly:

```bash
pkill -SIGTERM python
```

To target a process owned by a specific user:

```bash
pkill -u ben ssh
```

To use a case-insensitive name match:

```bash
pkill -i chrome
```

> [!alert]
> Commands such as `pkill firefox`, `pkill python`, or `pkill -i chrome` may terminate more than one matching process. Only run them when you understand which processes will be affected.

---

## Task 16 - Terminate Processes with `killall`

The `killall` command terminates processes that match a process name.

Example:

```bash
killall gedit
```

To send `SIGKILL`:

```bash
killall -9 gedit
```

To terminate processes owned by a particular user:

```bash
killall -u bob firefox
```

To wait until matching processes have stopped before returning to the terminal:

```bash
killall -w gedit
```

> [!alert]
> Be careful when using `killall`. A process name may match multiple running instances.

---

## Task 17 - Validate Your Work

Before finishing the lab, confirm that you can complete each of the following actions.

- [ ] Run `ps` and explain the `PID`, `TTY`, `TIME`, and `CMD` fields.
- [ ] Run `ps x` and identify a process state.
- [ ] Run `ps aux` and locate an application process.
- [ ] Search for a process using `pgrep`.
- [ ] Monitor running processes using `top`.
- [ ] Use `watch` to repeatedly run a command.
- [ ] Display the process hierarchy using `pstree`.
- [ ] Use `kill` to terminate a process you started.
- [ ] Explain the difference between `SIGTERM` and `SIGKILL`.
- [ ] Explain what `pkill` and `killall` do.

---

## Troubleshooting

### Problem: `pgrep` returns no output

The process name may be different from the application name shown on the desktop.

Run:

```bash
ps aux
```

Then inspect the `COMMAND` column and try again with the process name you find.

---

### Problem: `top` fills the entire terminal

This is expected. `top` is an interactive process monitor.

Press:

```text
q
```

To exit.

---

### Problem: `watch` keeps refreshing

This is expected.

Press:

```text
Ctrl + C
```

To stop it.

---

### Problem: `strace: command not found`

The `strace` package may not be installed in the lab image.

If software installation is not permitted, skip the `strace` task and continue with the remaining activities.

---

### Problem: `kill` says `No such process`

The process may already have exited, or the PID may have changed.

Run:

```bash
ps aux
```

or:

```bash
pgrep -li <process-name>
```

Then use the current PID.

---

### Problem: Permission denied when terminating a process

You may be trying to terminate a process owned by another user or a system service.

For this lab, only terminate applications and processes that you started yourself.

---

## Knowledge Check

The following questions are added as a structured self-assessment based on the lab activities.

1. What does `PID` mean, and why is it useful?
2. What is the difference between `ps` and `ps aux`?
3. What does the `STAT` field show in `ps x` output?
4. Which process state represents a running process?
5. Which command can search for processes by name?
6. Which command displays processes continuously in real time?
7. What is the purpose of the `watch` command?
8. What does `pstree` show?
9. What is the difference between `SIGTERM` and `SIGKILL`?
10. What is the difference between `kill`, `pkill`, and `killall`?

---

## Optional Challenge

1. Open Calculator and a browser.
2. Use `pgrep` to find the PID of each application.
3. Use `pstree -p` to see whether you can identify their parent processes.
4. Run `top` and compare their CPU and memory usage.
5. Terminate only the Calculator using `kill <PID>`.
6. Confirm that the browser is still running.

Write a short answer:

```text
Why is terminating a process by PID sometimes safer than terminating processes by name?
```

---

## Summary

In this lab, you practised Linux process management by:

- starting applications and locating their processes;
- listing processes using `ps`, `ps x`, and `ps aux`;
- identifying common process states;
- searching for processes with `pgrep`;
- monitoring process activity using `top` and `watch`;
- observing system calls using `strace`;
- displaying process relationships using `pstree`;
- reviewing Linux process signals;
- terminating processes with `kill`, `pkill`, and `killall`.

You are now ready to continue with more advanced Linux process-management activities.
