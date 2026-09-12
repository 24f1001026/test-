# PROMPT STRINGS
### A Complete Academic & Exam-Reference Guide
*(Shells · Bash Prompt Variables · Escape Sequences · Python Interactive Prompts)*

> **Scope of this document:** This guide is prepared as a self-contained study resource for university/college exams, viva-voce, and practical lab assessments on the topic of **Prompt Strings** in Unix/Linux shells and the Python interpreter. Every concept is explained with definitions, syntax, working examples, output, and exam-oriented notes (important points, common mistakes, and probable questions).

---

## 📑 Table of Contents

1. [Introduction](#1-introduction)
   - 1.1 [Definition of a Prompt String](#11-definition-of-a-prompt-string)
   - 1.2 [Why Prompt Strings Are Important](#12-why-prompt-strings-are-important)
   - 1.3 [Where Prompt Strings Are Used](#13-where-prompt-strings-are-used)
2. [Types of Shells](#2-types-of-shells)
   - 2.1 [Comparison Table of Shells](#21-comparison-table-of-shells)
   - 2.2 [How to Check Your Current Shell](#22-how-to-check-your-current-shell)
3. [Bash Prompt Variables — Overview](#3-bash-prompt-variables--overview)
4. [PS1 — Primary Prompt String](#4-ps1--primary-prompt-string)
5. [PS2 — Secondary Prompt String](#5-ps2--secondary-prompt-string)
6. [PS3 — Select Loop Prompt](#6-ps3--select-loop-prompt)
7. [PS4 — Execution Trace Prompt](#7-ps4--execution-trace-prompt)
8. [Escape Sequences in Prompt Strings](#8-escape-sequences-in-prompt-strings)
   - 8.1 [Complete Escape Sequence Table](#81-complete-escape-sequence-table)
   - 8.2 [Worked Examples for Every Escape Sequence](#82-worked-examples-for-every-escape-sequence)
9. [Customizing $PS1 — Step by Step](#9-customizing-ps1--step-by-step)
   - 9.1 [Adding Colors to PS1](#91-adding-colors-to-ps1)
   - 9.2 [Multi-line PS1 Prompts](#92-multi-line-ps1-prompts)
10. [\t for Time and \d for Date — Detailed Study](#10-t-for-time-and-d-for-date--detailed-study)
11. [\# — Command History Numbering](#11--command-history-numbering)
12. [Making Prompt Changes Permanent — source ~/.bashrc](#12-making-prompt-changes-permanent--source-bashrc)
    - 12.1 [Difference Between Login and Non-Login Shells](#121-difference-between-login-and-non-login-shells)
    - 12.2 [Order of File Loading](#122-order-of-file-loading)
13. [PS2 in Detail — Unclosed Brackets/Quotes](#13-ps2-in-detail--unclosed-bracketsquotes)
14. [PS3 in Detail — Select Loop Example](#14-ps3-in-detail--select-loop-example)
15. [PS4 in Detail — Debugging with set -x](#15-ps4-in-detail--debugging-with-set--x)
16. [Python Interactive Prompts — sys.ps1 and sys.ps2](#16-python-interactive-prompts--sysps1-and-sysps2)
    - 16.1 [Static Custom Prompts](#161-static-custom-prompts)
    - 16.2 [Dynamic Prompts Using `__str__`](#162-dynamic-prompts-using-__str__)
    - 16.3 [Real-World Style Examples (Jupyter-like Counters)](#163-real-world-style-examples-jupyter-like-counters)
17. [Other REPL Tools: Octave, Gnuplot, Sage](#17-other-repl-tools-octave-gnuplot-sage)
18. [Comparative Analysis: Shell vs Python Prompts](#18-comparative-analysis-shell-vs-python-prompts)
19. [Common Errors and Troubleshooting](#19-common-errors-and-troubleshooting)
20. [Important Points to Remember (Exam Focus)](#20-important-points-to-remember-exam-focus)
21. [Probable Exam Questions & Model Answers](#21-probable-exam-questions--model-answers)
22. [Summary](#22-summary)
23. [Final Cheat Sheet](#23-final-cheat-sheet)

---

## 1. Introduction

### 1.1 Definition of a Prompt String

> **Definition:** A **prompt string** is a sequence of characters — often containing special, dynamically-evaluated escape codes — that a command-line interpreter (shell) or an interactive language environment (REPL) displays to signal that it is ready to accept the next unit of user input.

In simple terms, the prompt is the "cue" you see before you type — for example:

```
john@myserver:~$
```

This is **not a fixed string of text**. It is *constructed at runtime* by substituting special variables such as the current username, hostname, and directory into a template. This is the defining characteristic that separates a "prompt string" from ordinary static text.

### 1.2 Why Prompt Strings Are Important

| Benefit | Explanation |
|---|---|
| **Context Awareness** | Instantly shows *who* you are, *where* you are (directory/host), and *when* (time/date), without running separate commands like `whoami`, `pwd`, `date`. |
| **Mode Differentiation** | Different prompts (PS1 vs PS2 vs PS3 vs PS4) tell you *what kind* of input the shell expects — a fresh command, a continuation, a menu choice, or a debug trace line. |
| **Productivity** | Reduces cognitive load and typing — you don't need to query system state manually. |
| **Debugging Aid** | PS4, combined with `set -x`, is one of the most powerful built-in shell script debugging tools. |
| **Personalization** | Lets users/organizations brand or color-code their terminal (e.g., red prompt for root, green for normal user — a common security convention). |

### 1.3 Where Prompt Strings Are Used

Prompt strings are a **universal REPL concept** — they appear anywhere a program follows a **Read–Evaluate–Print Loop (REPL)** pattern:

- **Unix/Linux shells:** bash, dash, zsh, ksh, csh
- **Programming language interactive consoles:** Python
- **Scientific/mathematical tools:** Octave, Sage
- **Utilities:** Gnuplot

> **Exam Tip:** If asked to *define* a REPL — Read (reads user input) → Evaluate (executes/interprets it) → Print (displays the result) → Loop (returns to read the next input). The **prompt string** is what visually marks the start of each "Read" step.

---

## 2. Types of Shells

A **shell** is a command-line program that acts as an interface between the user and the operating system kernel, interpreting typed commands and executing them.

### 2.1 Comparison Table of Shells

| Shell | Full Form | Origin/Author | Default Prompt Symbol | Key Characteristic |
|---|---|---|---|---|
| **bash** | Bourne Again SHell | Brian Fox (GNU Project, 1989) | `$` (user) / `#` (root) | Most common Linux default shell; superset of the original Bourne shell (`sh`) with scripting enhancements |
| **dash** | Debian Almquist Shell | Kenneth Almquist | `$` | Extremely lightweight and fast; used as `/bin/sh` in Debian/Ubuntu for **script execution** (not usually interactive use) |
| **zsh** | Z Shell | Paul Falstad | `%` | Highly extensible; supports plugins/themes (e.g., Oh-My-Zsh); default shell on macOS since Catalina |
| **ksh** | Korn Shell | David Korn (AT&T Bell Labs) | `$` | Merges Bourne shell syntax with C-shell-like interactive features (command history, aliasing) |
| **csh** | C Shell | Bill Joy (UC Berkeley) | `%` | Syntax resembles the C programming language; scripting considered less robust/portable than bash |

### 2.2 How to Check Your Current Shell

```bash
echo $SHELL          # shows the shell set as your default login shell
echo $0               # shows the shell currently running
ps -p $$               # shows the process name of current shell
```

> **Exam Tip:** A common trick question is *"Which file defines PS1 for dash?"* — Remember: **dash is normally non-interactive** (used for running `/bin/sh` scripts) and typically does **not** use PS1 the way bash does interactively.

---

## 3. Bash Prompt Variables — Overview

Bash exposes **four special environment variables**, collectively controlling every prompt situation you will encounter:


| Variable | Purpose | Default Value |
|----------|---------|----------------|
| **PS1** | Primary prompt string — shown for every new command | `$` (or `\u@\h:\w\$` on many distros) |
| **PS2** | Secondary prompt — shown when a command spans multiple lines | `>` |
| **PS3** | Prompt shown inside a `select` loop while waiting for a menu choice | `#?` |
| **PS4** | Prompt shown before each line when execution tracing (`set -x`) is enabled | `+` |


```
┌─────────┬────────────────────────────────────────┬────────────┐
│ Variable│ Triggered When...                      │ Default    │
├─────────┼─────────────────────────────────────── ┼────────────┤
│  PS1    │ Shell is ready for a brand-new command │   \s-\v\$  │
│  PS2    │ A started command is incomplete        │     >      │
│  PS3    │ Inside a `select` menu loop            │     #?     │
│  PS4    │ Each line during `set -x` trace mode   │     +      │
└─────────┴────────────────────────────────────────┴────────────┘
```

All four are **ordinary shell variables** — they can be viewed, set, exported, and reset just like any other variable:

```bash
echo $PS1        # view current PS1
PS1="myshell> "  # change it (session only)
unset PS1         # remove customization (bash falls back to a minimal prompt)
```

> **Important distinction for exams:** PS1–PS4 are plain **string variables holding a template**; they are **not functions**. Bash re-evaluates escape sequences and embedded command substitutions inside them *every time* the prompt is displayed — this is what makes the clock/date/directory update live.

---

## 4. PS1 — Primary Prompt String

**Definition:** `PS1` (Prompt String 1) is displayed by the shell every time it is ready to accept a **new**, top-level command.

**Default value (typical Ubuntu bash):**
```bash
PS1='\u@\h:\w\$ '
```

**Basic demonstration:**

```bash
$ echo "Hello World"
Hello World
$ pwd
/home/john
$
```

Each `$` shown above **is PS1** — redrawn fresh after every completed command.

**Checking and setting PS1:**
```bash
echo $PS1
# Output: \u@\h:\w\$

PS1="Learning-Bash > "
```
```
Learning-Bash > ls
Desktop  Documents  Downloads
Learning-Bash >
```

> **Exam Tip:** PS1 is evaluated **after** the previous command finishes and **before** the shell reads your next line — this is why it can show updated info (like new directory after `cd`) every single time.

---

## 5. PS2 — Secondary Prompt String

**Definition:** `PS2` is displayed when the shell has determined that the command entered so far is **syntactically incomplete** and needs more input before it can be executed.

**Default value:** `>`

**Simple demonstration:**
```bash
$ echo "Welcome
> to
> Bash"
Welcome
to
Bash
```

Here the shell keeps showing `>` (PS2) because the double quote opened on line 1 was not yet closed.

**Custom PS2:**
```bash
PS2="continue-> "
```
```
$ echo "Multi
continue-> line
continue-> string"
Multi
line
string
```

---

## 6. PS3 — Select Loop Prompt

**Definition:** `PS3` is a special prompt used **exclusively inside a bash `select` loop**, which is bash's built-in construct for generating a simple numbered menu and capturing the user's numeric choice.

**Default value:** `#?`     

 `(Can't access by echo $PS3 )`

**Syntax of a select loop:**
```bash
select variable in list_of_items
do
   commands
done
```

**Full working example:**
```bash
#!/bin/bash
PS3="Please enter your choice (1-4): "

echo "----- MAIN MENU -----"
select choice in "Create File" "Delete File" "List Files" "Exit"
do
    case $choice in
        "Create File")
            touch newfile.txt
            echo "File created."
            ;;
        "Delete File")
            rm -i newfile.txt
            echo "File deleted."
            ;;
        "List Files")
            ls -l
            ;;
        "Exit")
            echo "Exiting program."
            break
            ;;
        *)
            echo "Invalid selection, please try again."
            ;;
    esac
done
```

**Sample run:**
```
----- MAIN MENU -----
1) Create File
2) Delete File
3) List Files
4) Exit
Please enter your choice (1-4): 3
-rw-r--r-- 1 john john  220 Jan  5 09:00 file1.txt
Please enter your choice (1-4): 4
Exiting program.
```

> **Important:** If the user enters an invalid number (or presses Enter with no input), `$choice` is set to an **empty/null** value, and bash re-displays the `PS3` prompt automatically — this is handled by the `*)` default case above.

---

## 7. PS4 — Execution Trace Prompt

**Definition:** `PS4` is the prompt string prefixed to each line of shell script output when **execution tracing** (`set -x`) is active. It is bash's built-in step-by-step debugger prompt.

**Default value:** `+`

**Basic demonstration:**
```bash
#!/bin/bash
set -x
a=5
b=3
echo "Sum: $((a+b))"
set +x
```

**Output:**
```
+ a=5
+ b=3
+ echo 'Sum: 8'
Sum: 8
```

**Enhanced PS4 with line numbers (very common exam/interview example):**
```bash
#!/bin/bash
PS4='+ [Line ${LINENO}] '
set -x

x=10
y=0
if [ $y -ne 0 ]; then
    echo $((x / y))
else
    echo "Cannot divide by zero"
fi

set +x
```

**Output:**
```
+ [Line 5] x=10
+ [Line 6] y=0
+ [Line 7] '[' 0 -ne 0 ']'
+ [Line 10] echo 'Cannot divide by zero'
Cannot divide by zero
```

> **Exam Tip:** `${LINENO}` is a built-in bash variable holding the **current line number** in the script — extremely popular in PS4 customization questions.

---

## 8. Escape Sequences in Prompt Strings

### 8.1 Complete Escape Sequence Table

| Escape | Meaning | Example Output |
|---|---|---|
| `\a` | ASCII bell character (produces a beep, no visible text) | *(sound only)* |
| `\A` | Current time, 24-hour, `HH:MM` | `14:05` |
| `\d` | Date as "Weekday Month Day" | `Tue May 26` |
| `\D{format}` | Date formatted using `strftime` (e.g., `\D{%Y-%m-%d}`) | `2026-09-10` |
| `\e` | ASCII escape character (used for starting color codes) | *(invisible control char)* |
| `\h` | Hostname up to the first `.` | `myserver` |
| `\H` | Full hostname | `myserver.example.com` |
| `\j` | Number of jobs currently managed by the shell | `2` |
| `\l` | Basename of the shell's terminal device name | `pts/0` |
| `\n` | Newline | *(line break)* |
| `\r` | Carriage return | *(cursor to line start)* |
| `\s` | Name of the shell (basename of `$0`) | `bash` |
| `\t` | Current time, 24-hour, `HH:MM:SS` | `14:05:32` |
| `\T` | Current time, 12-hour, `HH:MM:SS` | `02:05:32` |
| `\@` | Current time, 12-hour, with am/pm | `02:05 PM` |
| `\u` | Current username | `john` |
| `\v` | Bash version (short, e.g. `5.1`) | `5.1` |
| `\V` | Bash release (version + patch level) | `5.1.16` |
| `\w` | Current working directory (full path, `~` for home) | `~/projects` |
| `\W` | Basename only of current working directory | `projects` |
| `\!` | History number of this command (includes commands from earlier sessions, saved in the history file) | `102` |
| `\#` | Command number of this command (counts only within the **current session**) | `7` |
| `\$` | `#` if effective UID is 0 (root), else `$` | `$` |
| `\\` | A literal backslash | `\` |
| `\[` ... `\]` | Wraps a sequence of **non-printing characters** (e.g., color codes) so bash correctly calculates cursor/line-wrap position | *(no visible output)* |

> **Exam Tip:** `\!` vs `\#` is one of the most commonly confused pairs. `\!` = position in the **saved history file** (keeps growing across sessions); `\#` = position **within the current terminal session only** (resets to 1 each time you open a new terminal).

### 8.2 Worked Examples for Every Escape Sequence

**Example — Combining multiple escapes into one professional prompt:**
```bash
PS1="┌─[\u@\h]─[\w]\n└─[\t]\$ "
```
**Output:**
```
┌─[john@myserver]─[~/projects]
└─[14:22:10]$
```
*(Notice `\n` creates a two-line prompt — the top line shows identity/location, the bottom line shows time and the actual input cursor.)*

**Example — Root vs normal user distinguishing prompt:**
```bash
PS1="\u@\h:\w\$ "
```
- Normal user → `john@myserver:~$`
- Root user → `root@myserver:~#`

*(This works because `\$` automatically switches based on UID — a key security-awareness convention: seeing `#` should always alert you that you have root privileges.)*

---

## 9. Customizing $PS1 — Step by Step

### Step 1: View the existing PS1
```bash
echo $PS1
```

### Step 2: Try a new value temporarily
```bash
PS1="[\u@\h \W]\$ "
```
```
[john@myserver projects]$
```

### Step 3: Test with directory changes
```bash
[john@myserver projects]$ cd ..
[john@myserver john]$
```

### 9.1 Adding Colors to PS1

Bash supports **ANSI escape codes** for text color. The general syntax is:

```
\[\e[<code>m\]text\[\e[0m\]
```

| Code | Color |
|---|---|
| 30 | Black |
| 31 | Red |
| 32 | Green |
| 33 | Yellow |
| 34 | Blue |
| 35 | Magenta |
| 36 | Cyan |
| 37 | White |
| 0  | Reset (back to default) |

**Example — Green username/host, blue directory:**
```bash
PS1="\[\e[32m\]\u@\h\[\e[0m\]:\[\e[34m\]\w\[\e[0m\]\$ "
```
**Visual result (conceptually):**
```
john@myserver:/home/john/projects$      (username@host in green, path in blue)
```

> **Exam Tip:** The `\[` and `\]` wrappers are **mandatory** around color codes. Omitting them doesn't break the colors visually, but it **breaks bash's cursor-position math**, causing the terminal to misbehave when you press the Up arrow, use `Ctrl+R` search, or resize the window.

### 9.2 Multi-line PS1 Prompts

```bash
PS1="\n\u@\h [\w]\n\$ "
```
**Output:**
```

john@myserver [~/projects]
$
```
*(A blank line, then info, then the actual prompt symbol on its own line — commonly used to keep long paths from crowding the typing area.)*

---

## 10. \t for Time and \d for Date — Detailed Study

| Escape | Format | Includes Seconds? |
|---|---|---|
| `\A` | `HH:MM` (24-hr) | No |
| `\t` | `HH:MM:SS` (24-hr) | Yes |
| `\T` | `HH:MM:SS` (12-hr) | Yes |
| `\@` | `HH:MM am/pm` (12-hr) | No |
| `\d` | `Weekday Month Day` | N/A (date, not time) |

**Example 1 — Full date and time:**
```bash
PS1="[\d \t] \u@\h:\w\$ "
```
```
[Tue May 26 14:32:07] john@myserver:~$
```

**Example 2 — 12-hour clock with am/pm, more human-friendly:**
```bash
PS1="(\@) \u@\h:\w\$ "
```
```
(02:32 PM) john@myserver:~$
```

**Example 3 — Custom date format with `\D{}`:**
```bash
PS1="[\D{%d-%m-%Y}] \u@\h:\w\$ "
```
```
[26-05-2026] john@myserver:~$
```

> **Exam Tip:** `\D{format}` internally uses the C library's `strftime()` format codes (e.g., `%Y` = 4-digit year, `%m` = month number, `%d` = day) — the same specifiers used in many other programming languages, so this is a good cross-topic link to mention in answers.

---

## 11. \# — Command History Numbering

`\#` displays the number of the command **within the current shell session**, incrementing by 1 each time you press Enter on a new command.

```bash
PS1="\#: \u@\h:\w\$ "
```

**Sample session:**
```
1: john@myserver:~$ ls
Desktop  Documents
2: john@myserver:~$ cd Documents
3: john@myserver:Documents$ pwd
/home/john/Documents
4: john@myserver:Documents$
```

**Related and often paired command:** `history`
```bash
history          # lists all past commands with their numbers
!4                # re-executes command number 4 from history
```

> **Exam Tip:** Distinguish clearly — `\#` (session command count, resets each new shell) vs `\!` (persistent history count, read from `~/.bash_history`, keeps growing across sessions until the history file is cleared).

---

## 12. Making Prompt Changes Permanent — source ~/.bashrc

Typing `PS1=...` directly at the terminal only changes the **current running shell process** — it is lost the moment you close that terminal window. To make a prompt persist across **all future terminal sessions**, you must place the assignment inside a **shell startup/configuration file**.

### Step-by-Step Procedure:

**Step 1 — Open the configuration file:**
```bash
nano ~/.bashrc
```
*(You may also use `vim`, `gedit`, or any text editor.)*

**Step 2 — Add the export line, typically near the end of the file:**
```bash
export PS1="\[\e[32m\]\u@\h\[\e[0m\]:\w\$ "
```

**Step 3 — Save and close the editor.**
*(In nano: `Ctrl+O` to save, `Enter` to confirm, `Ctrl+X` to exit.)*

**Step 4 — Reload the file into the current session:**
```bash
source ~/.bashrc
```
*(Equivalent shorthand: `. ~/.bashrc` — the single dot is a POSIX alias for `source`.)*

**Result:** The new prompt applies **immediately** to the current terminal, and will also automatically load in **every new terminal** you open from now on, because `.bashrc` is executed at the start of every new interactive non-login bash session.

### 12.1 Difference Between Login and Non-Login Shells

| Shell Type | When It Occurs | Startup File(s) Read |
|---|---|---|
| **Login shell** | Logging in via console, SSH, or `su -` | `/etc/profile`, then first found of `~/.bash_profile`, `~/.bash_login`, `~/.profile` |
| **Non-login (interactive) shell** | Opening a new terminal window/tab in a GUI desktop | `~/.bashrc` |

> **Exam Tip:** This is a **frequently asked** conceptual question: *"Why does changing PS1 in `.bash_profile` sometimes not work in a new terminal tab?"* Answer: because a new GUI terminal tab typically starts a **non-login shell**, which reads `~/.bashrc`, not `~/.bash_profile`. Many `.bash_profile` files explicitly `source ~/.bashrc` to bridge this gap.

### 12.2 Order of File Loading

```
Login Shell:
  /etc/profile → ~/.bash_profile (or ~/.bash_login or ~/.profile)

Non-Login Interactive Shell:
  /etc/bash.bashrc → ~/.bashrc

Non-Interactive Shell (running a script):
  Uses $BASH_ENV if set; PS1 is irrelevant here (no prompt is shown)
```

---

## 13. PS2 in Detail — Unclosed Brackets/Quotes

`PS2` (default `>`) appears whenever bash's **parser** determines the current logical command is **not yet syntactically complete**. Triggers include:

| Trigger | Example |
|---|---|
| Unclosed double/single quote | `echo "Hello` |
| Unclosed parenthesis/brace/bracket | `echo $(echo hi` |
| Trailing backslash (explicit continuation) | `echo Hi \` |
| Open control structure | `if`, `for`, `while`, `case` without their closing keyword |
| Open here-document | `cat << EOF` (waits for the literal `EOF` marker) |

**Example 1 — Unclosed quote:**
```bash
$ echo "This spans
> multiple lines"
This spans
multiple lines
```

**Example 2 — Explicit line continuation:**
```bash
$ echo Hello \
> World
Hello World
```

**Example 3 — Incomplete for-loop:**
```bash
$ for i in 1 2 3
> do
>   echo "Number: $i"
> done
Number: 1
Number: 2
Number: 3
```

**Example 4 — Here-document:**
```bash
$ cat << EOF
> Line one
> Line two
> EOF
Line one
Line two
```

**Customizing PS2:**
```bash
PS2="... waiting> "
```
```
$ echo "test
... waiting> continued"
test
continued
```

> **Exam Tip:** A frequent exam trap: forgetting a **closing quote** on purpose to test if students understand PS2 will appear and the shell will *not* execute the command until the quote is closed (or the user cancels with `Ctrl+C`).

---

## 14. PS3 in Detail — Select Loop Example

Expanding further on the `select` construct's mechanics:

**Key facts about `select`:**
- It automatically numbers each item in the given list starting from `1`.
- The user's numeric choice is stored in the special variable `REPLY`, while the corresponding *text* is stored in the loop variable (e.g., `$choice`).
- The loop **repeats indefinitely** until explicitly broken with `break` — this is why an `Exit`/`Quit` option is essential.
- If `COLUMNS` (terminal width) is large enough, `select` arranges options into multiple columns automatically.

**Extended example demonstrating `$REPLY`:**
```bash
#!/bin/bash
PS3="Pick a number (1-3): "
select color in "Red" "Green" "Blue"
do
    echo "You typed: $REPLY"
    echo "Which maps to: $color"
    break
done
```
**Sample run:**
```
1) Red
2) Green
3) Blue
Pick a number (1-3): 2
You typed: 2
Which maps to: Green
```

> **Exam Tip:** If no `PS3` is set, bash falls back to displaying `#? ` — worth memorizing as the **literal default value**.

---

## 15. PS4 in Detail — Debugging with set -x

**How tracing works internally:**
1. `set -x` (or running the script as `bash -x script.sh`) turns on the `xtrace` shell option.
2. For every **simple command** the shell executes, it prints the command (after **variable/parameter expansion**, so you see actual substituted values) to standard error, prefixed with `PS4`.
3. `set +x` turns tracing back off.

**Example — showing expanded variables:**
```bash
#!/bin/bash
PS4='+ (${LINENO}) '
set -x

name="Alice"
greeting="Hello, $name!"
echo "$greeting"

set +x
```
**Output:**
```
+ (4) name=Alice
+ (5) greeting='Hello, Alice!'
+ (6) echo 'Hello, Alice!'
Hello, Alice!
```

**Running a whole script in trace mode without editing it:**
```bash
bash -x myscript.sh
```

**Nested/function-aware PS4 (advanced, useful for larger scripts):**
```bash
PS4='+ ${BASH_SOURCE}:${LINENO}:${FUNCNAME[0]:-MAIN}: '
```
This additionally shows the **filename** and **current function name** (or `MAIN` if not inside a function) — very handy for debugging scripts with multiple sourced files/functions.

> **Exam Tip:** Remember the **two ways to enable tracing**: (1) `set -x` inside the script, or (2) invoking the whole script with `bash -x`. Also remember `set +x` (plus sign) **disables** tracing — a classic "opposite of what you'd expect" exam trick since `-` usually means "on" and `+` means "off" for bash's `set` options.

---

## 16. Python Interactive Prompts — sys.ps1 and sys.ps2

Python's **interactive interpreter mode** (started by typing `python` or `python3` with no script filename) also uses prompt strings, but the mechanism differs fundamentally from bash:

| Aspect | Bash (PS1–PS4) | Python (sys.ps1, sys.ps2) |
|---|---|---|
| Defined in | Shell environment variables | Attributes of the `sys` module |
| Number of prompt types | 4 (PS1, PS2, PS3, PS4) | 2 (`ps1`, `ps2`) |
| Escape sequences supported? | Yes (`\u`, `\h`, `\w`, etc.) | No — plain strings only, unless manually computed |
| Applies to scripts? | Yes (PS4 applies during script tracing) | No — **only in interactive mode**, never when running a `.py` file |
| Dynamic behavior | Achieved via escape sequences evaluated by bash itself | Achieved via **object `__str__` overriding**, since Python just calls `str()` on the prompt object |

**Default values:**
```python
>>> import sys
>>> sys.ps1
'>>> '
>>> sys.ps2
'... '
```

### 16.1 Static Custom Prompts

```python
>>> import sys
>>> sys.ps1 = "MyPython>>> "
MyPython>>> print("Hello")
Hello
MyPython>>>
```

Changing `sys.ps2` (secondary prompt — appears for incomplete blocks, e.g., inside `if`, `for`, function definitions, or unclosed brackets):
```python
>>> import sys
>>> sys.ps2 = "....: "
>>> if True:
....:     print("Inside block")
....:
Inside block
```

**Unclosed bracket triggering ps2:**
```python
>>> total = (1 + 2 +
... 3 + 4)
>>> print(total)
10
```

### 16.2 Dynamic Prompts Using `__str__`

Because Python's REPL calls `str()` on whatever object is assigned to `sys.ps1`/`sys.ps2` **every single time** it needs to display the prompt, assigning a **custom object** (instead of a plain string) lets you compute a fresh value on each call.

**Rule to remember:** *Any object with a `__str__` method can act as a prompt — Python does not require it to be a `str` instance, only something that `str()` can convert.*

**Example — Live clock prompt:**
```python
import sys
import datetime

class TimePrompt:
    def __str__(self):
        now = datetime.datetime.now().strftime("%H:%M:%S")
        return f"[{now}] >>> "

sys.ps1 = TimePrompt()
```
**Behavior:**
```
[10:15:02] >>> x = 5
[10:15:08] >>> print(x)
5
[10:15:11] >>>
```

**Example — Prompt showing current working directory (like a mini-shell):**
```python
import sys
import os

class DirPrompt:
    def __str__(self):
        return f"{os.getcwd()} >>> "

sys.ps1 = DirPrompt()
```
```
/home/john >>> os.chdir("Documents")
/home/john/Documents >>>
```

### 16.3 Real-World Style Examples (Jupyter-like Counters)

This mirrors how tools like **IPython/Jupyter Notebook** number their `In [n]:` execution prompts:

```python
import sys

class CounterPrompt:
    def __init__(self):
        self.count = 0
    def __str__(self):
        self.count += 1
        return f"In [{self.count}]: "

sys.ps1 = CounterPrompt()
```
```
In [1]: a = 10
In [2]: b = 20
In [3]: a + b
30
In [4]:
```

**Combining counter + timestamp (advanced exam-style example):**
```python
import sys
import datetime

class AdvancedPrompt:
    def __init__(self):
        self.count = 0
    def __str__(self):
        self.count += 1
        now = datetime.datetime.now().strftime("%H:%M:%S")
        return f"[{now}] In[{self.count}]>>> "

sys.ps1 = AdvancedPrompt()
```
```
[10:20:00] In[1]>>> print("test")
test
[10:20:05] In[2]>>>
```

> **Exam Tip:** A favorite viva question: *"Why can't you achieve a live-updating prompt in Python simply by assigning a plain string to `sys.ps1`?"* **Answer:** Because a plain string is evaluated **once**, at assignment time, and never changes afterward. Only an **object with `__str__`** gets *re-evaluated* every time the interpreter needs to draw the prompt — this is the core conceptual difference from bash, where the *shell itself* re-parses escape sequences on every draw.

---

## 17. Other REPL Tools: Octave, Gnuplot, Sage

| Tool | Purpose | Default Prompt | How to Customize |
|---|---|---|---|
| **Octave** | High-level numerical computing (MATLAB-compatible) | `octave:1>` (includes command count) | `PS1('myprompt> ')` function call inside Octave |
| **Gnuplot** | Command-driven interactive plotting utility | `gnuplot>` | `set prompt "myprompt> "` (in supporting versions) |
| **Sage** | Python-based open-source computer algebra/mathematics system | `sage:` | Configurable via underlying IPython settings, since Sage's console is built on IPython |

**Example — Octave prompt showing command number:**
```
octave:1> x = 5
x = 5
octave:2> y = x^2
y = 25
octave:3>
```

> **Exam Tip:** Octave's default prompt **already includes a command counter** (similar to bash's `\#`), which is a nice comparative point to raise if a question asks you to "compare prompt behavior across tools."

---

## 18. Comparative Analysis: Shell vs Python Prompts

| Feature | Bash Shell | Python Interpreter |
|---|---|---|
| Number of distinct prompt variables | 4 (PS1, PS2, PS3, PS4) | 2 (ps1, ps2) |
| Storage location | Environment variable | Attribute inside `sys` module |
| Built-in dynamic placeholders | Yes — extensive escape sequence set | No — must be built manually |
| Persistence across sessions | Via `~/.bashrc` / `~/.bash_profile` | Not persistent by default; must be re-set each interactive session (or automated via a `startup` script referenced by `PYTHONSTARTUP` environment variable) |
| Debug-oriented prompt available? | Yes — PS4 with `set -x` | No direct equivalent (Python debugging typically uses `pdb`, a separate tool) |
| Menu-oriented prompt available? | Yes — PS3 with `select` | No direct equivalent |

> **Exam Tip:** Note the `PYTHONSTARTUP` environment variable — set it to the path of a `.py` file, and Python will automatically execute that file (where you can set `sys.ps1`/`sys.ps2`) every time you start an interactive session, giving a *persistence* mechanism roughly analogous to bash's `.bashrc`.

```bash
export PYTHONSTARTUP=~/.pythonrc.py
```

---

## 19. Common Errors and Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| Prompt shows literal `\u@\h` instead of username/hostname | Used single quotes incorrectly, or the shell isn't bash (escape sequences aren't supported the same way in all shells) | Ensure you're in bash; use `PS1='\u@\h:\w\$ '` with single quotes so escapes aren't expanded too early by the outer shell |
| Terminal cursor jumps oddly / line-wrapping breaks after adding colors | Missing `\[ \]` wrappers around non-printing ANSI codes | Wrap every color code sequence in `\[...\]` |
| PS1 changes disappear after closing terminal | Change was only made at the command line, not saved to a config file | Add `export PS1=...` to `~/.bashrc` and run `source ~/.bashrc` |
| `PS3` prompt never appears | Not inside a `select` loop, or `select` block already exited | Confirm the prompt logic is written inside `select ... do ... done` |
| `PS4` not showing during script run | Forgot to add `set -x`, or added `set +x` too early | Ensure `set -x` precedes the code you want traced, and `set +x` comes only after |
| Python prompt not updating dynamically | Assigned a **plain string**, not an object with `__str__` | Define a class with a `__str__` method and assign an **instance** of it to `sys.ps1` |
| `sys.ps1` changes have no visible effect | Python was run as a **script** (`python file.py`) rather than **interactively** | `sys.ps1`/`sys.ps2` only apply in interactive mode (run `python` with no arguments, or `python -i file.py`) |

---

## 20. Important Points to Remember (Exam Focus)

- ✅ Prompt strings are **dynamically evaluated templates**, not static text.
- ✅ Bash has exactly **four** prompt variables: **PS1, PS2, PS3, PS4**.
- ✅ Default values: PS1 = `$` (or `\u@\h:\w\$`), PS2 = `>`, PS3 = `#?`, PS4 = `+`.
- ✅ `\#` = command count **within current session**; `\!` = command count **from history file** (persists across sessions).
- ✅ Escape sequences **must** be defined using **single quotes** around the PS1 assignment to prevent premature shell expansion: `PS1='\u@\h:\w\$ '`.
- ✅ Non-printing sequences (colors) **must** be wrapped in `\[ \]` or the terminal miscalculates line length.
- ✅ Temporary changes (typed directly) vs. Permanent changes (`~/.bashrc` + `source`).
- ✅ `source ~/.bashrc` (or `. ~/.bashrc`) reloads config **without restarting** the terminal.
- ✅ Login shells read `~/.bash_profile`; non-login interactive shells read `~/.bashrc`.
- ✅ PS3 works **only** inside a `select` loop; falls back to `#?` if unset.
- ✅ PS4 requires `set -x` to actually appear; `set +x` disables tracing.
- ✅ Python's `sys.ps1`/`sys.ps2` apply **only in interactive mode**, never for `.py` scripts run normally.
- ✅ To make a Python prompt **dynamic**, override `__str__` in a custom class and assign an **instance** (not a string) to `sys.ps1`.
- ✅ `PYTHONSTARTUP` environment variable is Python's rough equivalent of `.bashrc` for persisting interactive-session customizations.

---

## 21. Probable Exam Questions & Model Answers

**Q1. Define a prompt string. Why is it described as "dynamic"?**
> A prompt string is a template displayed by a shell/interpreter to indicate readiness for input; it is "dynamic" because it embeds escape sequences or expressions (username, time, directory, etc.) that are re-evaluated and substituted with current values every single time the prompt is shown, rather than remaining fixed text.

**Q2. List and briefly explain the four bash prompt variables.**
> PS1 (primary prompt, shown for every new command), PS2 (secondary/continuation prompt, shown for incomplete commands), PS3 (menu prompt, shown inside `select` loops), PS4 (trace prompt, shown before each line when `set -x` debugging is active).

**Q3. Write a bash command to set PS1 to display username, hostname, and current directory, followed by `$`.**
```bash
PS1='\u@\h:\w\$ '
```

**Q4. How do you make a custom PS1 permanent?**
> Add `export PS1="..."` to `~/.bashrc`, save the file, then run `source ~/.bashrc` (or open a new terminal) to apply it permanently across sessions.

**Q5. What is the difference between `\#` and `\!`?**
> `\#` shows the command number **within the current session only** (resets on new terminal); `\!` shows the command's position in the **persistent history file**, which keeps counting across sessions.

**Q6. Write a script demonstrating PS4 with `set -x` and explain the output.**
> *(See Section 15 — full worked example provided.)* The output prefixes each executed (and variable-expanded) line with the PS4 value, useful for tracing script execution step by step.

**Q7. How is a Python interactive prompt made dynamic, given that `sys.ps1` is normally a string?**
> By assigning an **object of a custom class** (instead of a plain string) to `sys.ps1`, where the class defines a `__str__` method. Python calls `str()` on `sys.ps1` every time it redraws the prompt, so overriding `__str__` allows fresh computation (e.g., current time or a counter) on each call.

**Q8. Why does `sys.ps1` have no effect when running `python script.py`?**
> Because `sys.ps1`/`sys.ps2` are only consulted by Python's **interactive REPL loop**. Running a file as a script executes it top-to-bottom without ever entering that interactive read-prompt-eval loop, so the prompt attributes are simply never read.

**Q9. What happens if you don't set PS3 before a `select` loop?**
> Bash uses its built-in default value, `#?`, as the selection prompt.

**Q10. Differentiate between a login shell and a non-login shell with respect to prompt configuration files.**
> A login shell (e.g., SSH login, `su -`) reads `/etc/profile` then `~/.bash_profile` (or fallback files); a non-login interactive shell (e.g., a new GUI terminal tab) reads `~/.bashrc`. This is why prompt customizations are often placed in `.bashrc`, or `.bash_profile` is made to explicitly source `.bashrc`.

---

## 22. Summary

Prompt strings are the dynamic, configurable text templates that command-line shells and interactive language environments display to indicate readiness for input. In **bash** and POSIX-compatible shells (dash, zsh, ksh, csh), four dedicated variables govern this behavior: **PS1** for standard commands, **PS2** for incomplete/continued commands, **PS3** for `select`-loop menus, and **PS4** for execution tracing during debugging. Bash enriches these variables with a rich set of **escape sequences** (`\u`, `\h`, `\w`, `\t`, `\d`, `\#`, `\$`, and more) that are substituted with live system values on every redraw, and supports ANSI color codes (wrapped in `\[ \]`) for visual customization. Temporary changes apply only to the current session; **permanent** changes require editing `~/.bashrc` (for non-login shells) or `~/.bash_profile` (for login shells) and reloading with `source`.

**Python**, by contrast, exposes only two prompt attributes — `sys.ps1` and `sys.ps2` — inside the `sys` module, used exclusively in **interactive mode**. Because these lack built-in escape-sequence support, dynamic behavior (such as a live clock or execution counter) must be engineered manually by assigning an **object with a custom `__str__` method**, which Python re-invokes every time the prompt is displayed.

Beyond bash and Python, the same underlying REPL concept — and the need for a readiness-signaling prompt — appears across many interactive tools including **Octave**, **Gnuplot**, and **Sage**, each offering their own (though generally simpler) customization mechanisms.

---

## 23. Final Cheat Sheet

### 🔹 Bash Prompt Variables
```
PS1  → Primary prompt              default: $   (or \u@\h:\w\$)
PS2  → Continuation prompt         default: >
PS3  → select-loop menu prompt     default: #?
PS4  → Debug trace prompt          default: +
```

### 🔹 Escape Sequences (Most Exam-Relevant)
```
\u  username            \h  hostname (short)       \H  hostname (full)
\w  full directory       \W  directory basename      \s  shell name
\t  time HH:MM:SS(24h)   \T  time HH:MM:SS(12h)       \A  time HH:MM(24h)
\@  time HH:MM am/pm     \d  date "Day Mon DD"        \D{fmt}  custom date
\#  session cmd number   \!  history cmd number       \$  '#' if root else '$'
\n  newline              \\  literal backslash        \[ \]  wrap non-printing chars
```

### 🔹 Key Commands
```bash
echo $PS1                          # view current PS1
PS1="custom> "                     # temporary change
nano ~/.bashrc                     # open permanent config file
export PS1="\u@\h:\w\$ "           # add inside .bashrc
source ~/.bashrc                   # reload without restart
set -x                             # start debug trace (uses PS4)
set +x                             # stop debug trace
select opt in "A" "B"; do ...; done  # menu loop (uses PS3)
```

### 🔹 Python Prompts
```python
import sys
sys.ps1                     # default '>>> '
sys.ps2                     # default '... '

sys.ps1 = "custom>>> "      # static custom prompt

class DynamicPrompt:
    def __str__(self):
        return "computed-each-time> "

sys.ps1 = DynamicPrompt()   # dynamic prompt (re-evaluated every call)
```

### 🔹 Quick Comparison
```
Bash:    4 variables (PS1-4) | escape sequences built-in | persists via .bashrc
Python:  2 variables (ps1,2) | no built-in escapes        | persists via PYTHONSTARTUP
         → dynamic behavior needs a class with __str__
```

### 🔹 Golden Rules for Exams
1. PS1 quotes should be **single quotes** to delay escape expansion.
2. Color codes need `\[ \]` wrapping — always.
3. `\#` ≠ `\!` — session count vs. history-file count.
4. `set -x` = trace ON, `set +x` = trace OFF (opposite of intuition).
5. Python prompt dynamism = object + `__str__`, **not** a plain string.
6. Login shell → `.bash_profile`; Non-login interactive shell → `.bashrc`.

---

*End of Notes — Prompt Strings (Expanded Professional Edition)*
