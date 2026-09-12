# Week 3 — Lecture 1 & 2: Combining Commands and File Redirection

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Combining Multiple Commands](#2-combining-multiple-commands)
   - 2.1 [Sequential Execution — `;`](#21-sequential-execution--)
   - 2.2 [Conditional AND Execution — `&&`](#22-conditional-and-execution--)
   - 2.3 [Conditional OR Execution — `||`](#23-conditional-or-execution--)
   - 2.4 [Command Grouping with Subshells — `()`](#24-command-grouping-with-subshells--)
   - 2.5 [Summary Table: Command Combination Operators](#25-summary-table-command-combination-operators)
3. [File Descriptors](#3-file-descriptors)
   - 3.1 [The Three Standard File Descriptors](#31-the-three-standard-file-descriptors)
4. [Output Redirection](#4-output-redirection)
   - 4.1 [Redirecting stdout — `>`](#41-redirecting-stdout--)
   - 4.2 [Appending stdout — `>>`](#42-appending-stdout--)
   - 4.3 [Redirecting stderr — `2>`](#43-redirecting-stderr--2)
   - 4.4 [Redirecting stdout and stderr to Different Files — `> file1 2> file2`](#44-redirecting-stdout-and-stderr-to-different-files--file1-2-file2)
   - 4.5 [Redirecting stderr to stdout — `2>&1`](#45-redirecting-stderr-to-stdout--21)
5. [Input Redirection — `<`](#5-input-redirection--)
6. [Piping Commands — `|`](#6-piping-commands--)
   - 6.1 [Simple Pipe — `command1 | command2`](#61-simple-pipe--command1--command2)
   - 6.2 [Pipe with Output Redirection — `command1 | command2 > file1`](#62-pipe-with-output-redirection--command1--command2--file1)
7. [Discarding Output — `/dev/null`](#7-discarding-output--devnull)
8. [The `tee` Command](#8-the-tee-command)
   - 8.1 [Using `diff` to Verify `tee` Output](#81-using-diff-to-verify-tee-output)
   - 8.2 [Writing to Multiple Files with `tee`](#82-writing-to-multiple-files-with-tee)
   - 8.3 [Combining Redirection, `tee`, and Pipes](#83-combining-redirection-tee-and-pipes)
9. [Consolidated Reference Table — All Redirection Operators](#9-consolidated-reference-table--all-redirection-operators)
10. [Summary](#10-summary)

---

## 1. Introduction

In Linux/Unix, the shell allows multiple commands to be **combined** and allows the **input/output** of a command to be **redirected** to and from files, rather than the default keyboard and screen. This is one of the most powerful features of the shell, enabling automation, logging, and building complex data-processing pipelines from simple, single-purpose commands.

This lecture covers two closely related topics:

- **Combining commands** — controlling the order and conditions under which multiple commands run.
- **Redirection** — controlling where a command reads its input from and sends its output/errors to.

---

## 2. Combining Multiple Commands

Multiple commands can be written on a single line and executed together using special operators. The three operators covered are `;`, `&&`, and `||`.

### 2.1 Sequential Execution — `;`

**Concept:**
The semicolon operator runs each command **one after another**, regardless of whether the previous command succeeded or failed. It is simply a way to type multiple independent commands on one line.

**Syntax:**
```bash
command1; command2; command3
```

**Explanation:** Each command will be executed one after the other, unconditionally.

**Examples:**
```bash
# Example 1: Print date, then list files, then show current directory
date; ls; pwd

# Example 2: Create a directory and then move into it (each step runs regardless)
mkdir project; cd project; touch file.txt

# Example 3: Even if the first command fails, the second still executes
lss; echo "This will still print"
```

---

### 2.2 Conditional AND Execution — `&&`

**Concept:**
The `&&` operator runs **command2 only if command1 succeeds** (i.e., command1 returns an exit status of `0`). This is used when the second command logically depends on the success of the first.

**Syntax:**
```bash
command1 && command2
```

**Explanation:** command2 will be executed only if command1 succeeds.

**Examples:**
```bash
# Example 1: Compile a program, and only run it if compilation succeeds
gcc program.c -o program && ./program

# Example 2: Create a directory only proceeds to enter it if creation succeeded
mkdir backup && cd backup

# Example 3: Update package list, then upgrade only if update succeeded
sudo apt update && sudo apt upgrade
```

---

### 2.3 Conditional OR Execution — `||`

**Concept:**
The `||` operator runs **command2 only if command1 fails** (i.e., command1 returns a non-zero exit status). command2 acts as a fallback or error-handling step.

**Syntax:**
```bash
command1 || command2
```

**Explanation:** command2 will *not* be executed if command1 succeeds. It only executes when command1 fails.

**Examples:**
```bash
# Example 1: Try to remove a directory; if it fails, print an error message
rmdir myfolder || echo "Directory could not be removed"

# Example 2: Attempt to ping a server; if it fails, notify the user
ping -c 1 google.com || echo "Network is unreachable"

# Example 3: Try creating a file only if it doesn't already exist / fallback action
mkdir data || echo "Folder already exists"
```

---

### 2.4 Command Grouping with Subshells — `()`

**Concept:**
Enclosing one or more commands inside parentheses `( )` runs them in a **subshell** — a separate, temporary child shell process spawned by the current shell. Commands inside `()` execute in this child environment, and once they finish, control returns to the parent shell. This is useful for **grouping** commands so their combined output can be redirected together, or so that changes made inside (like `cd`, exported variables) don't affect the parent shell.

**Syntax:**
```bash
(command1; command2; command3)
```

**Explanation:**
- Everything inside the parentheses runs in a **new child shell**, not the current one.
- Any `cd`, variable assignment, or `export` done inside `()` is **local to the subshell** and disappears once the subshell exits — the parent shell's environment is untouched.
- The grouped output of all commands inside `()` can be redirected or piped as a single unit.

**Checking Subshell Depth — `$BASH_SUBSHELL`:**
Bash provides the special variable `$BASH_SUBSHELL`, which reports how many levels of subshell nesting the current shell is inside. It starts at `0` in the normal (top-level) shell, and increases by `1` for every additional subshell you enter.

**Syntax:**
```bash
echo $BASH_SUBSHELL
```

**Examples:**
```bash
# Example 1: Group commands so cd doesn't affect the parent shell
(cd /tmp && ls)
pwd   # still shows the original directory, NOT /tmp

# Example 2: Redirect the combined output of multiple grouped commands to one file
(date; whoami; pwd) > session_info.txt

# Example 3: Check subshell depth in the normal top-level shell
echo $BASH_SUBSHELL
# Output: 0

# Example 4: Check subshell depth from inside a subshell
(echo $BASH_SUBSHELL)
# Output: 1
```

**Subshells within Subshells (Nested Subshells):**
Subshells can be nested inside one another — each additional level of `()` creates another child shell, and `$BASH_SUBSHELL` increases with each level of nesting. This demonstrates how deeply the current command is nested away from the original login shell.

**Examples:**
```bash
# Example 1: Two levels of nested subshells
(echo "Level 1: $BASH_SUBSHELL"; (echo "Level 2: $BASH_SUBSHELL"))
# Output:
# Level 1: 1
# Level 2: 2

# Example 2: Three levels of nested subshells
(echo "L1: $BASH_SUBSHELL"; (echo "L2: $BASH_SUBSHELL"; (echo "L3: $BASH_SUBSHELL")))
# Output:
# L1: 1
# L2: 2
# L3: 3

# Example 3: Variables set inside a nested subshell don't leak out to the parent
x=10
(x=20; (x=30; echo "innermost x=$x")); echo "outer x=$x"
# Output:
# innermost x=30
# outer x=10
```

---

### 2.5 Summary Table: Command Combination Operators

| Operator | Name | Behaviour | Runs command2 when? |
|----------|------|-----------|----------------------|
| `;` | Sequential | Executes commands one after another | Always |
| `&&` | Logical AND | Executes command2 only if command1 succeeds | command1 exit status = 0 |
| `\|\|` | Logical OR | Executes command2 only if command1 fails | command1 exit status ≠ 0 |

---

## 3. File Descriptors

**Concept:**
A **file descriptor (FD)** is a small integer that the operating system uses to identify an open input/output channel for a process. Every command/process, by default, has **three standard file descriptors** automatically opened for it: one for input and two for output.

### 3.1 The Three Standard File Descriptors

| File Descriptor Number | Name | Abbreviation | Default Source/Destination |
|-------------------------|------|---------------|------------------------------|
| `0` | Standard Input | stdin | Keyboard |
| `1` | Standard Output | stdout | Screen/Monitor |
| `2` | Standard Error | stderr | Screen/Monitor |

**Explanation of the diagram concept:**
- **stdin (0):** The command reads its input from the keyboard by default.
- **stdout (1):** The command sends its normal (successful) output to the screen by default.
- **stderr (2):** The command sends its error messages to the screen by default.

By using redirection operators, any of these three channels can be pointed away from their default source/destination (keyboard/screen) toward a file, another command, or a special device.

---

## 4. Output Redirection

### 4.1 Redirecting stdout — `>`

**Concept:**
The `>` operator redirects a command's **standard output (stdout)** into a file instead of displaying it on the screen. stderr still goes to the screen.

**Syntax:**
```bash
command > file1
```

**⚠️ Warning:**
- If `file1` already exists, its **contents will be overwritten** (completely replaced).
- If `file1` does not exist, a **new file will be created**.

**Examples:**
```bash
# Example 1: Save directory listing to a file
ls -l > listing.txt

# Example 2: Save the output of a script to a log file (overwrites each run)
./myscript.sh > output.log

# Example 3: Save current date and time to a file
date > timestamp.txt

# Example 4: Save the output of a hardware information command to a file
hwinfo > hardware_report.txt
```

**Using `cat` with `>` to Create a File Interactively:**
When `cat` is run with **no input file argument**, it reads from **stdin** (the keyboard) instead. Combining this with `>` redirection lets you type text directly into a new file from the terminal — a quick way to create small files without opening a text editor.

**Syntax:**
```bash
cat > file1
```

**Explanation:**
1. `cat` waits for you to type lines of text at the keyboard (since no input file was given, stdin defaults to the keyboard).
2. Each line you type is sent to `cat`'s stdout, which has been redirected into `file1`.
3. Press **Ctrl+D** (End-Of-File) on a new line to stop input and close the file.

**Examples:**
```bash
# Example 1: Create a new file and type its content directly
cat > notes.txt
This is line one.
This is line two.
# (press Ctrl+D to save and exit)

# Example 2: Quickly create a short config file
cat > config.txt
debug=true
verbose=false
# (press Ctrl+D)

# Example 3: View the file you just created using plain cat (reads from a file, prints to screen)
cat notes.txt
```

---

### 4.2 Appending stdout — `>>`

**Concept:**
The `>>` operator redirects standard output to a file, but instead of overwriting, it **appends** the new output to the end of the existing file content.

**Syntax:**
```bash
command >> file1
```

**Note/Warning:**
- Contents will be **appended** to `file1` (existing content is preserved).
- A **new `file1` will be created** if it does not already exist.

**Examples:**
```bash
# Example 1: Keep adding new log entries without erasing old ones
echo "Server started" >> server.log

# Example 2: Append the current date to a running log file every time script runs
date >> activity.log

# Example 3: Append list of files to an existing inventory file
ls >> inventory.txt
```

**Combining `>>` with Sequential Execution (`;`):**
The append operator can be combined with the `;` operator (see Section 2.1) so that several unrelated commands each append their own output to the **same file**, one after another — building up a combined log or report step by step.

**Syntax:**
```bash
command1 >> file; command2 >> file; command3 >> file
```

**Examples:**
```bash
# Example 1: Build a system report by appending three separate commands to one file
date >> report.txt; whoami >> report.txt; pwd >> report.txt

# Example 2: Log the start, middle, and end of a maintenance task
echo "Task started" >> maintenance.log; df -h >> maintenance.log; echo "Task finished" >> maintenance.log

# Example 3: Append hostname, uptime, and disk usage to a single status file
hostname >> status.txt; uptime >> status.txt; du -sh /home >> status.txt
```

**Using `cat` with `>>` to Append Text Interactively:**
Just like `cat > file` creates a new file from keyboard input, `cat >> file` **appends** typed keyboard input to the end of an existing file instead of overwriting it.

**Syntax:**
```bash
cat >> file1
```

**Examples:**
```bash
# Example 1: Append a new note to an existing file
cat >> notes.txt
This is an additional line added later.
# (press Ctrl+D to save and exit)

# Example 2: Append a new entry to a running diary/log file
cat >> diary.txt
Day 5: Completed the Linux redirection lecture notes.
# (press Ctrl+D)

# Example 3: Verify the appended content by viewing the file
cat notes.txt
```

---

### 4.3 Redirecting stderr — `2>`

**Concept:**
The `2>` operator redirects only the **standard error (stderr)** stream to a file. Normal output (stdout) continues to go to the screen.

**Syntax:**
```bash
command 2> file1
```

**⚠️ Warning:**
- Contents of `file1` will be **overwritten**.
- A **new `file1` will be created** if it does not exist.

**Examples:**
```bash
# Example 1: Capture only error messages while listing a non-existent directory
ls /nonexistent 2> errors.txt

# Example 2: Run a script and store only its errors, while still seeing normal output on screen
./deploy.sh 2> deploy_errors.log

# Example 3: Redirect errors while compiling code
gcc buggy.c -o buggy 2> compile_errors.txt
```

---

### 4.4 Redirecting stdout and stderr to Different Files — `> file1 2> file2`

**Concept:**
stdout and stderr can be sent to **two separate files simultaneously** — normal output goes to one file, and error messages go to a different file.

**Syntax:**
```bash
command > file1 2> file2
```

**⚠️ Warning:** Contents of **both `file1` and `file2`** will be overwritten.

**Examples:**
```bash
# Example 1: Separate normal output and errors when listing files
ls -l /home /nonexistent > output.txt 2> errors.txt

# Example 2: Separate build output from build errors
make > build_output.log 2> build_errors.log

# Example 3: Run backup script, tracking success messages and failures separately
./backup.sh > success.log 2> failure.log
```

---

### 4.5 Redirecting stderr to stdout — `2>&1`

**Concept:**
The `2>&1` syntax redirects stderr **into wherever stdout is currently pointing**. When combined with `>`, both stdout and stderr end up in the **same file**. The order matters: `> file1 2>&1` must have `> file1` **first**, so that stderr is then pointed to the same destination as stdout.

**Syntax:**
```bash
command > file1 2>&1
```

**⚠️ Warning:** Contents of `file1` will be overwritten.

**Examples:**
```bash
# Example 1: Capture both normal output and errors into a single log file
./run.sh > full_log.txt 2>&1

# Example 2: Combine output and errors of a find command into one file
find / -name "*.conf" > search_results.txt 2>&1

# Example 3: Redirect all output (both streams) of a cron job to one file
0 2 * * * /home/user/script.sh > /home/user/cron.log 2>&1
```

---

## 5. Input Redirection — `<`

**Concept:**
The `<` operator redirects a command's **standard input (stdin)** so that it reads from a **file** instead of the keyboard.

**Syntax:**
```bash
command < file1
```

**Explanation:** Instead of the command waiting for keyboard input, it reads its input directly from the contents of `file1`.

**Examples:**
```bash
# Example 1: Sort the contents of a file
sort < names.txt

# Example 2: Count the number of lines in a file via stdin
wc -l < data.csv

# Example 3: Feed a file as input to a mail command
mail -s "Report" user@example.com < report.txt
```

---

## 6. Piping Commands — `|`

### 6.1 Simple Pipe — `command1 | command2`

**Concept:**
A **pipe** connects the **stdout of command1 directly to the stdin of command2**. This allows the output of one command to become the input of the next, without needing an intermediate file. Note: stderr of command1 is **not** piped — it still goes to the screen by default.

**Syntax:**
```bash
command1 | command2
```

**Examples:**
```bash
# Example 1: List files and filter for a keyword
ls -l | grep ".txt"

# Example 2: Count number of running processes
ps aux | wc -l

# Example 3: View a large file page by page
cat largefile.txt | less
```

---

### 6.2 Pipe with Output Redirection — `command1 | command2 > file1`

**Concept:**
A pipe can be combined with output redirection: command1's stdout feeds command2's stdin, and command2's stdout is then written to a file.

**Syntax:**
```bash
command1 | command2 > file1
```

**⚠️ Warning:** Contents of `file1` will be overwritten.

**Examples:**
```bash
# Example 1: Filter log entries and save matches to a file
cat system.log | grep "ERROR" > errors_found.txt

# Example 2: Sort a list of names and save the sorted result
cat names.txt | sort > sorted_names.txt

# Example 3: Get a unique, sorted list of users and save it
cut -d: -f1 /etc/passwd | sort | uniq > users.txt
```

---

## 7. Discarding Output — `/dev/null`

**Concept:**
`/dev/null` is a special system file known as the **"null device"** or **"bit bucket."** It acts as a **sink** — anything written to it is immediately and permanently discarded. It is commonly used to silence unwanted output (e.g., errors) so it does not clutter the screen or a log file.

**Use case:** Writing silent and clean scripts, where certain output (usually errors) should be suppressed entirely.

**Syntax:**
```bash
command > file1 2> /dev/null
```

**⚠️ Warning:** Contents of `file1` will be overwritten. (stderr is simply discarded, not saved anywhere.)

**Examples:**
```bash
# Example 1: Save normal output but silently discard all errors
find / -name "*.log" > results.txt 2> /dev/null

# Example 2: Discard both stdout and stderr completely (fully silent command)
./script.sh > /dev/null 2>&1

# Example 3: Ping a host, showing nothing on screen at all — useful in scripts
ping -c 1 google.com > /dev/null 2>&1 && echo "Online" || echo "Offline"
```

---

## 8. The `tee` Command

**Concept:**
`tee` reads from stdin and **writes to both stdout (the screen) and a file simultaneously**. It is used when you want to **see the output on screen in real time** while also **saving a copy to a file** — unlike `>`, which only writes to the file and shows nothing on screen.

**Syntax:**
```bash
command1 | tee file1
```

**⚠️ Warning:** Contents of `file1` will be overwritten (unless the `-a` append flag is used).

**Examples:**
```bash
# Example 1: View directory listing on screen AND save it to a file
ls -l | tee listing.txt

# Example 2: Monitor a long-running process while logging its output
./install.sh | tee install_log.txt

# Example 3: Append instead of overwrite, using the -a flag
echo "New entry" | tee -a notes.txt
```

---

### 8.1 Using `diff` to Verify `tee` Output

**Concept:**
Since `tee` writes an identical copy of its input to a file **and** to the screen (or the next command in a pipe), the `diff` command is useful for **verifying** that the file `tee` created truly matches the original data — comparing two files line by line and reporting any differences.

**Syntax:**
```bash
diff file1 file2
```

**Explanation:** If the two files are identical, `diff` produces **no output** and exits silently. If they differ, `diff` prints the specific lines that differ between the two files.

**Examples:**
```bash
# Example 1: Confirm that a file created by tee matches the command's actual output
ls -l | tee listing.txt
ls -l > listing_direct.txt
diff listing.txt listing_direct.txt
# No output = files are identical

# Example 2: Compare two files saved by tee to two different destinations
diff file1.txt file2.txt

# Example 3: Show a summary instead of full details when files differ
diff -q report_a.txt report_b.txt
```

---

### 8.2 Writing to Multiple Files with `tee`

**Concept:**
`tee` can accept **more than one filename**, writing an identical copy of its input to **every file listed**, all while still passing the data through to stdout (or onward to the next command in a pipeline).

**Syntax:**
```bash
command1 | tee file1 file2 | command2
```

**Explanation:**
1. `command1`'s output is piped into `tee`.
2. `tee` writes an identical copy of that data into **both `file1` and `file2`**.
3. `tee` also passes the same data through its own stdout, which is piped into `command2` for further processing.

**Examples:**
```bash
# Example 1: Save a directory listing to two separate backup files while also counting the lines
ls -l | tee file1.txt file2.txt | wc -l

# Example 2: Save filtered log output to two files while continuing to sort it
cat system.log | grep "ERROR" | tee errors_copy1.txt errors_copy2.txt | sort

# Example 3: Save raw output to two audit files while piping onward to search for a keyword
dmesg | tee audit1.log audit2.log | grep -i "usb"
```

---

### 8.3 Combining Redirection, `tee`, and Pipes

**Concept:**
Redirection, `tee`, and pipes can all be combined in a single command line for advanced workflows — for example, discarding a command's error messages while simultaneously saving its normal output to multiple files and continuing to process that output further down a pipeline.

**Syntax:**
```bash
command1 2> /dev/null | tee file1 file2 | command2
```

**Explanation:**
1. `command1 2> /dev/null` → runs `command1`, silently **discarding its stderr** (error messages) so they don't clutter the screen.
2. `| tee file1 file2` → the surviving **stdout** of `command1` is duplicated into both `file1` and `file2`.
3. `| command2` → the same stdout data is simultaneously passed onward into `command2` for further processing.

**Examples:**
```bash
# Example 1: Search the filesystem, silence "permission denied" errors, save results to two files, and count them
find / -name "*.conf" 2> /dev/null | tee found1.txt found2.txt | wc -l

# Example 2: Discard errors from a build, log the output to two files, and filter for warnings
make 2> /dev/null | tee build1.log build2.log | grep -i "warning"

# Example 3: Silently ping a host, save results to two logs, and check for packet loss
ping -c 5 google.com 2> /dev/null | tee ping1.log ping2.log | grep "loss"
```

---

## 9. Consolidated Reference Table — All Redirection Operators

| Syntax | Purpose | Overwrite / Append | Screen Output? |
|--------|---------|----------------------|------------------|
| `command1; command2` | Run commands sequentially | — | Both |
| `command1 && command2` | Run command2 only if command1 succeeds | — | Both (conditional) |
| `command1 \|\| command2` | Run command2 only if command1 fails | — | Both (conditional) |
| `(command1; command2)` | Run grouped commands in a subshell | — | Both (as a unit) |
| `command > file1` | Redirect stdout to file | Overwrite | stderr still shown |
| `cat > file1` | Create a file interactively from keyboard input | Overwrite | None (writes to file) |
| `command >> file1` | Redirect stdout to file | Append | stderr still shown |
| `command1 >> file; command2 >> file` | Multiple commands append to the same file in sequence | Append | stderr still shown |
| `cat >> file1` | Append keyboard input interactively to a file | Append | None (writes to file) |
| `command 2> file1` | Redirect stderr to file | Overwrite | stdout still shown |
| `command > file1 2> file2` | Redirect stdout and stderr to two different files | Overwrite (both) | None |
| `command > file1 2>&1` | Redirect both stdout and stderr to the same file | Overwrite | None |
| `command < file1` | Redirect stdin to read from file | — | N/A |
| `command1 \| command2` | Pipe stdout of command1 into stdin of command2 | — | Only command2's output |
| `command1 \| command2 > file1` | Pipe, then redirect final output to file | Overwrite | None |
| `command > file1 2> /dev/null` | Save stdout, discard stderr | Overwrite | None |
| `command1 \| tee file1` | Save output to file **and** show it on screen | Overwrite | Yes (both) |
| `diff file1 file2` | Compare two files line by line | — | Differences only |
| `command1 \| tee file1 file2 \| command2` | Duplicate output into two files, then continue piping | Overwrite | Only command2's output |
| `command1 2> /dev/null \| tee file1 file2 \| command2` | Discard errors, tee stdout to two files, continue piping | Overwrite | Only command2's output |

---

## 10. Summary

- **Combining commands** on one line is done using three operators:
  - `;` → runs commands unconditionally, one after another.
  - `&&` → runs the next command **only on success** of the previous one.
  - `||` → runs the next command **only on failure** of the previous one.
- **Subshells (`()`)** group commands to run in a separate child shell process — useful for isolating `cd`/variable changes from the parent shell, or redirecting the combined output of several commands as one unit. `$BASH_SUBSHELL` reports the current nesting depth (`0` in the top-level shell, incrementing with each nested `()`), and subshells can be nested inside one another indefinitely.
- Every command has **three standard file descriptors**: `0` (stdin, keyboard), `1` (stdout, screen), `2` (stderr, screen).
- **Redirection operators** change where these streams read from / write to:
  - `>` overwrites a file with stdout; `>>` appends to it. Both can also be combined with `;` to have several commands write to the same file in sequence.
  - `cat > file` / `cat >> file` use `cat`'s default stdin behaviour (reading from the keyboard when no input file is given) to create or append to a file interactively, ending with **Ctrl+D**.
  - `2>` redirects only error messages; it can go to its own file, or be combined with `>` to send stdout and stderr to two separate files.
  - `2>&1` merges stderr into the same destination as stdout — must be written **after** the `>` redirection for stdout.
  - `<` redirects stdin so a command reads from a file instead of the keyboard.
- **Pipes (`|`)** connect the stdout of one command directly to the stdin of the next, enabling multi-step processing chains; they can also be combined with file redirection at the end of the chain.
- **`/dev/null`** is a special discard destination used to silently suppress unwanted output (commonly errors) in scripts, and is frequently combined with pipes (e.g. `command 2> /dev/null | tee file1 file2 | command2`) to build clean, multi-stage pipelines.
- **`tee`** is unique in that it duplicates output — sending it to the screen (or the next command in a pipe) **and** one or more files at the same time, unlike plain redirection which only sends output to one destination. `tee` can write to **multiple files simultaneously** (`tee file1 file2`) while still passing data onward in a pipeline, and `diff` can be used afterward to confirm that the saved copies exactly match the original output.
- Mastering these operators is essential for **automation, scripting, logging, debugging, and building efficient command pipelines** in Linux/Unix systems.

---

*End of Notes — Week 3, Lecture 1 & 2: Combining Commands and Redirection*
