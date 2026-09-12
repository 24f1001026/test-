# Linux Process Management — Week 3, Lecture 5

> **Course:** Linux Fundamentals — Process Management
> **Week:** 3 | **Lecture:** 5
> **Topic Focus:** Process Control, Job Control, Signals, History, and Exit Codes

---

## Table of Contents

1. [Introduction to Process Management](#1-introduction-to-process-management)
2. [The `sleep` Command](#2-the-sleep-command)
3. [Coprocesses (`coproc`)](#3-coprocesses-coproc)
4. [The `kill` Command](#4-the-kill-command)
5. [Running a Process in the Background using `&`](#5-running-a-process-in-the-background-using-)
6. [Bringing a Process to the Foreground — `fg`](#6-bringing-a-process-to-the-foreground--fg)
7. [`Ctrl + C` — Terminating the Foreground Process](#7-ctrl--c--terminating-the-foreground-process)
8. [Two Ways of Killing a Process](#8-two-ways-of-killing-a-process)
9. [The `jobs` Command](#9-the-jobs-command)
10. [The `top` Command](#10-the-top-command)
11. [`Ctrl + Z` — Suspending a Process](#11-ctrl--z--suspending-a-process)
12. [`echo $-` — Viewing Shell Options](#12-echo---viewing-shell-options)
13. [Child Shell (Subshell)](#13-child-shell-subshell)
14. [Command History](#14-command-history)
15. [`!n` — Re-run Command by History Number](#15-n--re-run-command-by-history-number)
16. [`!!` — Re-run the Last Command](#16---re-run-the-last-command)
17. [Brace Expansion](#17-brace-expansion)
18. [Multiple Commands on a Single Line](#18-multiple-commands-on-a-single-line)
19. [Exit Codes](#19-exit-codes)
20. [Killing a Process Running in a Separate Shell](#20-killing-a-process-running-in-a-separate-shell)
21. [`ps -e` — Listing All Processes](#21-ps--e--listing-all-processes)
22. [Exit Code for Child Processes](#22-exit-code-for-child-processes)
23. [`bc` — Basic Calculator](#23-bc--basic-calculator)
24. [`Ctrl + D` — Quit / Exit (EOF)](#24-ctrl--d--quit--exit-eof)
25. [Why Learn Exit Codes?](#25-why-learn-exit-codes)
26. [Summary](#26-summary)

---

## 1. Introduction to Process Management

In Linux, every program that runs is called a **process**, and it is assigned a unique **Process ID (PID)**. Process management refers to the set of tools and techniques used to **create, monitor, control, suspend, resume, and terminate** processes from the shell.

This lecture builds a practical foundation covering:
- How to run processes in the **foreground** and **background**
- How to **suspend, resume, and kill** processes using signals
- How to view **running processes**
- Shell features like **history**, **brace expansion**, and **exit codes** that support efficient process control

Understanding these concepts is essential for **system administration, shell scripting, and automation**, and forms the backbone of real-world Linux usage.

---

## 2. The `sleep` Command

### Concept
`sleep` is a simple command that **pauses execution for a specified duration**. It is commonly used to simulate long-running processes, add delays in scripts, or test background/foreground process behavior.

### Syntax
```bash
sleep NUMBER[SUFFIX]
```
| Suffix | Meaning |
|--------|---------|
| (none) | Seconds (default) |
| `s`    | Seconds |
| `m`    | Minutes |
| `h`    | Hours |
| `d`    | Days |

### Examples
```bash
sleep 5          # Pauses for 5 seconds
sleep 2m         # Pauses for 2 minutes
sleep 1h         # Pauses for 1 hour
sleep 10 &       # Runs sleep in the background for 10 seconds
```

**Use case:** `sleep` is frequently used in this lecture as a "dummy long process" to demonstrate background jobs, suspending, and killing processes.

---

## 3. Coprocesses (`coproc`)

### Concept
A **coprocess** is a command that is executed **asynchronously (in the background)**, but unlike a normal background job, the shell automatically sets up **pipes** to communicate with its standard input and standard output. This allows the parent shell to **send data to** and **read data from** the coprocess.

### Syntax
```bash
coproc [NAME] command [redirections]
```
- `NAME` (optional) — a name for the coprocess (default is `COPROC`)
- The shell creates an array variable `NAME` containing the file descriptors: `NAME[0]` (read) and `NAME[1]` (write)

### Examples
```bash
# Basic coprocess
coproc mycoproc { sleep 5; echo "Coprocess finished"; }

# Reading output from a coprocess
coproc { cat; }
echo "Hello Coprocess" >&"${COPROC[1]}"
read line <&"${COPROC[0]}"
echo "$line"
```

**Note:** `coproc` is an advanced bash feature mainly used for **inter-process communication (IPC)** within scripts.

---

## 4. The `kill` Command

### Concept
`kill` is used to **send a signal to a process**, most commonly to terminate it. Despite the name, `kill` does not always "kill" a process — it sends a **signal**, and the default signal is `SIGTERM (15)`, which requests graceful termination.

### Syntax
```bash
kill [-signal_name | -signal_number] PID
```

### Common Signals

| Signal Name | Number | Meaning |
|-------------|--------|---------|
| `SIGHUP`    | 1      | Hangup — reload configuration |
| `SIGINT`    | 2      | Interrupt (same as Ctrl+C) |
| `SIGKILL`   | 9      | Force kill (cannot be ignored) |
| `SIGTERM`   | 15     | Graceful termination (default) |
| `SIGSTOP`   | 19     | Pause process (cannot be ignored) |
| `SIGCONT`   | 18     | Resume a paused process |

### Examples
```bash
kill 1234           # Sends SIGTERM to process with PID 1234
kill -9 1234         # Force kill (SIGKILL) process 1234
kill -SIGKILL 1234   # Same as above, using signal name
kill %1              # Kill background job number 1
```

---

## 5. Running a Process in the Background using `&`

### Concept
Appending `&` at the end of a command runs it **in the background**, allowing the shell prompt to return immediately so you can continue working while the process executes.

### Syntax
```bash
command &
```

### Examples
```bash
sleep 100 &
# Output: [1] 23456   -> Job number and PID

firefox &            # Launches Firefox in the background
```

**Tip:** The shell prints the **job number** (in brackets) and the **PID** of the background process immediately after starting it.

---

## 6. Bringing a Process to the Foreground — `fg`

### Concept
`fg` brings a background or suspended job **back to the foreground**, meaning it now has control of the terminal again and will block further input until it finishes or is stopped.

### Syntax
```bash
fg [%jobspec]
```

### Examples
```bash
fg          # Brings the most recent background job to foreground
fg %1       # Brings job number 1 to the foreground
fg %sleep   # Brings the job matching command name "sleep" to foreground
```

---

## 7. `Ctrl + C` — Terminating the Foreground Process

### Concept
`Ctrl + C` sends the **`SIGINT`** (Signal Interrupt) to the **currently running foreground process**, requesting it to stop immediately.

### Syntax
There is no typed command — it is a **keyboard shortcut**:
```
Ctrl + C
```

### Example
```bash
sleep 100
# While it's running, press Ctrl+C
# Output: ^C
# The sleep process is terminated immediately
```

**Note:** If a program has been coded to **ignore** `SIGINT`, `Ctrl+C` will not stop it — in that case, `SIGKILL` (`kill -9`) must be used.

---

## 8. Two Ways of Killing a Process

### Concept
There are generally **two primary methods** to terminate a running process in Linux:

| Method | Description | Example |
|--------|-------------|---------|
| **1. Keyboard Interrupt** | Works only on the **foreground** process using `Ctrl+C` (SIGINT) or `Ctrl+Z` (SIGTSTP, suspend not kill) | `Ctrl + C` |
| **2. `kill` Command** | Works on **any process** (foreground, background, or in another terminal) by specifying its PID or job number | `kill -9 1234` |

### Detailed Comparison

| Aspect | Ctrl+C | kill command |
|--------|--------|--------------|
| Target | Only current foreground process | Any process (via PID/job) |
| Signal sent | SIGINT (2) | Any signal (default SIGTERM) |
| Requires PID | No | Yes (or job number) |
| Works across terminals | No | Yes |

---

## 9. The `jobs` Command

### Concept
`jobs` lists all the **background and suspended jobs** associated with the **current shell session**, along with their job numbers and statuses.

### Syntax
```bash
jobs [options]
```

| Option | Meaning |
|--------|---------|
| `-l`   | Show PIDs along with job info |
| `-r`   | Show only running jobs |
| `-s`   | Show only stopped jobs |

### Examples
```bash
sleep 100 &
sleep 200 &
jobs
# Output:
# [1]-  Running    sleep 100 &
# [2]+  Running    sleep 200 &

jobs -l
# Output includes PID as well
```

**Note:** The `+` symbol marks the **current job**, and `-` marks the **previous job**.

---

## 10. The `top` Command

### Concept
`top` is an **interactive, real-time process monitoring tool** that displays system resource usage (CPU, memory) and a live list of running processes, refreshed periodically.

### Syntax
```bash
top [options]
```

### Common Interactive Keys (while `top` is running)

| Key | Action |
|-----|--------|
| `q` | Quit `top` |
| `k` | Kill a process (prompts for PID) |
| `P` | Sort by CPU usage |
| `M` | Sort by Memory usage |
| `r` | Renice (change priority of) a process |
| `h` | Help |

### Example
```bash
top          # Launches the interactive process viewer
top -u user1 # Show only processes owned by user1
```

**Displayed Information includes:** Total tasks, CPU usage %, memory usage, load average, and a live table of processes with PID, USER, %CPU, %MEM, TIME, and COMMAND.

---

## 11. `Ctrl + Z` — Suspending a Process

### Concept
`Ctrl + Z` sends the **`SIGTSTP`** signal, which **suspends (pauses)** the current foreground process rather than terminating it. The process remains in memory and can later be resumed using `fg` (foreground) or `bg` (background).

### Syntax
```
Ctrl + Z
```

### Example
```bash
sleep 500
# Press Ctrl+Z
# Output: [1]+  Stopped   sleep 500

bg %1        # Resume it in the background
# OR
fg %1        # Resume it in the foreground
```

**Difference from Ctrl+C:** `Ctrl+C` **terminates** the process; `Ctrl+Z` only **pauses** it — the process can be resumed later.

---

## 12. `echo $-` — Viewing Shell Options

### Concept
The special shell variable `$-` holds the **current set of active shell option flags** (set using `set -o` or startup flags). Printing it tells you the shell's current mode.

### Syntax
```bash
echo $-
```

### Example
```bash
echo $-
# Example Output: himBHs

# Common flags meaning:
# h - hashall (remember command locations)
# i - interactive shell
# m - monitor mode (job control enabled)
# B - braceexpand
# H - histexpand (enables !! and !n)
# s - commands read from stdin
```

| Flag | Meaning |
|------|---------|
| `i` | Interactive shell |
| `m` | Monitor mode — enables job control (`fg`, `bg`, `jobs`) |
| `H` | History expansion enabled (`!!`, `!n`) |
| `B` | Brace expansion enabled |
| `h` | Remembers command locations (hashing) |

---

## 13. Child Shell (Subshell)

### Concept
A **child shell** (or **subshell**) is a new shell process spawned from the current (parent) shell. It **inherits** the parent's environment variables but runs **independently** — changes made in the child shell (like `cd` or variable assignments) do **not** affect the parent shell.

### Syntax
```bash
bash                # Starts a new child shell
( command )          # Runs command(s) in a subshell
```

### Examples
```bash
echo $$              # Shows current shell's PID
bash                  # Spawns a child shell
echo $$               # Shows the new child shell's PID (different!)
exit                  # Exits child shell, returns to parent

# Subshell using parentheses
(cd /tmp && ls)       # 'cd' only affects the subshell, not the parent shell
pwd                   # Parent shell's directory is unchanged
```

**Key Point:** Use `pstree` or `ps -f` to visually confirm the parent-child relationship between shells.

---

## 14. Command History

### Concept
Bash keeps a record of previously executed commands in the **history list**, stored in memory during the session and saved to a history file (usually `~/.bash_history`) upon exit.

### Syntax
```bash
history [options]
```

| Option | Meaning |
|--------|---------|
| `history` | Show all commands in history |
| `history N` | Show last N commands |
| `history -c` | Clear the history list |
| `history -d N` | Delete entry number N |

### Examples
```bash
history          # Lists all previous commands with numbers
history 10       # Shows the last 10 commands
history -c       # Clears the entire history
```

**Related file/variables:**
- `HISTFILE` — path to the history file (default `~/.bash_history`)
- `HISTSIZE` — number of commands kept in memory
- `HISTFILESIZE` — number of commands stored in the history file

---

## 15. `!n` — Re-run Command by History Number

### Concept
`!n` re-executes the command at **history line number `n`**, saving time from retyping long commands.

### Syntax
```bash
!n
```

### Example
```bash
history
#  12  ls -l
#  13  cd /var/log
#  14  sleep 100 &

!12          # Re-runs "ls -l"
!14          # Re-runs "sleep 100 &"
```

---

## 16. `!!` — Re-run the Last Command

### Concept
`!!` repeats the **most recently executed command** exactly as it was run. It is commonly used with `sudo` to re-run a command with elevated privileges after a permission error.

### Syntax
```bash
!!
```

### Examples
```bash
apt update
# Permission denied

sudo !!
# Executes: sudo apt update
```

---

## 17. Brace Expansion

### Concept
**Brace expansion** is a shell mechanism to generate **multiple strings** from a pattern containing braces `{}`, without needing loops. It is purely a **text-generation** feature — it happens before the command is executed.

### Syntax
```bash
{item1,item2,item3}        # List expansion
{start..end}                # Range expansion
{start..end..step}          # Range expansion with step
```

### Examples
```bash
echo file{1,2,3}.txt
# Output: file1.txt file2.txt file3.txt

echo {1..5}
# Output: 1 2 3 4 5

echo {a..e}
# Output: a b c d e

echo {10..0..2}
# Output: 10 8 6 4 2 0

mkdir project{Frontend,Backend,Database}
# Creates: projectFrontend  projectBackend  projectDatabase

touch file{1..3}.txt
# Creates: file1.txt file2.txt file3.txt
```

---

## 18. Multiple Commands on a Single Line

### Concept
Bash allows chaining **multiple commands** on a single line using special operators, controlling whether the next command runs **regardless of**, **only on success of**, or **only on failure of** the previous one.

### Syntax & Operators

| Operator | Meaning | Executes Next Command When... |
|----------|---------|-------------------------------|
| `;`  | Sequential execution | Always, regardless of previous result |
| `&&` | Logical AND | Previous command **succeeded** (exit code 0) |
| `\|\|` | Logical OR | Previous command **failed** (exit code ≠ 0) |
| `&`  | Background execution | Runs command in background, continues immediately |

### Examples
```bash
echo "Hello"; echo "World"
# Both always run, one after another

mkdir newdir && cd newdir
# 'cd' runs only if 'mkdir' succeeds

ls /nonexistent || echo "Directory not found"
# Second command runs only because first fails

sleep 5 & echo "Running in background"
# sleep runs in background while echo runs immediately
```

---

## 19. Exit Codes

### Concept
Every command in Linux returns an **exit code** (also called **exit status**) when it finishes — a number between **0 and 255** that indicates whether it succeeded or failed.

- `0` → **Success**
- **Non-zero (1–255)** → **Failure** (different numbers can represent different error types)

### Syntax
```bash
command
echo $?     # Prints exit code of the last executed command
```

### Examples
```bash
ls /home
echo $?
# Output: 0   (success)

ls /nonexistent
echo $?
# Output: 2   (No such file or directory)

grep "pattern" file.txt
echo $?
# 0 = pattern found, 1 = pattern not found, 2 = error (e.g., file not found)
```

### Common Exit Code Meanings

| Exit Code | Meaning |
|-----------|---------|
| `0`   | Success |
| `1`   | General error |
| `2`   | Misuse of shell command |
| `126` | Command found but not executable |
| `127` | Command not found |
| `130` | Terminated by Ctrl+C (SIGINT) |
| `137` | Killed (SIGKILL, 128+9) |

---

## 20. Killing a Process Running in a Separate Shell

### Concept
A process started in **another terminal/shell window** cannot be stopped with `Ctrl+C` from your current terminal, since keyboard signals only affect the **foreground process of that specific terminal**. Instead, you must find its **PID** and use `kill`.

### Syntax
```bash
ps -ef | grep process_name     # Step 1: Find the PID
kill -9 PID                    # Step 2: Kill it
```

### Example
```bash
# In Terminal 1:
sleep 1000

# In Terminal 2:
ps -ef | grep sleep
# Output: user   4521  1032  0  10:15 pts/1  00:00:00 sleep 1000

kill -9 4521
# The sleep process running in Terminal 1 is terminated
```

---

## 21. `ps -e` — Listing All Processes

### Concept
`ps` (process status) displays information about active processes. The `-e` option shows **every process** currently running on the system, not just those from the current shell/session.

### Syntax
```bash
ps -e [additional_options]
```

### Common Variants

| Command | Description |
|---------|-------------|
| `ps` | Shows processes for the current shell only |
| `ps -e` | Shows every process on the system |
| `ps -ef` | Full-format listing of all processes |
| `ps -eu` | Shows processes with user info |
| `ps aux` | BSD-style, detailed listing (CPU, MEM, etc.) |
| `ps -e --forest` | Shows process tree (parent-child hierarchy) |

### Examples
```bash
ps -e
#   PID TTY          TIME CMD
#     1 ?        00:00:02 systemd
#   452 ?        00:00:00 sshd
#  4521 pts/1    00:00:00 sleep

ps -ef | grep firefox      # Find all Firefox-related processes
ps -e --forest              # View process hierarchy as a tree
```

---

## 22. Exit Code for Child Processes

### Concept
When a **parent process** spawns a **child process**, the child's exit code can be captured by the parent (commonly using the `wait` command in scripts). This allows a script to check whether a background/child task completed successfully.

### Syntax
```bash
command &
wait $!      # Wait for the last background process and get its exit code
echo $?      # Exit code of that child process
```

### Examples
```bash
sleep 5 &
child_pid=$!
wait $child_pid
echo "Child exited with code: $?"

# Example with a failing child command
false &
wait $!
echo $?
# Output: 1   (since 'false' always returns exit code 1)
```

**Key Point:** `$!` stores the PID of the **most recently started background process**, and `wait` pauses the parent until that child finishes, after which `$?` reflects the **child's** exit code.

---

## 23. `bc` — Basic Calculator

### Concept
`bc` (basic calculator) is a command-line utility for performing **arithmetic calculations**, including floating-point math, from the shell — since bash itself only handles integer arithmetic natively.

### Syntax
```bash
bc [options]
echo "expression" | bc
bc <<< "expression"
```

### Examples
```bash
echo "5 + 3" | bc
# Output: 8

echo "10 / 3" | bc
# Output: 3   (integer division by default)

echo "scale=2; 10/3" | bc
# Output: 3.33   (scale sets decimal precision)

bc <<< "2^10"
# Output: 1024

bc
# Enters interactive mode; type expressions directly
# 5*6
# 30
# quit
```

| Option | Meaning |
|--------|---------|
| `scale=N` | Sets number of decimal places |
| `-l` | Loads math library (adds functions like sine, cosine, sqrt) |

---

## 24. `Ctrl + D` — Quit / Exit (EOF)

### Concept
`Ctrl + D` sends an **EOF (End Of File)** signal to the terminal's standard input. In an interactive shell, this typically **logs out / exits the shell**. In programs reading from stdin (like `cat` or `bc`), it signals "no more input."

### Syntax
```
Ctrl + D
```

### Examples
```bash
bash               # Start a shell
# Press Ctrl+D
# Effect: exits the shell (equivalent to typing 'exit')

cat
Hello
World
# Press Ctrl+D
# Effect: ends input, cat prints and terminates
```

**Difference from `exit` command:** `exit` explicitly terminates the shell with an optional exit code; `Ctrl+D` sends EOF, which achieves the same result for an empty input line in an interactive shell.

---

## 25. Why Learn Exit Codes?

### Concept
Exit codes are the **foundation of automation and scripting logic** in Linux. Understanding them lets you:

1. **Build reliable shell scripts** — decide next steps based on success/failure (`&&`, `||`, `if` statements)
2. **Debug failures** — a specific non-zero code often tells you *why* something failed
3. **Automate workflows** — CI/CD pipelines, cron jobs, and deployment scripts rely entirely on exit codes to know whether to proceed, retry, or alert
4. **Chain commands intelligently** — using `&&` and `||` to build conditional logic without writing full `if` blocks
5. **Monitor and log system health** — many monitoring tools check the exit code of health-check scripts

### Example: Practical Use in Scripting
```bash
#!/bin/bash
backup_database
if [ $? -eq 0 ]; then
    echo "Backup successful"
else
    echo "Backup failed with exit code $?"
    exit 1
fi
```

This demonstrates why exit codes are not just theoretical — they are actively used to control real automation logic.

---

## 26. Summary

| # | Topic | Key Takeaway |
|---|-------|---------------|
| 1 | `sleep` | Pauses execution for a set time; used to simulate long processes |
| 2 | `coproc` | Runs a command asynchronously with bidirectional pipe communication |
| 3 | `kill` | Sends signals (default SIGTERM) to processes by PID |
| 4 | `&` | Runs a command in the background, freeing the terminal |
| 5 | `fg` | Brings a background/suspended job to the foreground |
| 6 | `Ctrl+C` | Sends SIGINT to terminate the foreground process |
| 7 | Two ways of killing | Keyboard interrupt (foreground only) vs `kill` command (any process) |
| 8 | `jobs` | Lists background/suspended jobs of the current shell |
| 9 | `top` | Real-time, interactive process and resource monitor |
| 10 | `Ctrl+Z` | Sends SIGTSTP to suspend (not kill) the foreground process |
| 11 | `echo $-` | Displays currently active shell option flags |
| 12 | Child shell | A subshell inherits but doesn't affect the parent shell's environment |
| 13 | History | Bash records past commands for reuse and auditing |
| 14 | `!n` | Re-executes a specific command from history by number |
| 15 | `!!` | Re-executes the most recent command |
| 16 | Brace Expansion | Generates multiple strings/patterns without loops |
| 17 | Multiple Commands | `;`, `&&`, `\|\|`, `&` control command chaining logic |
| 18 | Exit Codes | Numeric result (0 = success, non-zero = failure) of every command |
| 19 | Kill in separate shell | Use `ps` to find PID, then `kill` it from any terminal |
| 20 | `ps -e` | Lists every running process on the system |
| 21 | Exit code (child) | `$!` + `wait` lets a parent capture a child process's exit status |
| 22 | `bc` | Command-line calculator for arithmetic beyond bash's integer limits |
| 23 | `Ctrl+D` | Sends EOF; exits shell or ends stdin input |
| 24 | Why exit codes matter | Core to scripting, automation, debugging, and CI/CD reliability |

### Final Exam-Prep Notes
- Always remember: **`Ctrl+C` kills**, **`Ctrl+Z` suspends**, **`Ctrl+D` sends EOF**.
- `kill` sends **signals**, not just "kills" — default is `SIGTERM (15)`, forceful is `SIGKILL (9)`.
- `$?` = exit code of the **last command**; `$!` = PID of the **last background command**.
- `jobs`, `fg`, `bg` only work on jobs of the **current shell session** — use `ps` + `kill` for processes elsewhere.
- Brace expansion and history expansion (`!!`, `!n`) are **shell conveniences**, not commands themselves.

---

*End of Notes — Linux Process Management, Week 3, Lecture 5*
