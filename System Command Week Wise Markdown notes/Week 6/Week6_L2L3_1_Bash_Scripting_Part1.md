# Bash Shell Scripting — Part 1
### Week 6 | Lecture 2 & 3
### Topic: Creating Your Own Commands (Shell Scripts)

---

## Table of Contents

1. [Software Tools Principles](#1-software-tools-principles)
2. [Introduction to Shell Scripts](#2-introduction-to-shell-scripts)
    - 2.1 [Creating Your First Script (Hands-on, Step by Step)](#21-creating-your-first-script-hands-on-step-by-step)
3. [Sourced vs Executed Scripts](#3-sourced-vs-executed-scripts)
    - 3.1 [Hands-on Walkthrough: Proving the PID and Environment Difference](#31-hands-on-walkthrough-proving-the-pid-and-environment-difference)
    - 3.2 [Summary Table of the Hands-on Demonstration](#32-summary-table-of-the-hands-on-demonstration)
4. [Script Location and Execution](#4-script-location-and-execution)
5. [Bash Environment — Login vs Non-Login Shell](#5-bash-environment--login-vs-non-login-shell)
6. [Output From Shell Scripts](#6-output-from-shell-scripts)
7. [Input to Shell Scripts](#7-input-to-shell-scripts)
8. [Shell Script Arguments](#8-shell-script-arguments)
    - 8.1 [Detecting How a Script Was Invoked, Using `$0`](#81-detecting-how-a-script-was-invoked-using-0)
    - 8.2 [More Examples on Arguments](#82-more-examples-on-arguments)
9. [Command Substitution](#9-command-substitution)
10. [Loops in Shell Scripting](#10-loops-in-shell-scripting)
    - 10.1 [for Loop](#101-for-loop)
      - [Using `grep` Inside a `for` Loop (Practical Example)](#using-grep-inside-a-for-loop-practical-example)
    - 10.2 [while Loop](#102-while-loop)
    - 10.3 [until Loop](#103-until-loop)
11. [Conditional Statements](#11-conditional-statements)
    - 11.1 [if Statement](#111-if-statement)
    - 11.2 [case Statement](#112-case-statement)
12. [Conditions and Test Expressions](#12-conditions-and-test-expressions)
    - 12.1 [Types of Conditions](#121-types-of-conditions)
    - 12.2 [Numeric Comparisons](#122-numeric-comparisons)
    - 12.3 [String Comparisons](#123-string-comparisons)
    - 12.4 [Unary File Comparisons](#124-unary-file-comparisons)
    - 12.5 [Binary File Comparisons](#125-binary-file-comparisons)
13. [Functions in Shell Scripts](#13-functions-in-shell-scripts)
14. [Summary](#14-summary)

---

## 1. Software Tools Principles

Before writing shell scripts, it is important to understand the philosophy behind the **UNIX/Linux Software Tools** design. Every good shell script or command-line tool should ideally follow these principles.

### Concept

| Principle | Explanation |
|---|---|
| Do one thing well | Each tool/script should focus on a single, well-defined task rather than trying to do everything. |
| Process lines of text, not binary | Tools should operate on plain text streams so that output of one tool can become the input of another. |
| Use regular expressions | Pattern matching should follow a standard, common syntax (regex) understood across tools. |
| Default to standard I/O | Tools should read from standard input and write to standard output by default, unless a file is specified. |
| Don't be chatty | Avoid unnecessary messages/prompts; keep output clean and predictable (important for pipelines). |
| Generate the same output format as accepted input | This allows tools to be chained together (output of one becomes input of the next). |
| Let someone else do the hard part | Reuse existing tools/utilities instead of re-inventing functionality. |
| Detour to build specialized tools | If no existing tool fits the need, take the time to build a new, focused one. |

> **Reference:** *Classic Shell Scripting* — Arnold Robbins & Nelson H.F. Beebe

### Why This Matters for Exams
These principles explain **why** shell scripting favors small, composable, text-based commands connected using pipes (`|`) instead of large monolithic programs. This is a common **conceptual/theory question**.

---

## 2. Introduction to Shell Scripts

A **shell script** is a text file containing a sequence of commands that the shell can execute, just like a program.

### Structure of a Shell Script

```bash
#!/bin/bash          # interpreter line (shebang)
# comments           # lines starting with # are comments
commands              # normal shell commands
loops                 # for, while, until
variables              # variable assignment and usage
case statements         # multi-branch conditional
functions               # reusable blocks of code
```

### The Shebang (`#!`) Line

- The first line of a script, e.g. `#!/bin/bash`, tells the operating system **which interpreter** should run the script.
- Common interpreters used in scripting:

| Interpreter | Shebang Example | Typical Use |
|---|---|---|
| Bash (shell) | `#!/bin/bash` | General-purpose shell scripting |
| awk | `#!/usr/bin/awk -f` | Text/column processing |
| sed | `#!/bin/sed -f` | Stream editing |
| Python | `#!/usr/bin/env python3` | General scripting/programming |
| Ruby | `#!/usr/bin/env ruby` | General scripting/programming |
| Perl | `#!/usr/bin/perl` | Text processing/scripting |

### Example

```bash
#!/bin/bash
# This is a simple greeting script
echo "Hello, welcome to Bash Scripting!"
```

### 2.1 Creating Your First Script (Hands-on, Step by Step)

For a beginner, the easiest way to create a script is by using a text editor such as **`vi`** (or `nano`, `gedit`, etc.) directly from the terminal.

**Step-by-step:**

```bash
vi s1.sh
```
This opens the `vi` editor and creates a new (empty) file named `s1.sh` if it doesn't already exist.

Inside `vi`:

1. Press **`i`** to enter **Insert mode** (so you can start typing).
2. Type your script content, for example:
   ```bash
   #!/bin/bash
   echo "This is my first script"
   echo "My PID is: $$"
   ```
3. Press **`Esc`** to exit Insert mode.
4. Type **`:wq`** and press **Enter** to **save and quit** (`w` = write/save, `q` = quit).

> **Beginner Tip:** If you make a mistake and want to quit **without saving**, type `:q!` instead of `:wq`.

At this point, `s1.sh` exists as a plain text file, but it is **not yet executable** and has not been run — it is just a script waiting to be either **sourced** or **executed** (explained next in Section 3).

---

## 3. Sourced vs Executed Scripts

A script can be run in **two different ways** in Bash, and this is a very important **exam concept** because the behavior is completely different.

### Concept and Syntax

| Feature | Sourced Script | Executed Script |
|---|---|---|
| Syntax | `. scriptname` or `source scriptname` | `./scriptname` |
| Permission needed | No execute permission required | Needs execute permission (`chmod +x`) |
| Process | No new process is created; runs in the **current shell** | A **new (child) process** is created to run the script |
| PID | Same as the current shell's PID | Different from the shell's PID |
| Command execution | Commands executed one after another | Commands executed one after another |
| Effect on environment | Shell environment **continues/persists** after script finishes | New environment is **lost** after the script returns |
| Typical use | Used to **prepare/set up environment** (e.g., set variables, aliases) | Used to **create new functionality** (run a standalone task) |

### Example — Sourcing a Script

```bash
# myenv.sh
export PROJECT_HOME=/home/user/project
alias ll='ls -la'
```

Run it using:
```bash
source myenv.sh
# or
. myenv.sh
```
After sourcing, `$PROJECT_HOME` and the `ll` alias remain available in your **current terminal session**.

### Example — Executing a Script

```bash
# hello.sh
#!/bin/bash
echo "Hello World"
```

Run it using:
```bash
chmod +x hello.sh   # give execute permission
./hello.sh          # run in a new process
```
Any variables set inside `hello.sh` will **not** be available in the parent shell after it finishes.

> **Exam Tip:** If asked "why does `export` inside a script not affect my terminal?" — the answer is because the script was **executed** (new process), not **sourced**.

### 3.1 Hands-on Walkthrough: Proving the PID and Environment Difference

This is the **most important practical demonstration** for this topic — it proves *why* sourcing and executing behave differently, using real commands you can try yourself. Follow along step by step.

#### Step 1 — Check the PID of your current (parent) shell

```bash
echo $$
```
- `$$` is a special Bash variable that always holds the **PID (Process ID) of the current shell**.
- Example output: `2431` (this is the terminal's own shell process ID).

#### Step 2 — Create the script `s1.sh` with a PID check inside it

```bash
vi s1.sh
```

Content of `s1.sh`:

```bash
#!/bin/bash
echo "PID inside script: $$"
```

Save and quit (`Esc` → `:wq`).

#### Step 3 — Run it using `source` (or `.`)

```bash
. s1.sh
# or
source s1.sh
```

**Output example:**
```
PID inside script: 2431
```

→ Notice the PID is **exactly the same** as `echo $$` from Step 1. This proves that **sourcing runs the script inside the current shell** — no new process was created.

#### Step 4 — Make the script executable and run it using its path

```bash
chmod +x s1.sh
./s1.sh
```

**Output example:**
```
PID inside script: 2587
```

→ This time the PID is **different**. This proves that `./s1.sh` creates a **brand-new child process** to run the script, separate from your terminal's shell.

#### Step 5 — Visualize the process tree using `ps --forest`

Add this line inside `s1.sh` (temporarily), or run it right after launching the script, to see the **parent-child relationship** of processes:

```bash
ps --forest
```

**Sample Output (when executed with `./s1.sh`):**
```
  PID TTY          TIME CMD
 2431 pts/0    00:00:00 bash
 2587 pts/0    00:00:00  \_ s1.sh
```

- The `\_` symbol visually shows that `s1.sh` (PID 2587) is a **child process** branching off from the parent shell `bash` (PID 2431).
- If you run the **same check while sourcing** the script, you will **not** see a separate branch for `s1.sh` — because no child process was created; everything happened inside PID 2431 itself.

> **Beginner Tip:** `ps --forest` is a great debugging tool to *visually* understand parent-child process relationships — very useful whenever you're confused about "is this running in a new process or not?"

#### Step 6 — Check variable availability in the parent shell (source vs execute)

This confirms the **environment persistence** difference practically.

**Test A — Using `source`:**

```bash
# s1.sh
#!/bin/bash
myvar="Hello from script"
```

```bash
source s1.sh
echo $myvar
```

**Output:**
```
Hello from script
```

→ The variable `myvar` **is available** in the parent shell after sourcing, because sourcing runs the script's commands **inside** the current shell — so any variable created becomes part of the current shell's environment.

**Test B — Using `./s1.sh` (execute):**

```bash
./s1.sh
echo $myvar
```

**Output:** *(blank — nothing is printed)*

→ The variable `myvar` **is NOT available** in the parent shell, because `./s1.sh` ran in a **separate child process**. Once that child process finished, its entire environment (including `myvar`) was **destroyed**, and the parent shell never had access to it in the first place.

### 3.2 Summary Table of the Hands-on Demonstration

| Test Performed | Sourced (`. s1.sh` / `source s1.sh`) | Executed (`./s1.sh`) |
|---|---|---|
| `echo $$` inside script vs outside | **Same PID** as parent shell | **Different PID** (new child process) |
| `ps --forest` | No separate branch/child process shown | Shows `s1.sh` as a child branching from `bash` |
| Variable created inside script (e.g. `myvar`) | **Available** after script finishes (`echo $myvar` prints value) | **Not available** (`echo $myvar` prints nothing — variable lost with child process) |

> **Exam Tip:** If an exam question shows a script creating a variable and asks "will `echo $variable` work after running the script?" — always check **how** the script was run. `source`/`.` → yes, it will work. `./script` → no, it will not work (unless the variable was explicitly exported *and* the script was sourced, or unless you use command substitution to extract a value).

---

## 4. Script Location and Execution

### Concept

To run a script, the shell must know **where to find it**. There are three common approaches:

| Method | Description | Example |
|---|---|---|
| Absolute path | Full path from root (`/`) directory | `/home/user/scripts/backup.sh` |
| Relative path | Path relative to the current directory | `./backup.sh` or `scripts/backup.sh` |
| Keep script in `$PATH` | Place the script in a directory already listed in `$PATH` so it can be run by name only | `backup.sh` (no `./` needed) |

### Important Notes
- If a script is placed in a folder listed in `$PATH`, it can be executed just by typing its name — no need for `./` or full path.
- **Watch out for the sequence of directories in `$PATH`** — if two scripts/commands share the same name, the shell executes the one found **first**, based on the order of directories in `$PATH`.

### Example

```bash
echo $PATH
# Output: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/home/user/scripts

# If backup.sh is inside /home/user/scripts, and that directory is in $PATH:
backup.sh        # can run directly without ./ prefix
```

---

## 5. Bash Environment — Login vs Non-Login Shell

Bash reads different **startup/configuration files** depending on whether it is invoked as a **login shell** or a **non-login shell**. This determines which environment settings get loaded.

### Concept and Syntax

| Shell Type | Files Read (in typical order) |
|---|---|
| Login shell | `/etc/profile` → `~/.bash_profile` → `~/.bash_login` → `~/.profile` |
| Non-login shell | `/etc/bash.bashrc` → `~/.bashrc` |

### Explanation

- **Login shell**: Started when a user logs into the system (e.g., via console login, SSH session, or `su -`). It reads system-wide settings first (`/etc/profile`), then looks for **one** of the user-specific files: `~/.bash_profile`, `~/.bash_login`, or `~/.profile` (in that priority order).
- **Non-login shell**: Started when you open a new terminal window/tab within an already logged-in session (e.g., a new terminal in a GUI desktop). It reads `/etc/bash.bashrc` and then the user's `~/.bashrc`.

### Example

```bash
# Check whether your shell is a login shell:
shopt -q login_shell && echo "Login shell" || echo "Not a login shell"

# Typical content of ~/.bashrc (non-login shell)
export PS1="\u@\h:\w\$ "
alias grep='grep --color=auto'
```

> **Exam Tip:** A common question is "Why do environment variables set in `.bash_profile` not show up in a new terminal tab?" — because new terminal tabs are usually **non-login shells**, which read `.bashrc`, not `.bash_profile`.

---

## 6. Output From Shell Scripts

Bash provides two primary commands to generate output: `echo` and `printf`.

### 6.1 `echo`

**Concept:** Simple command to print text/strings to standard output.

**Syntax:**
```bash
echo [options] [string]
```

- By default, `echo` **terminates with a newline** unless the `-n` option is given (which suppresses the trailing newline).

**Example:**
```bash
echo My home is $HOME
# Output: My home is /home/user

echo -n "No newline at the end"
```

### 6.2 `printf`

**Concept:** Provides formatted output, similar to `printf` in the C programming language. Useful when precise formatting (strings, numbers, padding) is required.

**Syntax:**
```bash
printf "format string" arguments
```

**Example:**
```bash
printf "My home is %s\n" $HOME
# Output: My home is /home/user

printf "Name: %s, Age: %d\n" "Alice" 25
# Output: Name: Alice, Age: 25
```

### `echo` vs `printf` — Quick Comparison

| Feature | `echo` | `printf` |
|---|---|---|
| Newline at end | Automatic (unless `-n`) | Must be added manually (`\n`) |
| Format specifiers | Not supported | Supported (`%s`, `%d`, `%f`, etc.) |
| Simplicity | Very simple | Slightly more complex but more control |

---

## 7. Input to Shell Scripts

### Concept

The `read` command is used to accept input from the user (or from standard input) into a shell script.

### Syntax

```bash
read var
```
- The string typed at the command line is stored in the variable `$var`.

### Example

```bash
#!/bin/bash
echo "Enter your name:"
read name
echo "Hello, $name! Welcome to Bash scripting."
```

**Sample Run:**
```
Enter your name:
Priyanka
Hello, Priyanka! Welcome to Bash scripting.
```

### Additional Notes
- Multiple variables can be read at once: `read var1 var2`
- `read -p "Prompt: " var` displays a prompt and reads input in a single line.

---

## 8. Shell Script Arguments

When a script is executed with parameters, Bash provides special variables to access them.

### Concept and Syntax

| Symbol | Meaning |
|---|---|
| `$0` | Name of the shell program (the script itself) |
| `$#` | Number of arguments passed to the script |
| `$1` or `${1}` | First argument |
| `${11}` | Eleventh argument (curly braces required for arguments ≥ 10) |
| `$*` or `$@` | All arguments at once |
| `"$*"` | All arguments as a **single string** |
| `"$@"` | All arguments as **separate strings** (each preserving its own quoting) |

### Example

```bash
#!/bin/bash
echo "Script name: $0"
echo "Number of arguments: $#"
echo "First argument: $1"
echo "All arguments: $*"
```

**Run:**
```bash
./myscript.sh -l arg2 -v arg4
```

**Output:**
```
Script name: ./myscript.sh
Number of arguments: 4
First argument: -l
All arguments: -l arg2 -v arg4
```

### `"$*"` vs `"$@"` — Important Distinction

| Expression | Behavior | Example (args: `"a b"` `"c"`) |
|---|---|---|
| `"$*"` | Combines all arguments into **one single string** | `"a b c"` (1 item) |
| `"$@"` | Keeps each argument **separate** | `"a b"` `"c"` (2 items) |

> **Exam Tip:** `"$@"` is almost always preferred inside scripts when looping over arguments individually, because it preserves argument boundaries (especially with spaces).

### 8.1 Detecting How a Script Was Invoked, Using `$0`

**Concept:** Since `$0` always holds the **name/path used to call the script**, it can be used inside the script to detect **how it was run** — this connects directly back to the sourced-vs-executed concept from Section 3.

**Example:**
```bash
#!/bin/bash
echo "This script was called as: $0"
```

**Run it two different ways and compare:**

```bash
./s1.sh
# Output: This script was called as: ./s1.sh

source s1.sh
# Output: This script was called as: bash
# (or sometimes: -bash, depending on your system)
```

→ When a script is **executed** (`./s1.sh`), `$0` holds the **script's own name/path**.
→ When a script is **sourced**, `$0` holds the name of the **parent shell** (e.g., `bash`) — **not** the script's name — because the script's commands are running *inside* the existing shell process, which never changed its `$0`.

> **Beginner Tip:** This is a simple, reliable trick to make a script "self-aware" of how it was invoked — useful for writing setup/config scripts that should only ever be **sourced**, never executed directly (they can print a warning if `$0` doesn't look like the parent shell).

### 8.2 More Examples on Arguments

**Example — Using individual positional arguments:**
```bash
#!/bin/bash
echo "First argument:  $1"
echo "Second argument: $2"
echo "Third argument:  $3"
```
**Run:**
```bash
./args.sh red green blue
```
**Output:**
```
First argument:  red
Second argument: green
Third argument:  blue
```

**Example — Looping through all arguments using `"$@"`:**
```bash
#!/bin/bash
for arg in "$@"
do
    echo "Argument: $arg"
done
```
**Run:**
```bash
./args.sh "New Delhi" Mumbai "Los Angeles"
```
**Output:**
```
Argument: New Delhi
Argument: Mumbai
Argument: Los Angeles
```
→ Notice how `"New Delhi"` and `"Los Angeles"` are each treated as **one single argument** (because they were quoted), and `"$@"` correctly preserves them as separate items in the loop.

**Example — Checking if any arguments were passed at all (using `$#`):**
```bash
#!/bin/bash
if [ $# -eq 0 ]; then
    echo "No arguments were passed!"
else
    echo "You passed $# argument(s)."
fi
```

---

## 9. Command Substitution

### Concept

Command substitution allows the **output of a command** to be captured and assigned to a variable (or used inline).

### Syntax

```bash
var=`command`
var=$(command)
```
- The command is executed, and its output is substituted into the variable `var`.
- The `$( )` syntax is the modern, preferred form (easier to nest and read); backticks `` ` ` `` are the older/legacy form.

### Example

```bash
current_date=$(date)
echo "Today's date is: $current_date"

file_count=`ls | wc -l`
echo "Number of files in this directory: $file_count"
```

### Nested Example (only possible cleanly with `$()`)

```bash
echo "Files modified today: $(find . -mtime 0 | wc -l)"
```

---

## 10. Loops in Shell Scripting

Loops allow repeated execution of a block of commands. Bash supports three main loop types: `for`, `while`, and `until`.

### 10.1 `for` Loop

**Concept:** Iterates over a list of items, executing the loop body once for each item.

**Syntax:**
```bash
for var in list
do
    commands
done
```
- Commands are executed **once for each item** in the list.
- **Space** is the default field delimiter (separator) between list items.
- The `IFS` (Internal Field Separator) variable can be **set/changed if a different delimiter is required**.

**Example:**
```bash
#!/bin/bash
for fruit in apple banana mango
do
    echo "I like $fruit"
done
```

**Output:**
```
I like apple
I like banana
I like mango
```

**Example with changed IFS (comma-separated list):**
```bash
IFS=","
for item in apple,banana,mango
do
    echo "Item: $item"
done
```

**Example — Looping over a range of numbers:**
```bash
#!/bin/bash
for num in {1..5}
do
    echo "Number: $num"
done
```
**Output:**
```
Number: 1
Number: 2
Number: 3
Number: 4
Number: 5
```

**Example — Looping over files in a directory:**
```bash
#!/bin/bash
for file in *.txt
do
    echo "Found text file: $file"
done
```
→ This loops over every file in the current directory that matches the pattern `*.txt` (this pattern-based matching is called **globbing**).

#### Using `grep` Inside a `for` Loop (Practical Example)

**Concept:** A very common real-world use of the `for` loop is to **search multiple files (or lines) for a pattern** using `grep`, and process the matches one by one. This combines the "**do one thing well**" software tools principle (Section 1) with looping.

**Example — Searching for a keyword across multiple files:**
```bash
#!/bin/bash
for file in *.txt
do
    echo "Searching in: $file"
    grep "error" "$file"
done
```
→ This loops through every `.txt` file in the current directory and searches each one for the word `"error"` using `grep`.

**Example — Looping over the output of `grep` itself (line by line):**
```bash
#!/bin/bash
for line in $(grep "root" /etc/passwd)
do
    echo "Match found: $line"
done
```
→ Here, `$(grep "root" /etc/passwd)` is a **command substitution** (Section 9) that runs first — it searches `/etc/passwd` for lines containing `"root"`. The `for` loop then iterates over the **words** in that output (remember: space is the default field separator for `for`, so this works best word-by-word, not line-by-line, unless `IFS` is changed).

**Example — Correct line-by-line looping over `grep` output (recommended method):**
```bash
#!/bin/bash
grep "bash" /etc/passwd | while read -r line
do
    echo "User line: $line"
done
```
> **Beginner Tip:** When you need to process **whole lines** (which may contain spaces) from a command's output, it is safer to **pipe into a `while read` loop** (as shown above) rather than a `for` loop, because `for` splits on spaces/`IFS` by default and can accidentally break a single line into multiple "items."

**Example — Counting matching lines using `grep -c` inside a loop:**
```bash
#!/bin/bash
for file in *.log
do
    count=$(grep -c "fail" "$file")
    echo "$file has $count line(s) containing 'fail'"
done
```

### 10.2 `while` Loop

**Concept:** Repeats commands **as long as** the condition remains **true**.

**Syntax:**
```bash
while condition
do
    commands
done
```
- Commands are executed **only if the condition returns true**, and continue looping until it becomes false.

**Example:**
```bash
#!/bin/bash
count=1
while [ $count -le 5 ]
do
    echo "Count is: $count"
    count=$((count + 1))
done
```

**Output:**
```
Count is: 1
Count is: 2
Count is: 3
Count is: 4
Count is: 5
```

### 10.3 `until` Loop

**Concept:** Repeats commands **until** the condition becomes **true** — i.e., it loops **while the condition is false**. This is the logical opposite of `while`.

**Syntax:**
```bash
until condition
do
    commands
done
```
- Commands execute **only if the condition returns false**, and looping stops once the condition becomes true.

**Example:**
```bash
#!/bin/bash
count=1
until [ $count -gt 5 ]
do
    echo "Count is: $count"
    count=$((count + 1))
done
```

**Output:**
```
Count is: 1
Count is: 2
Count is: 3
Count is: 4
Count is: 5
```

### `for` vs `while` vs `until` — Comparison Table

| Loop Type | Executes When | Best Used For |
|---|---|---|
| `for` | Once per item in a list | Iterating over a known set/list of items |
| `while` | Condition is **true** | Repeating until a condition becomes false (e.g., counters, reading files line by line) |
| `until` | Condition is **false** | Repeating until a condition becomes true (opposite logic of `while`) |

---

## 11. Conditional Statements

### 11.1 `if` Statement

**Concept:** Executes a block of commands **only if** a given condition evaluates to true.

**Syntax:**
```bash
if condition
then
    commands
fi
```

**One-line variant:**
```bash
if condition; then
    commands
fi
```
- Commands are executed **only if the condition returns true**.

**Example:**
```bash
#!/bin/bash
age=20
if [ $age -ge 18 ]; then
    echo "You are eligible to vote."
fi
```

**Example with `if-elif-else`:**
```bash
#!/bin/bash
marks=75
if [ $marks -ge 90 ]; then
    echo "Grade: A"
elif [ $marks -ge 60 ]; then
    echo "Grade: B"
else
    echo "Grade: C"
fi
```

**Example — Checking if a file exists before reading it:**
```bash
#!/bin/bash
filename="data.txt"

if [ -f "$filename" ]; then
    echo "$filename exists. Reading contents..."
    cat "$filename"
else
    echo "$filename does not exist!"
fi
```

**Example — Checking script arguments using `if` (combines Sections 8 and 11):**
```bash
#!/bin/bash
if [ $# -lt 1 ]; then
    echo "Error: No argument supplied. Usage: $0 <name>"
else
    echo "Hello, $1!"
fi
```
**Run:**
```bash
./greet.sh
# Output: Error: No argument supplied. Usage: ./greet.sh <name>

./greet.sh Aditi
# Output: Hello, Aditi!
```

**Example — Nested `if` statement:**
```bash
#!/bin/bash
num=15

if [ $num -gt 0 ]; then
    if [ $((num % 2)) -eq 0 ]; then
        echo "$num is a positive even number"
    else
        echo "$num is a positive odd number"
    fi
else
    echo "$num is not positive"
fi
```

> **Beginner Tip:** Always leave a **space** after `[` and before `]` in test expressions (e.g., `[ $x -eq 5 ]`, **not** `[$x -eq 5]`) — this is one of the most common beginner mistakes and causes a `command not found` or syntax error, because `[` is actually a **command**, not just a bracket symbol.

### 11.2 `case` Statement

**Concept:** A multi-way branch statement — used as a cleaner alternative to multiple `if-elif` statements when comparing **one variable** against **multiple patterns**.

**Syntax:**
```bash
case var in
    pattern1)
        commands
        ;;
    pattern2)
        commands
        ;;
esac
```
- Commands are executed for **each pattern that matches** the variable `var`.
- `;;` marks the end of each pattern's command block (similar to `break` in other languages).

**Example:**
```bash
#!/bin/bash
echo "Enter a fruit name:"
read fruit

case $fruit in
    apple)
        echo "Apples are red or green."
        ;;
    banana)
        echo "Bananas are yellow."
        ;;
    mango)
        echo "Mangoes are the king of fruits!"
        ;;
    *)
        echo "Unknown fruit."
        ;;
esac
```

> **Note:** The `*)` pattern acts as a **default/catch-all** case, similar to `default` in C/Java `switch` statements.

---

## 12. Conditions and Test Expressions

Bash supports multiple ways to write conditions used in `if`, `while`, and `until` statements.

### 12.1 Types of Conditions

| Condition Type | Syntax | Example |
|---|---|---|
| Command | Any command (checks its exit status) | `wc -l file` |
| Pipeline | Combination of commands using Pipe | `who \| grep "joy" > /dev/null` |
| Test expression | `test exprn` or `[ exprn ]` | `test -e file` / `[ -e file ]` |
| Negation | `!` before condition | `! condition` |
| Extended test | `[[ exprn ]]` (Bash-specific, supports pattern matching) | `[[ $ver == 5.* ]]` |
| Arithmetic evaluation | `(( exprn ))` | `(( $v ** 2 > 10 ))` |

### Explanation of Expression Types
- **file comparison** — comparing properties/timestamps of files
- **unary** — tests applied to a single operand (e.g., checking if a file exists)
- **binary** — tests comparing two operands (e.g., comparing two files or two strings)
- **numeric** — comparing numbers
- **string comparison** — comparing text values

### Example — Using Different Condition Types

```bash
# Using a command as a condition
if wc -l myfile.txt > /dev/null; then
    echo "File has lines"
fi

# Using a pipeline as a condition
if who | grep "joy" > /dev/null; then
    echo "User joy is logged in"
fi

# Using test expression
if [ -e myfile.txt ]; then
    echo "File exists"
fi

# Using negation
if ! [ -e myfile.txt ]; then
    echo "File does not exist"
fi

# Using extended test (pattern matching)
ver="5.2"
if [[ $ver == 5.* ]]; then
    echo "Version starts with 5."
fi

# Using arithmetic evaluation
v=4
if (( v ** 2 > 10 )); then
    echo "Square of v is greater than 10"
fi
```

### 12.2 Test — Numeric Comparisons

| Operator | Meaning |
|---|---|
| `$n1 -eq $n2` | Check if `n1` is **equal to** `n2` |
| `$n1 -ge $n2` | Check if `n1` is **greater than or equal to** `n2` |
| `$n1 -gt $n2` | Check if `n1` is **greater than** `n2` |
| `$n1 -le $n2` | Check if `n1` is **less than or equal to** `n2` |
| `$n1 -lt $n2` | Check if `n1` is **less than** `n2` |
| `$n1 -ne $n2` | Check if `n1` is **not equal to** `n2` |

**Example:**
```bash
n1=10
n2=20
if [ $n1 -lt $n2 ]; then
    echo "$n1 is less than $n2"
fi
```

### 12.3 Test — String Comparisons

| Operator | Meaning |
|---|---|
| `$str1 = $str2` | Check if `str1` is **same as** `str2` |
| `$str1 != $str2` | Check if `str1` is **not same as** `str2` |
| `$str1 < $str2` | Check if `str1` is **less than** `str2` (lexicographically) |
| `$str1 > $str2` | Check if `str1` is **greater than** `str2` (lexicographically) |
| `-n $str2` | Check if string has length **greater than zero** |
| `-z $str2` | Check if string has length of **zero** |

**Example:**
```bash
str1="hello"
str2="world"

if [ "$str1" != "$str2" ]; then
    echo "Strings are different"
fi

if [ -z "$empty_var" ]; then
    echo "Variable is empty"
fi
```

> **Note:** `<` and `>` need to be used carefully inside `[ ]` — they must often be escaped (`\<`, `\>`) or used inside `[[ ]]` to avoid being interpreted as redirection operators.

### 12.4 Unary File Comparisons

Used to test a **single file's** existence/properties.

| Operator | Meaning |
|---|---|
| `-e file` | Check if file **exists** |
| `-d file` | Check if file exists and is a **directory** |
| `-f file` | Check if file exists and is a **regular file** |
| `-r file` | Check if file exists and is **readable** |
| `-s file` | Check if file exists and is **not empty** (size > 0) |
| `-w file` | Check if file exists and is **writable** |
| `-x file` | Check if file exists and is **executable** |
| `-O file` | Check if file exists and is **owned by current user** |
| `-G file` | Check if file exists and its **default group** matches the current user's group |

**Example:**
```bash
if [ -d /home/user/scripts ]; then
    echo "Directory exists"
fi

if [ -x backup.sh ]; then
    echo "Script is executable"
fi
```

### 12.5 Binary File Comparisons

Used to **compare two files** with each other.

| Operator | Meaning |
|---|---|
| `file1 -nt file2` | Check if `file1` is **newer than** `file2` |
| `file1 -ot file2` | Check if `file1` is **older than** `file2` |

**Example:**
```bash
if [ file1.txt -nt file2.txt ]; then
    echo "file1.txt was modified more recently than file2.txt"
fi
```

---

## 13. Functions in Shell Scripts

### Concept

A **function** is a reusable block of code that can be defined once and called (invoked) multiple times within a script — this avoids code repetition and keeps scripts modular (relates back to the **"Do one thing well"** software tools principle).

### Syntax

```bash
myfunc() {
    commands
}
```
- The commands are executed **each time `myfunc` is called**.
- **Important Rule:** Function **definitions must come before** the calls (i.e., a function must be defined earlier in the script than the point where it is called).

### Example

```bash
#!/bin/bash

# Function definition
greet() {
    echo "Hello, $1! Welcome to Bash scripting."
}

# Function call
greet "Ankit"
greet "Simran"
```

**Output:**
```
Hello, Ankit! Welcome to Bash scripting.
Hello, Simran! Welcome to Bash scripting.
```

### Function With Return Value

```bash
#!/bin/bash

add_numbers() {
    result=$(( $1 + $2 ))
    echo $result
}

sum=$(add_numbers 5 10)
echo "Sum is: $sum"
```

**Output:**
```
Sum is: 15
```

### Definition vs Call — Quick Reference

| Term | Meaning |
|---|---|
| **Definition** | The block where the function's commands are written (`myfunc() { ... }`) |
| **Call** | Simply writing the function's name (`myfunc`) to execute it |

---

## 14. Summary

This lecture (Week 6, Lecture 2 & 3) introduced the foundations of **Bash Shell Scripting — Part 1**, covering how to create custom, reusable commands using shell scripts. Key takeaways:

1. **Software Tools Principles** guide good scripting practice: build small, focused, text-based, non-chatty tools that work well together via standard I/O and pipelines.
2. A **shell script** is a text file with a shebang line (`#!interpreter`), comments, commands, loops, variables, case statements, and functions — commonly created using an editor such as `vi` (`vi s1.sh` → insert mode with `i` → `Esc` → `:wq` to save).
3. Scripts can be run in two ways:
   - **Sourced** (`. script` / `source script`) — runs in the current shell, **same PID** (verified using `echo $$`), environment (variables) **persists** after the script ends.
   - **Executed** (`./script`, after `chmod +x script`) — runs in a **new child process**, **different PID**, environment changes are **lost** after completion. This parent-child relationship can be visualized using `ps --forest`.
   - `$0` inside a script also reveals how it was invoked: the script's own name when **executed**, versus the parent shell's name (e.g., `bash`) when **sourced**.
4. Scripts can be located and run using an **absolute path**, a **relative path**, or by placing them in a directory listed in **`$PATH`** (watch the directory order in `$PATH`).
5. Bash reads different startup files depending on whether it is a **login shell** (`/etc/profile`, `~/.bash_profile`, etc.) or a **non-login shell** (`/etc/bash.bashrc`, `~/.bashrc`).
6. **Output** can be generated using `echo` (simple) or `printf` (formatted, like C).
7. **Input** is captured using the `read var` command.
8. **Script arguments** are accessed via `$0`, `$#`, `$1...${11}`, `$*`/`$@`, with `"$@"` preserving individual arguments and `"$*"` merging them into one string.
9. **Command substitution** (`` var=`command` `` or `var=$(command)`) captures a command's output into a variable.
10. **Loops** — `for` (iterate over a list), `while` (repeat while true), `until` (repeat while false) — all use `do ... done` blocks.
11. **Conditionals** — `if/then/fi` for true-condition branching, and `case/esac` for multi-pattern matching on a single variable.
12. **Test expressions** — `[ ]`, `[[ ]]`, `(( ))`, and the `test` command — support numeric comparisons (`-eq`, `-gt`, etc.), string comparisons (`=`, `!=`, `-z`, `-n`), and file comparisons (unary: `-e`, `-d`, `-f`, `-r`, `-w`, `-x`; binary: `-nt`, `-ot`).
13. **Functions** (`myfunc() { commands }`) allow code reuse; they must be **defined before** they are called.

> **Final Exam Tip:** Focus especially on the **differences** between: sourced vs executed scripts, `$*` vs `$@`, `while` vs `until`, and the various **test operators** (numeric vs string vs file) — these are the most commonly asked comparison-based questions in shell scripting exams.
