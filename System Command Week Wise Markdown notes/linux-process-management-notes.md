# Linux Process Management — Complete Notes
### A Full Reference Guide with Examples (Beginner → Advanced)

---

## 📑 Table of Contents

1. [What is a Process?](#1-what-is-a-process)
2. [How Processes Are Created — fork() and exec()](#2-how-processes-are-created--fork-and-exec)
3. [PID, PPID, and the Process Tree](#3-pid-ppid-and-the-process-tree)
4. [Process States (Lifecycle)](#4-process-states-lifecycle)
5. [Viewing Processes — ps](#5-viewing-processes--ps)
6. [Live Monitoring — top and htop](#6-live-monitoring--top-and-htop)
7. [Foreground vs Background Processes](#7-foreground-vs-background-processes)
8. [Job Control — jobs, fg, bg, disown](#8-job-control--jobs-fg-bg-disown)
9. [Signals — Communicating with Processes](#9-signals--communicating-with-processes)
10. [Killing Processes — kill, killall, pkill](#10-killing-processes--kill-killall-pkill)
11. [Process Priority — nice and renice](#11-process-priority--nice-and-renice)
12. [Zombie and Orphan Processes](#12-zombie-and-orphan-processes)
13. [Daemons and Background Services](#13-daemons-and-background-services)
14. [systemd & systemctl — Managing Services](#14-systemd--systemctl--managing-services)
15. [Scheduling Future/Recurring Processes — cron & at](#15-scheduling-futurerecurring-processes--cron--at)
16. [The /proc Filesystem — Processes as Files](#16-the-proc-filesystem--processes-as-files)
17. [Resource Monitoring Tools](#17-resource-monitoring-tools)
18. [Inter-Process Communication (IPC) — Quick Overview](#18-inter-process-communication-ipc--quick-overview)
19. [Common Errors & Troubleshooting](#19-common-errors--troubleshooting)
20. [Master Command Cheat Sheet](#20-master-command-cheat-sheet)
21. [Exam-Style Q&A](#21-exam-style-qa)
22. [Revision Summary](#22-revision-summary)

---

## 1. What is a Process?

A **process** is a program **in execution** — a running instance of a program, loaded into memory, with its own allocated resources (CPU time, memory, open file descriptors, etc.).

- A **program** is a passive file sitting on disk (e.g., `/usr/bin/firefox`).
- A **process** is that program actively running, with its own state, memory space, and unique identifier.
- The **same program** can be run multiple times, creating **multiple independent processes** (e.g., opening 3 terminal windows = 3 separate `bash` processes).

**Every process has, at minimum:**

| Attribute | Meaning |
|---|---|
| PID | Process ID — unique number identifying the process |
| PPID | Parent Process ID — the PID of the process that created it |
| UID/GID | User/Group that owns the process (determines permissions) |
| State | Current status (running, sleeping, stopped, zombie, etc.) |
| Priority / Nice value | How much CPU time it's favored to get |
| Memory space | Its own virtual address space (code, stack, heap) |
| Open file descriptors | Files, sockets, pipes it currently has open |

**Example — proving two runs of the same program are different processes:**
```bash
$ sleep 100 &
[1] 20481
$ sleep 100 &
[2] 20502
$ ps aux | grep sleep
john     20481  0.0  0.0   2384   604 pts/0    S    10:01   0:00 sleep 100
john     20502  0.0  0.0   2384   604 pts/0    S    10:01   0:00 sleep 100
```
Same program (`sleep`), two completely separate PIDs (`20481` and `20502`) — two distinct processes.

**📚 Learn more:** [The Linux Programming Interface — Ch.6 Processes](https://man7.org/tlpi/) · `man 7 credentials`

---

## 2. How Processes Are Created — fork() and exec()

On Linux, **every process (except the very first one) is created by another process** — there is no other way to create a process.

### The fork() + exec() model

1. **`fork()`** — an existing process duplicates itself, creating a near-identical copy (the **child**). Both parent and child now run independently, from the same point in the code, with separate memory spaces (initially copy-on-write).
2. **`exec()`** — the child process **replaces its own memory image** with a new program. This is how a duplicated shell becomes, say, `ls` or `vim`.

```
Parent process (bash)
      │
      │ fork()  →  creates identical child process
      ▼
Child process (a copy of bash)
      │
      │ exec("/bin/ls")  →  child's memory is replaced with the ls program
      ▼
Child is now running "ls", parent (bash) waits for it to finish
```

**Real demonstration — watching this happen:**
```bash
$ bash -c 'echo "My PID before exec: $$"; exec ls'
My PID before exec: 20601
Desktop  Documents  Downloads   # ls output — SAME PID (20601), because exec() replaces, doesn't create a new PID
```
Notice: `exec` did **not** create a new PID — it replaced the current process's program in place. This is different from just running `ls` normally, which would `fork()` first (new PID), then `exec()`.

> ⚠️ **Important distinction (common exam trap):** `fork()` creates a **new PID**. `exec()` does **not** — it just replaces the code running under the *existing* PID.

**The very first process — PID 1:**
```bash
$ ps -p 1
  PID TTY          TIME CMD
    1 ?        00:00:02 systemd
```
PID 1 (`systemd`, or `init` on older systems) is started directly by the kernel at boot and is the ancestor of **every other process** on the system.

**📚 Learn more:** [man7.org — fork(2)](https://man7.org/linux/man-pages/man2/fork.2.html) · [man7.org — execve(2)](https://man7.org/linux/man-pages/man2/execve.2.html)

---

## 3. PID, PPID, and the Process Tree

Because every process is created by `fork()`ing another, the entire set of running processes forms a **tree**, rooted at PID 1.

| Term | Meaning |
|---|---|
| **PID** | Process ID — unique to this process |
| **PPID** | Parent Process ID — PID of whoever created it |
| **PGID** | Process Group ID — used to group related processes (e.g., a pipeline) |
| **SID** | Session ID — groups process groups under one controlling terminal/login session |

### Viewing the process tree

```bash
pstree                 # visual tree of all processes
pstree -p               # include PIDs
pstree -p $$            # tree starting from your current shell
ps -ef --forest         # ps output shown as an indented tree
```

**Example output (`pstree -p`):**
```
systemd(1)─┬─NetworkManager(823)
           ├─sshd(945)───sshd(20301)───bash(20302)───pstree(20455)
           └─cron(760)
```
Reading this: `bash (20302)` was spawned by `sshd (20301)` (your SSH login session), which was spawned by the main `sshd (945)` daemon, which was spawned by `systemd (1)`.

**Finding a process's parent directly:**
```bash
$ ps -o pid,ppid,cmd -p 20302
  PID  PPID CMD
20302   20301 bash
```

> ⚠️ **Edge case:** If a parent process dies before its child, the child is **re-parented to PID 1** (`systemd`/`init`) automatically — it is never left "parentless." See [Section 12](#12-zombie-and-orphan-processes) for details.

**📚 Learn more:** `man pstree` · `man 5 proc`

---

## 4. Process States (Lifecycle)

A process moves through several possible **states** during its life:

| State | Code (in `ps`/`top`) | Meaning |
|---|:---:|---|
| **Running** | `R` | Actively executing on the CPU, or ready/queued to run |
| **Sleeping (interruptible)** | `S` | Waiting for an event (input, timer, I/O) — can be woken by a signal |
| **Sleeping (uninterruptible)** | `D` | Waiting on I/O (usually disk) — **cannot** be interrupted or killed until I/O completes |
| **Stopped** | `T` | Execution suspended (e.g., via `Ctrl+Z` or `SIGSTOP`) — can be resumed |
| **Zombie** | `Z` | Process has finished executing but its exit status hasn't been collected by its parent yet |

```
        fork()
          │
          ▼
      ┌───────┐   scheduled    ┌─────────┐
      │ Ready │ ─────────────► │ Running │
      └───────┘ ◄───────────── └─────────┘
          ▲        preempted        │
          │                          │ waits for I/O / event
          │                          ▼
          │                    ┌──────────┐
          └──── event occurs ──│ Sleeping │
                                └──────────┘
                                     │
                     Ctrl+Z/SIGSTOP  │  process exits
                                     ▼
                          ┌────────┐   parent calls wait()   ┌──────────┐
                          │ Stopped│ ───────────────────────►│ Zombie   │→ removed
                          └────────┘                          └──────────┘
```

**Seeing states live:**
```bash
$ ps aux | awk '{print $8}' | sort | uniq -c
      4 S
      1 R
      1 Z
```
```bash
$ ps aux | grep " D "     # find processes stuck in uninterruptible sleep (often disk trouble)
```

> ⚠️ **Important Point to Remember:** A process in state `D` **cannot be killed even with `kill -9`** — it must wait for the I/O operation to complete or timeout. If a system has many `D`-state processes piling up, that's a strong sign of a failing disk or an NFS/network-storage hang.

**📚 Learn more:** `man ps` (see the "PROCESS STATE CODES" section) · [Kernel docs — Process states](https://www.kernel.org/doc/html/latest/filesystems/proc.html)

---

## 5. Viewing Processes — `ps`

**Purpose:** Snapshot ("process status") of currently running processes at the moment the command is run (not live-updating).

**Basic syntax:**
```bash
ps [options]
```

### The two "flavors" of ps syntax (a very common source of confusion)

| Style | Example | Notes |
|---|---|---|
| BSD-style (no dash) | `ps aux` | Traditional, most commonly memorized |
| UNIX/POSIX-style (with dash) | `ps -ef` | More portable across UNIX systems |

Both show *similar* information but with **different column names/formats** — don't mix flags between the two styles carelessly (e.g. `ps -aux` with a dash is technically wrong syntax, though many systems tolerate it with a warning).

**Examples:**
```bash
ps aux                              # every process, every user, BSD style
ps -ef                              # every process, POSIX style, shows PPID clearly
ps aux | grep firefox               # find a specific process by name
ps -u john                          # only processes owned by user "john"
ps -p 20481                         # info about one specific PID
ps -o pid,ppid,%cpu,%mem,cmd -p 20481   # custom columns for one PID
ps -ef --forest                     # tree view showing parent/child indentation
ps aux --sort=-%cpu | head -6       # top 5 CPU-consuming processes
ps aux --sort=-%mem | head -6       # top 5 memory-consuming processes
ps -eLf                             # show threads too (L = LWP, lightweight process)
```

**Sample output (`ps aux`):**
```
USER   PID  %CPU %MEM    VSZ   RSS TTY   STAT START   TIME COMMAND
john  1022   0.3  1.2 412300 51200 pts/0 Sl   09:12   0:03 /usr/bin/gnome-terminal
john  20481  0.0  0.0   2384   604 pts/0 S    10:01   0:00 sleep 100
root   945   0.0  0.1  15200  3200 ?     Ss   08:00   0:00 /usr/sbin/sshd -D
```

**Column meanings:**

| Column | Meaning |
|---|---|
| `USER` | Owner of the process |
| `PID` | Process ID |
| `%CPU` | CPU usage percentage since process start |
| `%MEM` | Percentage of physical RAM used (RSS ÷ total RAM) |
| `VSZ` | Virtual memory size (KB) — total address space, including unused/shared |
| `RSS` | Resident Set Size (KB) — actual physical RAM currently in use |
| `TTY` | Controlling terminal (`?` = no terminal, i.e. a daemon) |
| `STAT` | Process state code (see Section 4), often with modifiers (`s`=session leader, `l`=multi-threaded, `+`=foreground) |
| `START` | Time/date the process started |
| `TIME` | Total CPU time actually consumed (not wall-clock time) |
| `COMMAND` | The command line that launched it |

> ⚠️ **Important Point to Remember:** `ps` is a **snapshot**, not live — it reflects the instant you ran it. For a live, refreshing view, use `top`/`htop` (Section 6). Also, a **high `TIME` value ≠ the process has been running long** — `TIME` is *CPU seconds consumed*, while `START` shows *wall-clock* start time; a process can run for hours with only a few seconds of `TIME` if it's mostly sleeping.

**📚 Learn more:** `man ps` · [GNU/Linux procps-ng project](https://gitlab.com/procps-ng/procps)

---

## 6. Live Monitoring — `top` and `htop`

### 6.1 `top`

**Purpose:** Real-time, auto-refreshing view of system resource usage and running processes, sorted (by default) by CPU usage.

```bash
top                # launch interactive live view
top -u john         # only show john's processes
top -p 20481,20502  # monitor specific PIDs only
top -n 1 -b          > snapshot.txt   # one-shot, non-interactive (good for logging/scripts)
top -d 5             # refresh every 5 seconds instead of default (usually 3s)
```

**Sample header output:**
```
top - 10:15:22 up  2:03,  1 user,  load average: 0.52, 0.61, 0.58
Tasks: 210 total,   1 running, 208 sleeping,   0 stopped,   1 zombie
%Cpu(s):  8.3 us,  2.1 sy,  0.0 ni, 88.9 id,  0.5 wa,  0.0 hi,  0.2 si,  0.0 st
MiB Mem :  15872.0 total,   6120.4 free,   4302.1 used,   5449.5 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.  10890.2 avail Mem
```

**Key fields explained:**

| Field | Meaning |
|---|---|
| `load average` | Avg. number of processes wanting CPU time, over 1/5/15 minutes — compare against CPU core count |
| `us` | % CPU time in user-space programs |
| `sy` | % CPU time in kernel (system) calls |
| `wa` | % CPU time waiting on I/O — **high `wa` = disk/network bottleneck**, not CPU |
| `id` | % CPU idle |

**Interactive keys inside `top` (very testable):**

| Key | Effect |
|---|---|
| `k` | Kill a process (prompts for PID, then signal) |
| `r` | Renice a process (change priority) |
| `M` | Sort by memory usage |
| `P` | Sort by CPU usage (default) |
| `1` | Toggle per-core CPU breakdown |
| `u` | Filter by username |
| `q` | Quit |

> ⚠️ **Important Point to Remember:** `load average` is **not a percentage**. A load average of `4.0` on a **4-core** machine means the CPUs are fully (100%) utilized on average; the same `4.0` on a **2-core** machine means the system is **overloaded** (twice the demand it can serve at once). Always compare load average against `nproc`.

```bash
nproc          # number of logical CPUs, for comparison against load average
```

### 6.2 `htop` (enhanced, more user-friendly alternative)

```bash
sudo apt install htop      # not installed by default on many systems
htop
```

**Advantages over `top`:**
- Color-coded, scrollable, mouse-clickable interface.
- Shows **per-core** CPU bars at the top by default (no toggle needed).
- Tree view (`F5`) shows parent/child relationships directly, like `pstree`.
- Can kill/renice/search processes with function-key shortcuts (`F9` = kill, `F7`/`F8` = nice/un-nice) without typing a PID.

> ✅ **Remember for exams:** `top` is available on virtually **every** Linux system by default (part of `procps`); `htop` must usually be **installed separately** — don't assume it's present on a minimal/server install.

**📚 Learn more:** `man top` · [htop official site](https://htop.dev/)

---

## 7. Foreground vs Background Processes

- **Foreground process:** occupies the terminal — you can't type new commands until it finishes.
- **Background process:** runs independently of terminal input; you get your shell prompt back immediately.

**Running something in the background** — append `&`:
```bash
$ sleep 300 &
[1] 20601
```
- `[1]` = job number (used with `fg`/`bg`, see Section 8).
- `20601` = the actual PID.

**Sending a currently-running foreground job to the background:**
```bash
$ sleep 300
^Z                          # Ctrl+Z suspends it (state becomes T = stopped)
[1]+  Stopped                 sleep 300
$ bg                        # resume it, but in the background
[1]+ sleep 300 &
```

**Bringing a background job back to the foreground:**
```bash
$ fg %1
sleep 300                   # now occupying the terminal again
```

**Running a command immune to terminal hangup (survives you logging out):**
```bash
nohup long_script.sh &
# Output redirected automatically to nohup.out unless you specify otherwise:
nohup long_script.sh > output.log 2>&1 &
```

> ⚠️ **Important Point to Remember:** `&` alone does **not** protect a process from being killed when you close the terminal — closing the terminal normally sends `SIGHUP` (hangup signal) to all jobs in that session. Use `nohup`, `disown`, or a terminal multiplexer (`tmux`/`screen`) to survive logout. See Section 8 for `disown`.

**📚 Learn more:** `man nohup` · [GNU Bash Manual — Job Control](https://www.gnu.org/software/bash/manual/html_node/Job-Control.html)

---

## 8. Job Control — `jobs`, `fg`, `bg`, `disown`

**Job control** lets one shell session manage multiple background/suspended commands.

```bash
jobs             # list jobs in the current shell, with job numbers and states
jobs -l           # also show PIDs
```

**Full worked example:**
```bash
$ sleep 500 &
[1] 21001
$ sleep 600 &
[2] 21050
$ vim notes.txt
^Z
[3]+  Stopped                 vim notes.txt
$ jobs
[1]   Running                 sleep 500 &
[2]-  Running                 sleep 600 &
[3]+  Stopped                 vim notes.txt
```

| Symbol next to job | Meaning |
|---|---|
| `+` | The "current" job — the default target of `fg`/`bg` with no argument |
| `-` | The "previous" job (second most recent) |
| *(none)* | Older background jobs |

**Controlling jobs by number:**
```bash
fg %1              # bring job 1 to foreground
bg %2               # resume job 2 in background (if stopped)
kill %3             # send SIGTERM to job 3 (equivalent to kill <its PID>)
```

**Detaching a job from the shell entirely (`disown`):**
```bash
$ long_backup.sh &
[1] 21200
$ disown %1
```
After `disown`, job `1` no longer appears in `jobs`, and closing the terminal will **not** send it `SIGHUP` — it becomes fully independent, similar to what `nohup` achieves but applied *after* the process has already started.

> ⚠️ **Edge case:** `disown` ≠ `nohup`. `nohup` must be applied **when starting** the command (so it can redirect signal handling from the start); `disown` is applied **afterward** to a job already running in your current shell. They solve the same underlying problem from two different angles.

**📚 Learn more:** [GNU Bash Manual — Job Control Builtins](https://www.gnu.org/software/bash/manual/html_node/Job-Control-Builtins.html)

---

## 9. Signals — Communicating with Processes

A **signal** is a limited, asynchronous message sent to a process, telling it to do something — terminate, pause, reload configuration, etc. Processes can choose to **handle**, **ignore**, or (for a few signals) are **forced** to obey.

### The most important signals

| Signal | Number | Default action | Can be caught/ignored? | Typical use |
|---|:---:|---|:---:|---|
| `SIGHUP` | 1 | Terminate (historically: "terminal hung up") | ✅ Yes | Often repurposed to mean "reload config" (e.g., `nginx`, `sshd`) |
| `SIGINT` | 2 | Terminate | ✅ Yes | Sent by `Ctrl+C` — "interrupt" |
| `SIGQUIT` | 3 | Terminate + core dump | ✅ Yes | Sent by `Ctrl+\` |
| `SIGKILL` | 9 | Terminate | ❌ **No — cannot be caught, blocked, or ignored** | Force-kill an unresponsive process |
| `SIGTERM` | 15 | Terminate | ✅ Yes | The **polite/default** kill signal — asks the process to clean up first |
| `SIGSTOP` | 19 | Stop (pause) | ❌ **No — cannot be caught or ignored** | Force-pause a process |
| `SIGCONT` | 18 | Continue if stopped | ✅ Yes | Resume a paused process |
| `SIGTSTP` | 20 | Stop (pause) | ✅ Yes | Sent by `Ctrl+Z` — "terminal stop" (can be caught, unlike SIGSTOP) |
| `SIGCHLD` | 17 | Ignored by default | ✅ Yes | Sent to a parent when a child terminates |
| `SIGUSR1` / `SIGUSR2` | 10 / 12 | Terminate (default) | ✅ Yes | Free for **custom, application-defined** use |

**Listing all signals:**
```bash
kill -l                 # list every signal name and number
```

**Sending a signal (three equivalent ways to request graceful termination):**
```bash
kill 20481              # sends SIGTERM (15) by default
kill -15 20481           # explicit, same as above
kill -SIGTERM 20481      # same, using the name
```

**Force-killing an unresponsive process:**
```bash
kill -9 20481            # SIGKILL — the kernel kills it immediately, no cleanup
kill -SIGKILL 20481
```

**Pausing and resuming with signals directly:**
```bash
kill -STOP 20481          # equivalent to Ctrl+Z, but from another terminal
kill -CONT 20481          # resume it
```

**A program reacting to a custom signal (Bash trap example):**
```bash
#!/bin/bash
trap 'echo "Caught SIGUSR1! Reloading config..."; exit 0' SIGUSR1
echo "PID $$ waiting for signal..."
while true; do sleep 1; done
```
Run it, then in another terminal:
```bash
$ kill -SIGUSR1 <pid_of_script>
```
Output in the first terminal:
```
Caught SIGUSR1! Reloading config...
```

> ⚠️ **Important Point to Remember (very commonly tested):** `SIGKILL` (9) and `SIGSTOP` (19) are the **only two signals a process can never intercept, block, or ignore** — this is enforced by the kernel itself, by design, so there is always a guaranteed way to stop any process no matter how badly it misbehaves. Always try `SIGTERM` (15) first to allow graceful shutdown (closing files, saving state); reach for `SIGKILL` only when a process is truly unresponsive.

**📚 Learn more:** `man 7 signal` · [man7.org — signal-safety(7)](https://man7.org/linux/man-pages/man7/signal-safety.7.html)

---

## 10. Killing Processes — `kill`, `killall`, `pkill`

| Command | Targets by | Example |
|---|---|---|
| `kill` | PID (exact) | `kill 20481` |
| `killall` | Exact process **name** | `killall firefox` |
| `pkill` | **Pattern match** (name, user, etc.) | `pkill -u john` |

**Examples:**
```bash
kill 20481                       # graceful SIGTERM to one PID
kill -9 20481 20502               # force-kill multiple PIDs at once
killall firefox                   # kill every process literally named "firefox"
killall -9 firefox                 # force-kill all of them
pkill -f "python3 server.py"       # match against the full command line, not just the process name
pkill -u guestuser                  # kill all processes owned by a specific user
pkill -9 -x sshd                    # -x = exact name match only (avoid matching "sshd-something")
```

**Finding a PID before killing it (common combo):**
```bash
pgrep firefox                     # list matching PIDs (companion tool to pkill)
pgrep -l firefox                   # also show the process name next to each PID
ps aux | grep '[f]irefox'          # classic alternative; the [f] trick avoids matching the grep command itself
```

> ⚠️ **Important Point to Remember:** Plain `ps aux | grep firefox` will **also match its own grep process** in the results (since the command line literally contains "firefox"). The `[f]irefox` bracket trick avoids this because the regex `[f]irefox` no longer matches the literal string `grep [f]irefox`. `pgrep`/`pkill` avoid this problem entirely since they don't spawn a matching grep process.

> ⚠️ **Safety edge case:** `killall` on Linux matches by **process name**, but on **Solaris/some UNIX variants**, an old-school `killall` (no arguments) kills **all processes you have permission to kill** — a dangerous difference if you're used to a different UNIX flavor. Always double check with `man killall` on unfamiliar systems.

**📚 Learn more:** `man kill` · `man killall` · `man pkill` · `man pgrep`

---

## 11. Process Priority — `nice` and `renice`

Linux schedules CPU time based partly on a process's **niceness** value, ranging from **-20 (highest priority) to +19 (lowest priority)**. Default niceness for a new process is `0`.

> 💡 Mnemonic: a "nice" process is *nice to others* — it yields CPU time, i.e., a **higher nice value = lower priority**.

**Starting a process with a custom priority:**
```bash
nice -n 10 ./backup_script.sh &      # start at LOWER priority (nicer to other processes)
nice -n -5 ./critical_task.sh &       # start at HIGHER priority (requires... see below)
```

**Changing the priority of an already-running process:**
```bash
renice -n 15 -p 20481                # lower the priority of PID 20481
renice -n 10 -u john                  # apply to all of user john's processes
```

**Viewing current niceness:**
```bash
ps -o pid,ni,cmd -p 20481
top      # the "NI" column shows niceness live; "PR" shows the resulting kernel priority
```

> ⚠️ **Important Point to Remember:** Only **root** can set a **negative** nice value (increase priority above default) or lower the niceness of a process further once it's already negative. A normal user can only make their own processes **less** favored (raise the value toward +19), never more favored than default — this prevents regular users from starving the system of CPU for other users' processes.

```bash
$ nice -n -10 ./task.sh
nice: cannot set niceness: Permission denied     # normal user, negative value → fails
$ sudo nice -n -10 ./task.sh                       # works with root privileges
```

**📚 Learn more:** `man nice` · `man renice` · [Kernel docs — CFS scheduler](https://www.kernel.org/doc/html/latest/scheduler/sched-design-CFS.html)

---

## 12. Zombie and Orphan Processes

### Zombie processes (state `Z`)

A **zombie** is a process that has **already finished executing**, but its entry still exists in the process table because its **parent hasn't yet called `wait()`** to read its exit status. It consumes essentially **no resources except a process table slot** — no CPU, no memory beyond bookkeeping.

**Simulating a zombie for demonstration:**
```bash
#!/bin/bash
# zombie_demo.sh
(sleep 1; exit 0) &     # child finishes quickly...
sleep 20                # ...but parent (this script) doesn't wait() for a while
```
```bash
$ ./zombie_demo.sh &
$ ps aux | grep Z
john  21402  0.0  0.0     0     0 pts/0    Z    10:20   0:00 [sleep] <defunct>
```
Notice `<defunct>` — the standard label for a zombie in `ps` output.

**Why you (almost) can't `kill` a zombie:** it's already dead — there's no running program left to signal. The fix is to make the **parent** reap it (call `wait()`), or if the parent itself is broken/won't exit, kill the **parent**, which causes the zombie to be re-parented to `systemd`/`init`, which automatically reaps orphaned zombies.

```bash
kill -9 20481      # killing the zombie itself typically does nothing — it's already dead
kill -9 <PPID>      # killing the misbehaving PARENT allows init to reap the zombie
```

> ⚠️ **Important Point to Remember:** A **small number** of transient zombies is completely normal and harmless (they disappear as soon as the parent gets around to `wait()`ing). A **large, growing** number of permanent zombies indicates a **buggy parent program** that never reaps its children — this is a real production bug to report/fix, not something to "clean up" manually process by process.

### Orphan processes

An **orphan** is a process whose **parent has terminated** while the child is still running. Linux immediately **re-parents** it to PID 1 (`systemd`/`init`), which takes over responsibility for eventually reaping it when it finishes.

```bash
$ (sleep 100 &) ; ps -o pid,ppid,cmd -p $!
    PID  PPID CMD
  21500     1 sleep 100
```
The subshell that launched `sleep 100` exits immediately after, orphaning it — and its `PPID` is now `1`.

| | Zombie | Orphan |
|---|---|---|
| Process itself | Already **dead**, just awaiting cleanup | Still **alive and running** |
| Cause | Parent hasn't called `wait()` yet | Parent has already exited |
| Resolved by | Parent eventually reaping it, or init reaping it if parent dies | Automatically re-parented to `init`/`systemd`, which reaps it normally when it finishes |
| Danger level | Harmless individually; a growing pile = bug | Generally harmless — this is standard, expected Unix behavior |

**📚 Learn more:** `man 2 wait` · [man7.org — wait(2)](https://man7.org/linux/man-pages/man2/wait.2.html)

---

## 13. Daemons and Background Services

A **daemon** is a background process, typically started at boot, with **no controlling terminal** (shown as `?` in the `TTY` column of `ps`), that runs continuously to provide a service (e.g., `sshd` for remote login, `cron` for scheduled tasks, `cupsd` for printing).

**Naming convention:** daemon names traditionally end in `d` — `sshd`, `crond`, `httpd`, `named`.

**Spotting daemons:**
```bash
$ ps -eo pid,tty,cmd | grep '?'
  945 ?        /usr/sbin/sshd -D
  760 ?        /usr/sbin/cron -f
 1102 ?        /usr/sbin/cupsd -l
```

**Traditional way a process detaches itself to become a daemon (conceptually):**
1. `fork()` and let the parent exit immediately (child is orphaned → reparented to init).
2. Call `setsid()` to become a session leader, detaching from any controlling terminal.
3. Redirect `stdin`/`stdout`/`stderr` away from the terminal (often to `/dev/null` or a log file).
4. Optionally `fork()` a second time to guarantee it can never re-acquire a controlling terminal.

> ✅ **Remember:** In modern Linux, most services are no longer written to manually daemonize like this — **`systemd`** manages the daemonizing/supervision for you (see Section 14), which is far more robust (auto-restart on crash, logging via `journald`, dependency ordering).

**📚 Learn more:** [man7.org — daemon(7)](https://man7.org/linux/man-pages/man7/daemon.7.html)

---

## 14. systemd & systemctl — Managing Services

**systemd** is the modern init system (PID 1) on most major distributions (Ubuntu, Debian, Fedora, RHEL, Arch). It manages services as **units**, most commonly `.service` files.

**Core `systemctl` commands:**
```bash
sudo systemctl start nginx           # start a service now
sudo systemctl stop nginx             # stop it now
sudo systemctl restart nginx          # stop then start
sudo systemctl reload nginx           # reload config WITHOUT dropping connections (if supported)
sudo systemctl status nginx           # current status + recent log lines
sudo systemctl enable nginx           # start automatically on every future boot
sudo systemctl disable nginx          # remove from auto-start at boot
sudo systemctl enable --now nginx     # enable AND start in one command
systemctl is-active nginx              # quick yes/no check: "active" / "inactive"
systemctl is-enabled nginx             # quick yes/no check for boot-start
systemctl list-units --type=service --state=running   # list all currently running services
```

**Sample `systemctl status` output:**
```
● nginx.service - A high performance web server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2026-09-12 09:00:11 UTC; 3h ago
   Main PID: 1188 (nginx)
      Tasks: 3 (limit: 4915)
     Memory: 5.2M
        CPU: 145ms
     CGroup: /system.slice/nginx.service
             ├─1188 nginx: master process /usr/sbin/nginx
             └─1189 nginx: worker process
```

**Viewing logs for a service (via journald):**
```bash
journalctl -u nginx               # all logs for the nginx unit
journalctl -u nginx -f             # follow live, like tail -f
journalctl -u nginx --since today  # only today's entries
journalctl -u nginx -p err          # only error-priority and above
```

> ⚠️ **Important Point to Remember:** `enable` only affects whether a service **starts at boot** — it does **not** start it right now. `start` only affects **right now** — it does **not** persist across a reboot. A very common mistake is running only one of the two and being surprised the service isn't running (or isn't running after a reboot). Use `enable --now` to do both at once.

**📚 Learn more:** `man systemctl` · `man systemd.service` · [freedesktop.org — systemd documentation](https://www.freedesktop.org/wiki/Software/systemd/)

---

## 15. Scheduling Future/Recurring Processes — `cron` & `at`

### `cron` — recurring scheduled tasks

Each user can have a **crontab** (cron table) listing commands to run on a recurring schedule.

```bash
crontab -e            # edit your own crontab (opens in default editor)
crontab -l             # list your current crontab entries
crontab -r             # remove your entire crontab (careful — no confirmation!)
sudo crontab -u john -e   # edit another user's crontab (root only)
```

**Crontab syntax:**
```
* * * * * command-to-run
│ │ │ │ │
│ │ │ │ └── day of week (0–6, Sunday=0)
│ │ │ └──── month (1–12)
│ │ └────── day of month (1–31)
│ └──────── hour (0–23)
└────────── minute (0–59)
```

**Worked examples:**
```bash
0 2 * * *      /home/john/backup.sh          # every day at 2:00 AM
*/15 * * * *   /home/john/check_disk.sh       # every 15 minutes
0 9 * * 1-5    /home/john/standup_reminder.sh # 9:00 AM, Monday through Friday
0 0 1 * *      /home/john/monthly_report.sh   # midnight on the 1st of every month
@reboot        /home/john/startup_task.sh     # once, at every system boot
```

**System-wide cron directories (no crontab editing needed):**
```bash
/etc/cron.hourly/     # scripts run once an hour
/etc/cron.daily/      # scripts run once a day
/etc/cron.weekly/     # scripts run once a week
/etc/cron.monthly/    # scripts run once a month
```
Just drop an executable script into the matching folder — `run-parts` (invoked by cron itself) executes everything inside on schedule.

> ⚠️ **Important Point to Remember:** Cron jobs run with a **minimal environment** — no interactive `$PATH`, no profile files sourced. A script that works fine when you run it manually can **silently fail under cron** because it can't find a command it assumed was in `$PATH`. Always use **full absolute paths** to commands/scripts inside crontab entries, or explicitly set `PATH=` at the top of the crontab.

### `at` — run a command exactly once, in the future

```bash
sudo apt install at            # often not installed by default
echo "/home/john/cleanup.sh" | at 23:00          # run once tonight at 11 PM
at now + 30 minutes <<< "/home/john/reminder.sh"  # run once, 30 minutes from now
atq                             # list pending 'at' jobs
atrm 3                          # cancel pending job number 3
```

| | `cron` | `at` |
|---|---|---|
| Purpose | **Recurring** scheduled tasks | **One-time**, future scheduled task |
| Typical use | Nightly backups, log rotation | "Remind me / run this once at 5 PM today" |

**📚 Learn more:** `man crontab` · `man 5 crontab` (format reference) · `man at`

---

## 16. The `/proc` Filesystem — Processes as Files

`/proc` is a **virtual filesystem** — it doesn't exist on disk; the kernel generates its contents on the fly. Every running process gets its own directory: `/proc/<PID>/`.

```bash
ls /proc/20481/
```
```
cmdline  cwd  environ  exe  fd  maps  status  limits  stat  statm  ...
```

**Key files inside a process's `/proc/<PID>/` directory:**

| File | Contents |
|---|---|
| `cmdline` | The exact command line used to start it |
| `status` | Human-readable summary — state, memory, UID/GID, threads |
| `environ` | The process's environment variables (null-separated) |
| `cwd` | Symlink to its current working directory |
| `exe` | Symlink to the actual binary being executed |
| `fd/` | Directory of symlinks — one per **open file descriptor** |
| `maps` | Memory-mapped regions (libraries, heap, stack) |
| `limits` | Its resource limits (ulimit-style: max open files, max processes, etc.) |

**Practical examples:**
```bash
cat /proc/20481/cmdline | tr '\0' ' '; echo    # readable command line (fields are NUL-separated)
cat /proc/20481/status | head -5                # quick state/memory summary
ls -l /proc/20481/fd                             # what files/sockets/pipes it has open
readlink /proc/20481/exe                         # exact binary path currently running
cat /proc/20481/environ | tr '\0' '\n'           # list its environment variables, one per line
```

**Sample `status` output:**
```
Name:   sleep
State:  S (sleeping)
Pid:    20481
PPid:   20302
Uid:    1000    1000    1000    1000
VmRSS:      604 kB
Threads:    1
```

**System-wide (not per-process) `/proc` files worth knowing:**
```bash
cat /proc/loadavg          # same numbers shown by `top`'s load average line
cat /proc/uptime            # system uptime in seconds
cat /proc/meminfo           # detailed memory stats (source data for `free`)
cat /proc/cpuinfo           # per-core CPU details (source data for `lscpu`)
```

> ⚠️ **Important Point to Remember:** If you `ls /proc/20481` and get `No such file or directory`, it simply means that PID **no longer exists** (it exited between your last check and now) — `/proc` entries appear and disappear live as processes start and stop; there is no caching or staleness to worry about.

**📚 Learn more:** [kernel.org — /proc filesystem documentation](https://www.kernel.org/doc/html/latest/filesystems/proc.html) · `man 5 proc`

---

## 17. Resource Monitoring Tools

| Tool | Best for |
|---|---|
| `ps` | One-shot process snapshot |
| `top` / `htop` | Live, continuously refreshing overview |
| `free -h` | RAM/swap summary |
| `vmstat 2 5` | Virtual memory / CPU / IO stats, sampled every 2s, 5 times |
| `uptime` | Quick load average + how long the system has been up |
| `iostat` | Per-disk I/O throughput (from `sysstat` package) |
| `lsof -p <PID>` | List every open file/socket for a specific process |
| `strace -p <PID>` | Trace every system call a running process makes (deep debugging) |
| `time <command>` | Measure how long a single command takes, and its CPU usage split |

**Examples:**
```bash
uptime
# 10:32:01 up 5:20,  2 users,  load average: 1.02, 0.98, 0.87

vmstat 2 5
# procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
#  r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
#  1  0      0 6120400 210040 5580200    0    0     2    18  120  240  8  2 89  1  0

lsof -p 20481                 # every file/socket 20481 currently has open
lsof -i :8080                  # WHICH process is using port 8080 — extremely useful!
lsof -u john                   # all open files belonging to user john

strace -p 20481                # live syscall trace of a running process (Ctrl+C to stop)
strace -c ls /                 # summarized count/time of each syscall type used by `ls`

time sleep 2
# real    0m2.002s   ← actual wall-clock elapsed time
# user    0m0.001s   ← CPU time spent in user-space code
# sys     0m0.001s   ← CPU time spent in kernel on this process's behalf
```

> ⚠️ **Important Point to Remember (`lsof -i`):** `lsof -i :PORT` is one of the fastest, most practical real-world debugging commands — "what process is hogging this port so I can't start my server?" is an extremely common scenario, and this answers it in one line, faster than fully parsing `netstat`/`ss` output.

**📚 Learn more:** `man lsof` · `man strace` · `man vmstat` · `man time`

---

## 18. Inter-Process Communication (IPC) — Quick Overview

Processes are isolated by default (separate memory spaces) — IPC mechanisms let them **exchange data** deliberately.

| Mechanism | Description | Example |
|---|---|---|
| **Pipe** (`\|`) | One-way stream connecting one process's stdout to another's stdin | `ps aux \| grep firefox` |
| **Named pipe (FIFO)** | A pipe with a filesystem path, usable by unrelated processes | `mkfifo mypipe; cat mypipe` |
| **Signals** | Simple async notifications (see Section 9) | `kill -SIGUSR1 <pid>` |
| **Sockets** | Bidirectional communication, local (Unix sockets) or over a network (TCP/IP) | Web servers, databases |
| **Shared memory** | Multiple processes read/write the same memory region directly (fastest, but needs synchronization) | Database engines, high-performance apps |

**Quick named-pipe demonstration:**
```bash
# Terminal 1:
mkfifo /tmp/mypipe
cat /tmp/mypipe

# Terminal 2:
echo "Hello via FIFO" > /tmp/mypipe
```
Terminal 1 immediately prints `Hello via FIFO` — two unrelated processes just communicated through a filesystem-visible pipe.

**📚 Learn more:** `man 7 pipe` · `man 1 mkfifo` · `man 7 unix`

---

## 19. Common Errors & Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `bash: fork: retry: Resource temporarily unavailable` | Hit the max-processes limit (`ulimit -u`) for the user, or system RAM/PID exhaustion | Check `ulimit -u`, `ps -ef \| wc -l`; kill runaway processes or raise the limit |
| Process won't die even with `kill -9` | It's in uninterruptible sleep (`D` state, usually stuck I/O) | Wait for the I/O to resolve, or investigate the underlying disk/NFS issue — `kill -9` genuinely cannot help here |
| `kill: (20481) - No such process` | PID already exited, or typo'd the number | Re-check with `ps -p 20481` or `pgrep <name>` first |
| Command runs fine manually but fails under `cron` | Cron uses a minimal `$PATH`/environment | Use full absolute paths, or set `PATH=` explicitly at the top of the crontab |
| `Permission denied` when using `renice`/`nice` to raise priority | Only root can set negative niceness | Use `sudo`, or accept default/lower priority as a normal user |
| Many `<defunct>` (zombie) processes accumulating | Parent process has a bug and never calls `wait()` | Report/fix the parent program; killing the parent lets `init` reap the zombies |
| Service `systemctl start` works but doesn't survive reboot | Forgot `enable` (only `start` was used) | `sudo systemctl enable --now <service>` |
| `top` shows very high `wa` (I/O wait) percentage | Disk or network storage bottleneck, not a CPU problem | Investigate with `iostat -dx` / check for `D`-state processes |

---

## 20. Master Command Cheat Sheet

```bash
# --- Viewing ---
ps aux                          # all processes, BSD style
ps -ef --forest                 # all processes, tree view
pstree -p                       # visual parent/child tree
top / htop                      # live monitoring
pgrep -l <name>                 # find PIDs by name

# --- Job control ---
command &                       # run in background
Ctrl+Z                          # suspend foreground job
jobs -l                         # list jobs + PIDs
fg %1 / bg %1                    # resume in foreground/background
nohup command &                  # survive terminal logout
disown %1                        # detach an already-running job

# --- Signals & killing ---
kill -l                         # list all signal names
kill <pid>                       # SIGTERM (graceful)
kill -9 <pid>                    # SIGKILL (force)
killall <name>                   # kill by exact process name
pkill -f "<pattern>"              # kill by command-line pattern match

# --- Priority ---
nice -n 10 command &              # start with lower priority
renice -n 5 -p <pid>               # change priority of running process

# --- Services (systemd) ---
sudo systemctl enable --now nginx  # enable at boot + start now
sudo systemctl status nginx        # check status
journalctl -u nginx -f              # follow live logs

# --- Scheduling ---
crontab -e                        # edit recurring schedule
echo "cmd" | at 22:00               # one-time future task

# --- Deep inspection ---
cat /proc/<pid>/status             # detailed process info
lsof -p <pid>                      # open files/sockets
lsof -i :8080                       # what's using a port
strace -p <pid>                     # live syscall trace
```

---

## 21. Exam-Style Q&A

**Q1. What is the difference between `fork()` and `exec()`?**
> `fork()` creates a new process (new PID) that is a duplicate of the calling process. `exec()` replaces the *current* process's program code in place, without creating a new PID.

**Q2. Which two signals can never be caught, blocked, or ignored by a process, and why does this matter?**
> `SIGKILL` (9) and `SIGSTOP` (19) — enforced directly by the kernel so there is always a guaranteed way to terminate or pause any process, no matter how it's written.

**Q3. What's the practical difference between a zombie and an orphan process?**
> A zombie is already dead, just waiting for its parent to collect its exit status; an orphan is still alive and running, but its original parent has already exited (so it gets re-parented to `init`/`systemd`).

**Q4. Why might `ps aux | grep firefox` show one extra, unexpected line?**
> The `grep firefox` command itself has "firefox" in its own command line, so it matches its own process in the output. Using `grep '[f]irefox'` or `pgrep firefox` avoids this.

**Q5. A process shows state `D` in `top`. Why won't `kill -9` stop it?**
> `D` = uninterruptible sleep, almost always waiting on I/O (often disk). The kernel won't deliver *any* signal, including `SIGKILL`, until the I/O operation completes — this is a hardware/driver-level wait, not something signals can interrupt.

**Q6. What's the difference between `systemctl start` and `systemctl enable`?**
> `start` runs the service immediately but doesn't affect boot behavior; `enable` configures it to start automatically at future boots but doesn't start it right now. Use `enable --now` for both.

**Q7. Why might a script work when run manually but fail when run via cron?**
> Cron jobs run with a minimal environment (limited `$PATH`, no shell profile sourced) — always use full/absolute paths in cron entries.

**Q8. What does a negative nice value require, and why?**
> Root privileges — because a negative value raises a process's scheduling priority above default, and only root is trusted to grant a process more CPU favor than everyone else's default share.

**Q9. How would you find out which process is using TCP port 3000?**
> `lsof -i :3000` (or `sudo ss -ltnp | grep :3000` as an alternative).

**Q10. What does the `TTY` column showing `?` in `ps aux` output indicate?**
> The process has no controlling terminal — it's typically a daemon/background service, not something started interactively from a terminal session.

---

## 22. Revision Summary

- A **process** is a running instance of a program; every process (except PID 1) is created via `fork()` (new PID, duplicate) followed optionally by `exec()` (replaces program code, same PID).
- Processes form a **tree** rooted at PID 1 (`systemd`); view it with `pstree -p` or `ps -ef --forest`.
- **States**: `R` running, `S` sleeping, `D` uninterruptible sleep (unkillable!), `T` stopped, `Z` zombie.
- **`ps`** = snapshot; **`top`/`htop`** = live view. Watch `%CPU`, `%MEM`, `TIME` vs `START`, and `load average` vs `nproc`.
- **Jobs**: `&` backgrounds a command; `Ctrl+Z` suspends; `fg`/`bg`/`jobs` manage them; `&` alone does **not** survive logout — use `nohup` or `disown` for that.
- **Signals**: `SIGTERM` (15, polite) vs `SIGKILL` (9, forced); `SIGSTOP`/`SIGKILL` can never be caught. Use `kill`, `killall`, or `pkill`/`pgrep`.
- **Priority**: `nice`/`renice`, range **-20 (highest) to +19 (lowest)**; only root can go negative.
- **Zombies** = dead but unreaped (parent bug if they pile up); **orphans** = alive, re-parented to `init` automatically — both are normal in small numbers.
- **systemd/systemctl**: `start`/`stop` = right now; `enable`/`disable` = at boot; `enable --now` = both; logs via `journalctl -u <service>`.
- **Scheduling**: `cron` for recurring tasks (always use absolute paths!), `at` for one-time future tasks.
- **`/proc/<pid>/`** exposes live process internals as files — `status`, `cmdline`, `fd/`, `environ`, `maps`.
- Deep-dive tools: `lsof` (open files/ports), `strace` (syscalls), `vmstat`/`iostat` (resource bottlenecks).

**One-line takeaway:** *Every process on Linux is born via `fork()`, identified by a PID, tracked through a lifecycle of states, controllable via signals and priority, and — whether a one-off job or a persistent daemon managed by systemd — ultimately reaped by its parent or by `init` when it exits.*

---

*End of notes — Linux Process Management, compiled as a complete reference for study, interviews, and day-to-day system administration.*
