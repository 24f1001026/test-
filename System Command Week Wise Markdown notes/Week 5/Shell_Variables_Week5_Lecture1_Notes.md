# Shell Variables — Week 5, Lecture 1

## Table of Contents

1. [Introduction to Shell Variables](#1-introduction-to-shell-variables)
2. [The `echo` Command](#2-the-echo-command)
   - [2.1 Printing Strings](#21-printing-strings)
   - [2.2 Printing Variable Values](#22-printing-variable-values)
   - [2.3 `echo` with Multiple Arguments](#23-echo-with-multiple-arguments)
   - [2.4 `echo` with Double Quotes](#24-echo-with-double-quotes)
   - [2.5 Multi-line `echo` and Nested Quotes](#25-multi-line-echo-and-nested-quotes)
   - [2.6 Single Quotes — Variables Not Expanded](#26-single-quotes--variables-not-expanded)
   - [2.7 Escaping `$` with a Backslash](#27-escaping--with-a-backslash)
3. [Frequently Used Shell Variables](#3-frequently-used-shell-variables)
4. [Viewing Environment Variables](#4-viewing-environment-variables-printenv-env-set)
5. [The `date` Command](#5-the-date-command)
   - [5.1 Basic Usage](#51-basic-usage)
   - [5.2 `date -R`](#52-date--r)
6. [Running Unaliased / Original Commands](#6-running-unaliased--original-commands)
7. [Special Shell Variables](#7-special-shell-variables)
   - [7.1 `$0` — Shell/Script Name](#71-0--shellscript-name)
   - [7.2 Positional Parameters `$1` to `$9`](#72-positional-parameters-1-to-9)
   - [7.3 `$#` — Number of Arguments](#73--number-of-arguments)
   - [7.4 `$@` — All Arguments](#74--all-arguments)
   - [7.5 `$$` — Process ID](#75---process-id)
   - [7.6 `$?` — Return/Exit Code](#76--return-exit-code)
   - [7.7 `$-` — Active Shell Flags](#77---active-shell-flags)
8. [Process Monitoring with `ps`](#8-process-monitoring-with-ps)
   - [8.1 `ps`](#81-ps)
   - [8.2 `ps --forest`](#82-ps---forest)
   - [8.3 `ps -ef`](#83-ps--ef)
   - [8.4 `ps -f`](#84-ps--f)
   - [8.5 `ps -e`](#85-ps--e)
9. [Process Control](#9-process-control)
10. [Program Exit Codes](#10-program-exit-codes)
11. [Flags Set in Bash](#11-flags-set-in-bash)
12. [Summary](#12-summary)

---

## 1. Introduction to Shell Variables

A **shell variable** is a named storage location, maintained by the shell (e.g., Bash), that holds a value such as text, a number, or a file path. Shell variables let the shell keep track of configuration, environment details, and the state of running programs, and they let users and scripts store and reuse data without hard-coding it.

There are broadly two categories covered in this lecture:

| Category | Description | Example |
|---|---|---|
| **User/Environment variables** | Hold system or user configuration information | `$HOME`, `$PATH` |
| **Special shell variables** | Automatically maintained by the shell to reflect its own state | `$$`, `$?`, `$0`, `$-` |

> **Note:** In Bash, a variable is referenced (its value is read) by prefixing its name with `$`. When *setting* a variable you do **not** use `$` (e.g., `NAME=value`), but when *reading* it you do (e.g., `echo $NAME`).

---

## 2. The `echo` Command

### Concept
`echo` is a built-in shell command used to display (print) text or the values of variables to the terminal (standard output). It is one of the most frequently used commands for debugging scripts and confirming variable values.

### Syntax
```bash
echo [options] [string | $variable]
```

### 2.1 Printing Strings
You can print any literal text directly.

**Syntax:**
```bash
echo "text to print"
```

**Example:**
```bash
$ echo hello, world
hello, world
```

### 2.2 Printing Variable Values
To print the *value* stored inside a variable, prefix the variable name with `$`.

**Syntax:**
```bash
echo $VARIABLE_NAME
```

**Example:**
```bash
$ echo $HOME
/home/username
```

> **Tip:** Wrapping the variable in quotes — `echo "$HOME"` — is a best practice in scripts because it prevents word-splitting issues if the value contains spaces.

### 2.3 `echo` with Multiple Arguments

**Concept:** `echo` can take more than one argument at once. It prints each argument separated by a single space, in the order given.

**Syntax:**
```bash
echo arg1 arg2 arg3
```

**Example:**
```bash
$ echo hello world today
hello world today

$ echo $USER $HOME
john /home/john
```

### 2.4 `echo` with Double Quotes

**Concept:** Wrapping text in double quotes (`"..."`) tells the shell to treat the enclosed content as a single string, while still **expanding** (substituting) any variables inside it.

**Syntax:**
```bash
echo "some text $VARIABLE more text"
```

**Example:**
```bash
$ echo "Welcome $USER, your home is $HOME"
Welcome john, your home is /home/john
```

> Double quotes preserve spacing exactly as typed and are the safest general-purpose way to `echo` a mix of text and variables.

### 2.5 Multi-line `echo` and Nested Quotes

**Concept:** `echo` can print text across multiple lines, and you can **nest** one quote type inside the other (single quotes inside double quotes, or vice-versa) to include literal quote characters in the output.

**Syntax:**
```bash
echo 'text with "double quotes" inside single quotes'
echo "text with 'single quotes' inside double quotes"
```

**Example:**
```bash
$ echo 'She said "hello"'
She said "hello"

$ echo "It's a great day"
It's a great day

$ echo "Line 1
Line 2"
Line 1
Line 2
```

> When a quote is left **unclosed**, Bash waits for you to close it, letting you type text across several lines before pressing Enter to execute — this is how multi-line `echo` strings are typically produced interactively.

### 2.6 Single Quotes — Variables Not Expanded

**Concept:** Unlike double quotes, **single quotes (`'...'`) suppress all expansion**. Anything inside single quotes — including `$VARIABLE` references — is treated as pure literal text.

**Syntax:**
```bash
echo '$VARIABLE_NAME'
```

**Example:**
```bash
$ echo "$USER"
john

$ echo '$USER'
$USER
```

> **Exam tip:** This is one of the most commonly tested distinctions — double quotes *expand* variables, single quotes *do not*.

### 2.7 Escaping `$` with a Backslash

**Concept:** If you want to print a literal `$` character inside double quotes (or without quotes) instead of triggering variable expansion, **escape** it with a backslash (`\$`).

**Syntax:**
```bash
echo \$VARIABLE_NAME
echo "cost: \$50"
```

**Example:**
```bash
$ echo \$USER
$USER

$ echo "The price is \$100"
The price is $100
```

| Quoting Style | Variables Expanded? | Example Input | Output |
|---|---|---|---|
| No quotes | Yes | `echo $USER` | `john` |
| Double quotes `"..."` | Yes | `echo "$USER"` | `john` |
| Single quotes `'...'` | No | `echo '$USER'` | `$USER` |
| Escaped `\$` | No (treated literally) | `echo \$USER` | `$USER` |

---

## 3. Frequently Used Shell Variables

These are common **environment variables** that the shell sets automatically and that scripts frequently rely on.

| Variable | Meaning | Example Output |
|---|---|---|
| `$USER` | The name of the currently logged-in user (the standard, most portable variable) | `john` |
| `$USERNAME` | Also holds the current username on many systems (less portable than `$USER`) | `john` |
| `$HOME` | The path to the current user's home directory | `/home/john` |
| `$HOSTNAME` | The network name (hostname) of the machine | `john-laptop` |
| `$PWD` | The present working directory (current folder) | `/home/john/projects` |
| `$PATH` | A colon-separated list of directories the shell searches for executable commands | `/usr/local/bin:/usr/bin:/bin` |

### Examples
```bash
$ echo $USER
john

$ echo $USERNAME
john

$ echo $HOME
/home/john

$ echo $HOSTNAME
john-laptop

$ echo $PWD
/home/john/projects

$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/bin
```

> **Concept check:** `$PWD` updates automatically every time you `cd` into a new directory — you never set it manually in normal use. `$PATH` is consulted by the shell every time you type a command name, to locate the matching executable file.

---

## 4. Viewing Environment Variables: `printenv`, `env`, `set`

### Concept
Besides `echo`-ing a single variable, Bash provides commands to **list all currently defined variables** at once.

| Command | Purpose |
|---|---|
| `printenv` | Prints all exported environment variables (or a specific one if named) |
| `env` | Displays the current environment, or runs a command in a modified environment |
| `set` | Lists **all** shell variables, including local (non-exported) ones, and shell functions |

### Syntax
```bash
printenv          # list all environment variables
printenv HOME     # list a specific variable
env               # list environment variables
set               # list all shell variables + functions
```

### Example
```bash
$ printenv HOME
/home/john

$ env | grep PATH
PATH=/usr/local/sbin:/usr/local/bin:/usr/bin:/bin

$ set | grep BASH_VERSION
BASH_VERSION='5.1.16(1)-release'
```

---

## 5. The `date` Command

### Concept
`date` is a standalone utility (not a shell variable) that displays or sets the current system date and time. It is included in this lecture because it's frequently combined with shell variables in scripts (e.g., for timestamped log files).

### 5.1 Basic Usage

**Syntax:**
```bash
date
```

**Example:**
```bash
$ date
Sat Aug 22 11:32:07 IST 2026
```

### 5.2 `date -R`

**Concept:** The `-R` option prints the date in **RFC 2822** format — a standardized format commonly used in email headers and logs.

**Syntax:**
```bash
date -R
```

**Example:**
```bash
$ date -R
Sat, 22 Aug 2026 11:32:07 +0530
```

| Command | Output Format |
|---|---|
| `date` | Human-readable local format (e.g., `Sat Aug 22 11:32:07 IST 2026`) |
| `date -R` | RFC 2822 format (e.g., `Sat, 22 Aug 2026 11:32:07 +0530`) |

---

## 6. Running Unaliased / Original Commands

### Concept
Many common commands (like `ls`, `date`, `rm`) are often **aliased** in a user's shell configuration (e.g., `alias date='date +%A'`) to add extra default options. To bypass an alias and run the **original** version of the command, you have two options:

| Method | How It Works |
|---|---|
| Prefix with a backslash `\command` | Tells Bash to ignore any alias with that name for this one invocation |
| Use the **full path** to the executable | Directly calls the binary, bypassing alias lookup entirely |

### Syntax
```bash
\command_name           # run unaliased version
/full/path/to/command   # run by full path
```

### Example
```bash
$ alias date='date +%A'
$ date
Saturday

$ \date
Sat Aug 22 11:32:07 IST 2026

$ /usr/bin/date
Sat Aug 22 11:32:07 IST 2026
```

> **Exam tip:** `\command` and the full-path method both skip the **alias**, but note that `\command` still respects functions/builtins resolution rules for that name — using the absolute path (`/usr/bin/date`) is the most explicit and unambiguous way to guarantee you're running the original binary.

---

## 7. Special Shell Variables

**Concept:** Special shell variables are automatically maintained by Bash to describe the *state of the shell itself* or the **arguments passed to a script/function** — not general user configuration. They cannot generally be set manually in the normal sense; the shell updates them as it runs.

| Variable | Meaning |
|---|---|
| `$0` | Name of the shell or the script currently executing |
| `$1` to `$9` | Positional parameters — the individual arguments passed to a script or function |
| `$#` | The total number of positional parameters (arguments) passed |
| `$@` | All positional parameters, expanded as **separate**, individually-quoted words |
| `$$` | Process ID (PID) of the current shell |
| `$?` | Exit/return status of the most recently executed command |
| `$-` | Current flags/options set in the running Bash shell |

### 7.1 `$0` — Shell/Script Name
**Syntax:**
```bash
echo $0
```
**Example:**
```bash
$ echo $0
/bin/bash
```
When run inside a script named `myscript.sh`, `$0` would print `myscript.sh` (or its path).

### 7.2 Positional Parameters `$1` to `$9`

**Concept:** When a script or function is called with arguments, Bash automatically stores the first nine arguments in the variables `$1` through `$9`, in the order they were supplied.

**Syntax:**
```bash
./script.sh arg1 arg2 arg3
# inside script.sh:
echo $1   # arg1
echo $2   # arg2
echo $3   # arg3
```

**Example (`greet.sh`):**
```bash
#!/bin/bash
echo "First argument: $1"
echo "Second argument: $2"
```
```bash
$ ./greet.sh Alice Bob
First argument: Alice
Second argument: Bob
```

### 7.3 `$#` — Number of Arguments

**Concept:** `$#` holds the **count** of positional parameters (arguments) that were passed to the script or function.

**Syntax:**
```bash
echo $#
```

**Example (`count.sh`):**
```bash
#!/bin/bash
echo "You passed $# argument(s)"
```
```bash
$ ./count.sh Alice Bob Carol
You passed 3 argument(s)
```

### 7.4 `$@` — All Arguments

**Concept:** `$@` expands to **all** positional parameters at once, with each one preserved as a separate word (this matters when arguments contain spaces and the variable is quoted as `"$@"`).

**Syntax:**
```bash
echo $@
echo "$@"
```

**Example (`showall.sh`):**
```bash
#!/bin/bash
echo "All arguments: $@"
```
```bash
$ ./showall.sh Alice Bob Carol
All arguments: Alice Bob Carol
```

> **Exam tip:** `$#` tells you *how many* arguments there are, `$@` gives you the arguments *themselves*, and `$1`...`$9` let you access them *individually*.

### 7.5 `$$` — Process ID
**Concept:** Every running shell (or script) has a unique Process ID (PID) assigned by the operating system. `$$` returns the PID of the current shell.

**Syntax:**
```bash
echo $$
```
**Example:**
```bash
$ echo $$
4821
```

### 7.6 `$?` — Return/Exit Code
**Concept:** After any command finishes, it returns a numeric **exit code** to the shell indicating success or failure. `$?` holds the exit code of the *most recently executed* command, and is one of the most useful variables for scripting (error checking). `0` indicates success, and any **non-zero** value indicates some kind of failure.

**Syntax:**
```bash
command
echo $?
```
**Example:**
```bash
$ ls /etc
... (listing) ...
$ echo $?
0

$ ls /nonexistent
ls: cannot access '/nonexistent': No such file or directory
$ echo $?
2
```
> See [Section 10](#10-program-exit-codes) for the full meaning of each exit code number.

### 7.7 `$-` — Active Shell Flags
**Concept:** Shows which configuration **flags/options** are currently active in the running shell (see [Section 11](#11-flags-set-in-bash) for what each letter means).

**Syntax:**
```bash
echo $-
```
**Example:**
```bash
$ echo $-
himBH
```
This output means the flags `h`, `i`, `m`, `B`, and `H` are currently active in this shell session.

---

## 8. Process Monitoring with `ps`

### Concept
`ps` (**process status**) displays information about currently running processes. It is essential for checking what is running on a system, including PIDs, parent-child relationships, and resource usage — complementary to the process-control tools in [Section 9](#9-process-control).

### 8.1 `ps`

**Syntax:**
```bash
ps
```
**Example:**
```bash
$ ps
  PID TTY          TIME CMD
 4821 pts/0    00:00:00 bash
 5321 pts/0    00:00:00 ps
```
By default, `ps` (with no options) shows only the processes running in the **current terminal session**, owned by the current user.

### 8.2 `ps --forest`

**Concept:** Displays processes in a **tree-like (ASCII-art) hierarchy**, making parent-child relationships between processes visually clear.

**Syntax:**
```bash
ps --forest
ps -ef --forest
```
**Example:**
```bash
$ ps -ef --forest
UID   PID  PPID  CMD
root    1     0  /sbin/init
root  450     1  ├─/usr/sbin/sshd
john  600   450  │ └─sshd: john@pts/0
john  601   600  │   └─bash
john  700   601  │     └─ps -ef --forest
```

### 8.3 `ps -ef`

**Concept:** Shows a **full-format listing** of **every** process running on the system (`-e` = all processes, `-f` = full/extended details such as UID, PPID, and start time).

**Syntax:**
```bash
ps -ef
```
**Example:**
```bash
$ ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 09:00 ?        00:00:02 /sbin/init
john       601     1  0 09:05 pts/0    00:00:00 bash
john       702   601  0 09:10 pts/0    00:00:00 ps -ef
```

### 8.4 `ps -f`

**Concept:** Shows a **full-format listing** but limited to the **current user's** processes in the current session (unlike `-ef`, which shows every process on the system).

**Syntax:**
```bash
ps -f
```
**Example:**
```bash
$ ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
john       601     1  0 09:05 pts/0    00:00:00 bash
john       705   601  0 09:12 pts/0    00:00:00 ps -f
```

### 8.5 `ps -e`

**Concept:** Lists **every** process currently running on the system, but in the shorter default column format (PID, TTY, TIME, CMD) rather than the full format.

**Syntax:**
```bash
ps -e
```
**Example:**
```bash
$ ps -e
  PID TTY          TIME CMD
    1 ?        00:00:02 init
  450 ?        00:00:00 sshd
  601 pts/0    00:00:00 bash
  706 pts/0    00:00:00 ps
```

### `ps` Options Quick Reference

| Option | Meaning |
|---|---|
| `ps` | Processes in the current terminal/session only |
| `ps -e` | All processes on the system, short format |
| `ps -f` | Current user's processes, full format |
| `ps -ef` | All processes on the system, full format |
| `ps --forest` | Displays results as a parent-child tree |

---

## 9. Process Control

**Concept:** Process control refers to the tools Bash provides to manage **jobs** (running programs) — running them in the background, bringing them to the foreground, monitoring them, and terminating them.

| Tool | Purpose |
|---|---|
| `&` | Runs a job in the **background**, freeing the terminal for other commands |
| `fg` | Brings a background job back to the **foreground** |
| `coproc` | Runs a command as a **co-process**, allowing bidirectional communication between the shell and the process |
| `jobs` | Lists all background/suspended jobs associated with the current shell session |
| `top` | Displays a real-time, live view of running processes and system resource usage |
| `kill` | Sends a signal to a process (commonly used to terminate it) |

### Syntax & Examples

**Run a job in the background:**
```bash
$ sleep 100 &
[1] 5321
```
The `[1]` is the job number and `5321` is the PID.

**Bring the last background job to the foreground:**
```bash
$ fg
```

**List current jobs:**
```bash
$ jobs
[1]+  Running     sleep 100 &
```

**Monitor system processes live:**
```bash
$ top
```

**Terminate a process by PID:**
```bash
$ kill 5321
```

**Co-process example:**
```bash
$ coproc mycoproc { cat; }
```
This starts `cat` as a co-process named `mycoproc`, with file descriptors available in the shell to read/write to it.

> **Exam tip:** `kill` by default sends signal `SIGTERM` (15) — a *polite* request to stop. `kill -9` sends `SIGKILL`, an *immediate, forceful* termination (see exit code `137` in the next section).

---

## 10. Program Exit Codes

**Concept:** When any command or program finishes execution, it returns a numeric exit status to the shell, stored in `$?`. This number tells you **why** or **how** the program ended.

**Syntax (to check the code of the last command):**
```bash
echo $?
```

### Exit Code Reference Table

| Exit Code | Meaning |
|---|---|
| `0` | Success — the command completed without errors |
| `1` | General failure (a catch-all for common errors) |
| `2` | Misuse of a shell command (e.g., wrong syntax/arguments) |
| `126` | Command found, but **cannot be executed** (e.g., permission denied) |
| `127` | Command **not found** (e.g., typo, not in `$PATH`) |
| `130` | Process was terminated by **Ctrl+C** (SIGINT) |
| `137` | Process was terminated using **`kill -9`** (SIGKILL) |

### Examples
```bash
$ true
$ echo $?
0

$ false
$ echo $?
1

$ nonexistentcommand
bash: nonexistentcommand: command not found
$ echo $?
127

$ chmod -x script.sh
$ ./script.sh
bash: ./script.sh: Permission denied
$ echo $?
126
```

> **Why this matters for exams:** Codes `130` and `137` are especially important because they reveal *how* a process died: `130` = user pressed Ctrl+C; `137` = the process was forcefully killed with `kill -9`.

---

## 11. Flags Set in Bash

**Concept:** Bash can run with various **options (flags)** enabled or disabled, which change its behavior (e.g., whether it's interactive, whether job control is active). The currently active flags are shown by `echo $-` (see [Section 7.7](#77---active-shell-flags)).

### Flags Reference Table

| Flag | Meaning |
|---|---|
| `h` | Locate and remember (hash) the location of commands as they are looked up |
| `B` | Brace expansion is enabled (e.g., `{a,b,c}` expansion) |
| `i` | Shell is running in **interactive mode** |
| `m` | **Job control** is enabled (allows use of `fg`, `bg`, `jobs`) |
| `H` | `!`-style **history substitution** is enabled |
| `s` | Commands are being read from **standard input (stdin)** |
| `c` | Commands are being read from the **command-line arguments** (`-c` option) |

### Syntax
```bash
echo $-
```

### Example
```bash
$ echo $-
himBH
```
**Breakdown of this output:**

| Character | Flag Meaning |
|---|---|
| `h` | Command hashing enabled |
| `i` | Interactive shell |
| `m` | Job control enabled |
| `B` | Brace expansion enabled |
| `H` | History substitution (`!`) enabled |

---

## 12. Summary

- **`echo`** prints literal strings and variable values (`$VAR`) to the screen, accepts multiple arguments, and behaves differently depending on quoting: **double quotes expand variables**, **single quotes do not**, and `\$` escapes a literal dollar sign.
- **Common environment variables** — `$USER`/`$USERNAME`, `$HOME`, `$HOSTNAME`, `$PWD`, `$PATH` — describe the user and system configuration; they can be listed in bulk using `printenv`, `env`, or `set`.
- **`date`** shows the current system date/time, and `date -R` formats it per RFC 2822.
- Aliased commands can be bypassed to run the **original** version using `\command` or a **full path** like `/usr/bin/date`.
- **Special shell variables** describe the shell's own runtime state and script arguments:
  - `$0` → shell/script name
  - `$1`–`$9` → individual positional arguments
  - `$#` → count of arguments
  - `$@` → all arguments
  - `$$` → process ID of the shell
  - `$?` → exit code of the last command
  - `$-` → active shell flags
- **`ps`** and its options (`-e`, `-f`, `-ef`, `--forest`) reveal what processes are running, in short or full detail, and can display them as a parent-child tree.
- **Process control** tools (`&`, `fg`, `coproc`, `jobs`, `top`, `kill`) let you manage how programs run — in the foreground, background, or as communicating co-processes — and how they are monitored or terminated.
- **Exit codes** (`0`, `1`, `2`, `126`, `127`, `130`, `137`) stored in `$?` tell you precisely how and why the last command succeeded or failed, including special cases like Ctrl+C (`130`) and `kill -9` (`137`).
- **Bash flags** (`h`, `B`, `i`, `m`, `H`, `s`, `c`), visible via `$-`, describe which shell behaviors (interactivity, job control, history, brace expansion, etc.) are currently active.

> **Quick recall for exams:** *"Zero is success, everything else is a story."* `$?` tells the story of the last command, `$$` tells you who's telling it (the shell's PID), `$0` tells you the storyteller's name, `$1`–`$9`/`$#`/`$@` tell you what was handed to the story (arguments), and `$-` tells you the storyteller's mood (behavioral flags).

---

