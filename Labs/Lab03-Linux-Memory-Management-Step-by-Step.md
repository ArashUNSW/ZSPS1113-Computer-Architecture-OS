# Lab 03 - Exploring Linux Memory Management with Python

**Suggested practical time:** 2-3 hours, depending on VM performance and the assessment scripts supplied by your instructor.

## Learning Objectives

By the end of this lab, you should be able to:

- Inspect physical memory, available memory, and swap using Linux commands.
- Use Python and `psutil` to report and monitor memory usage.
- Compare virtual memory size (**VSZ**) with resident memory (**RSS**) for running programs.
- Examine process memory mappings through `/proc/<PID>/maps` and Python's `mmap` module.
- Observe a simple Python memory-fragmentation demonstration.
- Monitor memory pressure, paging indicators, and disk activity without overloading the lab VM.
- Experiment with a temporary virtual-memory limit.
- Simulate least recently used (**LRU**) page replacement.
  
---

## Scenario

You are investigating how a Linux operating system allocates, tracks, and reuses memory. You will compare information reported by Linux with measurements taken from Python programs. Your job is to record **what your VM actually does**, explain differences using the program code, and identify whether CPU, memory, or disk I/O is the main bottleneck in instructor-provided programs.


## Before You Begin

You need a Linux VM with a terminal, Python 3, permission to install packages if necessary, and enough available memory for the selected demonstrations. `htop` and `iostat` are optional tools. Do not attempt the larger memory-allocation exercise on a low-memory or shared machine without checking its available RAM.

## Part A - Inspect System Memory

### Task 1 - Log In and Create a Lab Folder

1. Log in to your Linux VM using the credentials supplied by your instructor.
2. Open a terminal and create a folder for your code and observations:

```bash
mkdir -p ~/lab03-memory
cd ~/lab03-memory
pwd
```

**Expected result:** The terminal shows your `lab03-memory` directory.

### Task 2 - Check Memory with `free`

Run:

```bash
free -h
```

Locate **total**, **used**, **free**, **buff/cache**, **available**, and **Swap**. The `-h` option displays human-readable units.

| Measurement | Your VM's result |
|---|---|
| Total physical memory | |
| Used memory | |
| Available memory | |
| Total swap | |
| Used swap | |

> [!hint] Linux may use otherwise idle RAM for cache. `free` and `available` are not interchangeable: `available` estimates memory that can be given to new applications without swapping.

### Task 3 - Read `/proc/meminfo`

Run:

```bash
cat /proc/meminfo | head -20
```

Look for `MemTotal`, `MemFree`, `MemAvailable`, and any swap-related fields displayed. Values here are typically reported in **kB**.

**Expected result:** Detailed Linux memory counters. Compare them with `free -h`; remember that the tools may display different units and memory may change between commands.

### Task 4 - Examine `vmstat` Counters

Run:

```bash
vmstat -s
```

Find the counters for total memory, free memory, used memory, and swap. Record which information appears in this output that is not obvious from `free -h`.

**Checkpoint:** Explain in one or two sentences why all three commands can report slightly different-looking values without contradicting each other.

---

## Part B - Inspect Memory Using Python

### Task 5 - Install or Verify `psutil`

First check that Python is available:

```bash
python3 --version
```

Try to import `psutil`:

```bash
python3 -c 'import psutil; print(psutil.__version__)'
```

If it is not installed, on Ubuntu/Debian-based lab VMs run:

```bash
sudo apt update
sudo apt install python3-psutil -y
```

Check the import again. Alternatively, if your instructor uses a Python virtual environment, install `psutil` **inside that environment** using `python -m pip install psutil`.

> [!note] The PDF also mentions `pip install psutil`. On managed Linux installations, a virtual environment or the distribution package may be necessary.

### Task 6 - Create `mem_info.py`

Run:

```bash
nano mem_info.py
```

Enter:

```python
import psutil

memory = psutil.virtual_memory()

print("Total Memory:", memory.total / (1024 ** 3), "GiB")
print("Available Memory:", memory.available / (1024 ** 3), "GiB")
print("Used Memory:", memory.used / (1024 ** 3), "GiB")
print("Memory Percentage Used:", memory.percent, "%")
```

Save using **Ctrl+O**, press **Enter**, then exit with **Ctrl+X**. Run:

```bash
python3 mem_info.py
```

**Expected result:** Four lines showing the VM's current memory totals and percentage used. The numbers depend on your environment.

> [!note] The original PDF asks about `1024 * 3`, but the code actually uses **`1024 ** 3`**. The latter means 1024 cubed and converts bytes to **gibibytes (GiB)**. Multiplication by three would not perform that conversion.

**Checkpoint:** Compare your Python output with `free -h`. Do the total and available memory values approximately agree after accounting for units and timing?

---

## Part C - Virtual and Resident Memory

### Task 7 - Create the First Allocation Program

Create `memory1.py`:

```bash
nano memory1.py
```

Enter the demonstration from the original lab:

```python
import time

big_list = [0] * (10 ** 7)
time.sleep(30)
```

This makes a large list whose entries initially refer to the same integer object. Save the file.

### Task 8 - Create the Second Allocation Program

Create `memory2.py`:

```bash
nano memory2.py
```

Enter:

```python
import time

big_list = [i for i in range(10 ** 7)]
time.sleep(30)
```

Save the file.

> [!alert] The second program can consume substantially more physical memory because it constructs many distinct Python integer objects. If your VM is memory-constrained, change **both** programs to `10 ** 6` and record that you used a smaller, equal input size. Do not run this test while another memory-intensive exercise is active.

### Task 9 - Run and Compare the Two Programs

Run the first program and record its PID:

```bash
python3 memory1.py &
PID1=$!
sleep 5
ps -o pid,vsz,rss,%mem,cmd -p "$PID1"
wait "$PID1"
```

Run the second program **after the first has finished**:

```bash
python3 memory2.py &
PID2=$!
sleep 5
ps -o pid,vsz,rss,%mem,cmd -p "$PID2"
wait "$PID2"
```

If the process has already exited when you inspect it, rerun it and execute the `ps` command promptly. You can also watch processes in another terminal using `top`.

| Program | VSZ (KiB) | RSS (KiB) | %MEM | Observation |
|---|---:|---:|---:|---|
| `memory1.py` | | | | |
| `memory2.py` | | | | |

**What to look for:** `VSZ` is virtual address-space size; `RSS` is resident physical memory. Explain the measured difference by referring to **how each list is created** rather than assuming every reserved byte must be resident in RAM.

---

## Part D - Paging and Memory Mappings

### Task 10 - Inspect a Running Process's Memory Map

Start the first program again:

```bash
python3 memory1.py &
PID=$!
echo "Process PID: $PID"
```

While it is still running, inspect its memory mappings:

```bash
head -30 /proc/$PID/maps
```

Find the heap and stack if present:

```bash
grep -E '\[heap\]|\[stack\]' /proc/$PID/maps
```

Finally:

```bash
wait "$PID"
```

**Expected result:** Address ranges, permissions, and mapped files or regions. Locate `[heap]`, `[stack]`, and other segments if they appear in your output. If the process finishes too quickly, temporarily increase its `sleep` duration.

### Task 11 - Create and Inspect a Memory-Mapped File

Create `mmap_demo.py`:

```bash
nano mmap_demo.py
```

Enter:

```python
import mmap

size = 1024 * 1024  # 1 MiB

with open("mmapfile", "wb") as file:
    file.write(b"\x00" * size)

with open("mmapfile", "r+b") as file:
    mm = mmap.mmap(file.fileno(), 0)
    mm[0:10] = b"ABCDEFGHIJ"
    input("Press Enter to exit...")
    mm.close()
```

Save the file. In **Terminal 1**, run:

```bash
python3 mmap_demo.py
```

Leave the program at the prompt. In **Terminal 2**, run:

```bash
pgrep -af mmap_demo.py
```

Identify the correct PID and inspect its map:

```bash
cat /proc/<PID>/maps
```

Replace `<PID>` with the actual number; do not type the angle brackets. Look for an entry associated with `mmapfile`. Return to Terminal 1 and press **Enter** to finish.

**Checkpoint:** How is a file-backed memory mapping shown in `/proc/<PID>/maps`?

---

## Part E - Fragmentation, Swap Indicators, and Limits

### Task 12 - Run the Fragmentation Demonstration

Create `fragment.py`:

```bash
nano fragment.py
```

Enter:

```python
import time

blocks = []
for i in range(1000):
    blocks.append(bytearray(4096))  # Allocate 4 KiB
    if i % 2 == 0:
        blocks[i] = None           # Release alternate blocks

time.sleep(30)
```

Run:

```bash
python3 fragment.py &
FRAGMENT_PID=$!
ps -o pid,vsz,rss,%mem,cmd -p "$FRAGMENT_PID"
wait "$FRAGMENT_PID"
```

**Expected observation:** Some Python objects are released, but the process's RSS may not immediately fall by the same amount. Python's allocator and the operating system may retain pages for reuse.



### Task 13 - Monitor Disk Activity and Swap Indicators

Install `iostat` if necessary:

```bash
sudo apt install sysstat -y
```

In **Terminal 1**, monitor disk activity:

```bash
iostat -dx 2 5
```

In **Terminal 2**, monitor virtual-memory activity:

```bash
vmstat 2 5
```

Observe `si` (swap-in) and `so` (swap-out) in `vmstat`, together with disk metrics in `iostat`. **`iostat` alone does not measure swap activity.**

The source PDF provides this larger optional example, `memory3.py`:

```python
import time

arr = bytearray(500 * 1024 * 1024)  # Approximately 500 MiB
time.sleep(30)
```

**Do not run it unless your VM has sufficient available memory and your instructor permits the workload.** You may use a smaller allocation, such as `64 * 1024 * 1024`, and state the amount used. Use one memory-intensive program at a time.

**Checkpoint:** Did you actually observe swap-in or swap-out? If not, report **no swap activity observed**, rather than claiming that page replacement occurred.

### Task 14 - Test a Temporary Virtual-Memory Limit

Create `memory_limit_fail.py`:

```bash
nano memory_limit_fail.py
```

Enter:

```python
arr = bytearray(150 * 1024 * 1024)  # 150 MiB
print("Allocation completed")
```

In a **subshell** so your normal terminal's limit is not changed, run:

```bash
(ulimit -v 100000; python3 memory_limit_fail.py)
```

`ulimit -v 100000` limits the subprocess's virtual address space to approximately **100,000 KiB**. The 150 MiB allocation should fail, although the exact error may depend on Python and the VM.

> [!alert] Do not apply a restrictive `ulimit` permanently or to your main login shell. `python3` may fail before reaching the allocation if its runtime needs more virtual address space than the limit allows.

**Checkpoint:** Record the actual error or exit behaviour and explain why a 150 MiB request cannot fit inside a roughly 100 MiB virtual-address-space limit.


---


## Troubleshooting

### `ModuleNotFoundError: No module named 'psutil'`

Try:

```bash
sudo apt install python3-psutil -y
python3 -c 'import psutil; print(psutil.__version__)'
```

If your instructor requires a virtual environment, install the package inside that environment instead.

### `vmstat`, `iostat`, or `htop`: command not found

On an Ubuntu/Debian-based VM:

```bash
sudo apt install procps sysstat htop -y
```

### `/proc/<PID>/maps`: No such file or directory

The program probably finished before you inspected it, or the PID is wrong. Rerun the program, note `$!`, and inspect it before its sleep period ends.

### The memory-intensive program is slow or the VM becomes unstable

Stop the experiment, reduce the array/list size, and repeat **both comparison programs with the same reduced input**. Record your change. Do not deliberately exhaust a shared lab VM.

### No swapping appears in `vmstat`

That is a valid outcome. Record that the tested workload did not generate observable swap activity; do not describe disk I/O as swap without corroborating evidence.

### `memory1.c`, `memory2.c`, or assessment scripts are unavailable

The separate `scripts.zip` mentioned in the original PDF is required for the assessed programs. Obtain the instructor's package rather than treating the practice examples as identical substitutes.

---

## Final Validation

Before finishing, check that you have:

- [ ] Recorded `free -h`, `/proc/meminfo`, and `vmstat -s` results.
- [ ] Run `mem_info.py` and explained `1024 ** 3`.
- [ ] Measured VSZ and RSS for both practice programs.
- [ ] Inspected a running process's `/proc/<PID>/maps`.
- [ ] Created and inspected a file-backed `mmap` region.
- [ ] Run the allocation/reuse demonstration and described the observation accurately.
- [ ] Distinguished disk I/O metrics from swap-in/swap-out counters.
- [ ] Tested a temporary memory limit safely.
- [ ] Monitored memory with Linux tools and `psutil`.



## Knowledge Check

1. What is the difference between `MemFree` and `MemAvailable`?
2. What do VSZ and RSS measure, and why can they differ?
3. Why might releasing Python objects not immediately lower a process's RSS?
4. What information appears in `/proc/<PID>/maps`?
5. What does `mmap` allow a program to do?
6. Which `vmstat` columns provide evidence of swap-in and swap-out?


