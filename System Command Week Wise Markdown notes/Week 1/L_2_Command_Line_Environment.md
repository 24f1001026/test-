# Week 1 Lecture 2 - Command Line Environment   

## Table of Contents
1. [Introduction to the Command Line](#1-introduction-to-the-command-line)
2. [Opening a Terminal Emulator](#2-opening-a-terminal-emulator)
3. [Understanding the Shell Prompt](#3-understanding-the-shell-prompt)
4. [First Commands: pwd, ls, ps, uname](#4-first-commands-pwd-ls-ps-uname)
5. [Clearing the Screen](#5-clearing-the-screen)
6. [Exiting the Shell](#6-exiting-the-shell)
7. [Anatomy of a Command: Command, Option, Argument](#7-anatomy-of-a-command-command-option-argument)
8. [The `man` Command and Manual Page Sections](#8-the-man-command-and-manual-page-sections)
9. [The Filesystem Hierarchy Standard (FHS)](#9-the-filesystem-hierarchy-standard-fhs)
10. [Traversing the Filesystem Tree](#10-traversing-the-filesystem-tree)
11. [Important FHS Directories Explained](#11-important-fhs-directories-explained)
12. [The `/usr` and `/var` Hierarchies](#12-the-usr-and-var-hierarchies)
13. [Sharable vs Static vs Variable Data](#13-sharable-vs-static-vs-variable-data)
14. [Summary](#14-summary)
15. [Practice Questions](#15-practice-questions)

---

## 1. Introduction to the Command Line

The **Command Line Interface (CLI)** is a text-based way of interacting with a computer's operating system. Instead of clicking icons like in a Graphical User Interface (GUI), you type **commands** into a program called a **shell**, and the shell interprets and executes them.

Why learn the command line?
- It gives you **precise control** over the system.
- Many tasks (automation, scripting, remote server management) are only possible or efficient via CLI.
- It is **lightweight** — no graphics rendering needed, so it works even over slow remote connections (like SSH).
- It is the foundation for **scripting and automation** (Bash scripts, DevOps tools, CI/CD pipelines, etc.)

### Key Terms
| Term | Meaning |
|------|---------|
| **Terminal** | A program that gives you a window to type commands (the "front end"). |
| **Shell** | A program that reads and interprets the commands you type (e.g., `bash`, `zsh`, `sh`). The terminal talks to the shell. |
| **Console** | Historically, the physical device attached to a computer for input/output; now often used loosely to mean terminal. |

---

## 2. Opening a Terminal Emulator

A **terminal emulator** is a software application that emulates (imitates) old-school hardware terminals, allowing you to access a shell on modern systems.

### Common Terminal Emulators
- **Terminal** (macOS default)
- **Konsole** (KDE desktop environment)
- **xterm** (a minimal, classic X Window terminal)
- **guake** (a drop-down terminal for Linux, similar to games' console)
- **GNOME Terminal**, **iTerm2**, **Alacritty**, **Windows Terminal**, etc.

### How to Open
- On Linux: Look for "Terminal" in the applications menu, or use a shortcut like `Ctrl+Alt+T`.
- On macOS: Open **Spotlight** (`Cmd+Space`), type "Terminal", hit Enter.
- On Windows: Use **Windows Terminal**, **PowerShell**, or **WSL (Windows Subsystem for Linux)** for a Linux-like experience.

---

## 3. Understanding the Shell Prompt

Once a terminal is open, the shell displays a **prompt** — a line of text that tells you the shell is ready to accept a command.

### Syntax / Structure
```
username@hostname:path$
```

Example:
```
user@machine:~$
```

### Explanation of Each Part
| Part | Meaning |
|------|---------|
| `user` | The **username** of the currently logged-in user. |
| `machine` | The **hostname** — the name of the computer on the network. |
| `~` | The **path** — current working directory. `~` is shorthand for the home directory. |
| `$` | Indicates you are logged in as a **normal (non-root) user**. A `#` symbol instead of `$` means you are logged in as **root** (the superuser/administrator). |

### Why This Matters
The prompt gives you situational awareness at a glance:
- Who am I?
- What machine am I on?
- Where am I in the filesystem?
- What privilege level do I have?

---

## 4. First Commands: pwd, ls, ps, uname

These four commands are typically the *very first* commands a beginner learns because they answer: "Where am I?", "What's here?", "What's running?", and "What system am I on?"

---

### 4.1 `pwd` — Print Working Directory

**Concept:** Displays the **absolute path** of the directory you are currently "standing in" within the filesystem tree.

**Syntax:**
```
pwd [OPTION]
```

**Example:**
```bash
user@machine:~$ pwd
/home/user
```

**Explanation:** Since the filesystem is a tree structure starting at `/` (root), and you are always "inside" some directory when using the shell, `pwd` tells you exactly which directory that is, using the full (absolute) path from root.

---

### 4.2 `ls` — List Directory Contents

**Concept:** Lists the files and directories present in a given location (default: current directory).

**Syntax:**
```
ls [OPTION]... [FILE]...
```

**Common Options:**
| Option | Meaning |
|--------|---------|
| `-a` | Show **all** files, including hidden files (those starting with `.`) |
| `-l` | **Long listing** format — shows permissions, owner, size, modification date |
| `-h` | **Human-readable** file sizes (used with `-l`) |
| `-R` | Recursive listing (lists subdirectories too) |
| `-t` | Sort by modification time |

**Examples:**
```bash
user@machine:~$ ls
Desktop  Documents  Downloads  Pictures  MyCode

user@machine:~$ ls -a
.  ..  .bashrc  .config  Desktop  Documents  Downloads  Pictures  MyCode

user@machine:~$ ls -l
drwxr-xr-x 2 user user 4096 Jan 10 09:00 Desktop
```

**Explanation:**
- Files/directories beginning with `.` (like `.bashrc`) are **hidden files** — mostly configuration files. `ls` alone will *not* show them; you need `-a`.
- `.` refers to the **current directory itself**, and `..` refers to the **parent directory** — both appear when you run `ls -a`.

---

### 4.3 `ps` — Process Status

**Concept:** Displays information about currently **running processes** (programs that are executing).

**Syntax:**
```
ps [OPTION]
```

**Common Options:**
| Option | Meaning |
|--------|---------|
| `ps` (no option) | Shows processes running in the *current terminal session* |
| `ps -e` or `ps -A` | Shows **all** processes running on the system |
| `ps aux` | Shows all processes, for all users, in detailed format |
| `ps -f` | Full-format listing |

**Example:**
```bash
user@machine:~$ ps
  PID TTY          TIME CMD
 1234 pts/0    00:00:00 bash
 5678 pts/0    00:00:00 ps
```

**Explanation of Output Columns:**
| Column | Meaning |
|--------|---------|
| `PID` | Process ID — a unique number identifying the process |
| `TTY` | The terminal associated with the process |
| `TIME` | Total CPU time used by the process |
| `CMD` | The command/program name that started the process |

---

### 4.4 `uname` — Print System Information

**Concept:** Displays information about the operating system and hardware.

**Syntax:**
```
uname [OPTION]
```

**Common Options:**
| Option | Meaning |
|--------|---------|
| `-a` | **All** information |
| `-s` | Kernel name |
| `-r` | Kernel release version |
| `-n` | Network hostname |
| `-m` | Machine hardware name (e.g., x86_64) |

**Example:**
```bash
user@machine:~$ uname -a
Linux machine 5.15.0-91-generic #101-Ubuntu SMP x86_64 GNU/Linux
```

---

## 5. Clearing the Screen

### Concept
Over time, your terminal fills up with old command output, making it visually cluttered. **Clearing** removes this visible history (note: it does *not* delete your command history, only what's displayed).

### Syntax / Methods
```bash
clear
```
OR the keyboard shortcut:
```
Ctrl + L
```

**Explanation:** Both achieve the same visual result — a fresh, empty terminal screen — but `Ctrl+L` is faster since it doesn't require typing and pressing Enter.

---

## 6. Exiting the Shell

### Concept
Closes the current shell session, effectively "logging out" of that terminal instance.

### Syntax / Methods
```bash
exit
```
OR the keyboard shortcut:
```
Ctrl + D
```

**Explanation:**
- `exit` is a **built-in shell command** that terminates the shell process.
- `Ctrl+D` sends an **EOF (End Of File)** signal to the shell, which — when the input line is empty — also causes it to exit.
- If you are inside a nested shell (e.g., you typed `bash` inside a shell), `exit` only closes the innermost shell, returning you to the outer one.

---

## 7. Anatomy of a Command: Command, Option, Argument

### Concept
Every command line instruction generally follows this structure:

```
command [options] [arguments]
```

| Component | Description |
|-----------|--------------|
| **Command** | The name of the program/utility to run (e.g., `ls`, `man`) |
| **Option** (a.k.a. "flag" or "switch") | Modifies the *behavior* of the command. Usually prefixed with `-` (short form, e.g., `-a`) or `--` (long form, e.g., `--all`) |
| **Argument** | The *target* the command should act upon — often a filename, directory, or search term |

### Example
```bash
user@machine:~$ man 1 ls
```
| Part | Role |
|------|------|
| `man` | Command |
| `1` | Option (specifies which manual **section** to search) |
| `ls` | Argument (the topic/command to look up) |

Another example:
```bash
ls -l -a /home/user
```
- `ls` → command
- `-l -a` (or combined: `-la`) → options
- `/home/user` → argument

---

## 8. The `man` Command and Manual Page Sections

### Concept
`man` (short for **manual**) displays the official documentation ("man pages") for commands, system calls, library functions, file formats, and more, directly in the terminal.

### Syntax
```
man [SECTION] PAGE_NAME
```

### Example
```bash
user@machine:~$ man ls
user@machine:~$ man 1 ls
```

**Explanation:** Simply typing `man ls` searches through sections in a default order and shows the first match — usually Section 1. Explicitly specifying `man 1 ls` forces it to show Section 1 specifically, which is useful when a name exists in multiple sections (e.g., `printf` is both a shell command **and** a C library function).

### Manual Page Sections Table
| Section | Type of Pages |
|---------|----------------|
| 1 | Executable programs or shell commands |
| 2 | System calls provided by the kernel |
| 3 | Library calls (functions within program libraries) |
| 4 | Special files, usually found in `/dev` |
| 5 | File formats and conventions (e.g., `/etc/passwd` format) |
| 6 | Games |
| 7 | Miscellaneous (macro packages, conventions, protocols) |
| 8 | System administration commands (usually require root) |
| 9 | Kernel routines (non-standard, kernel internals) |

### Navigating `man` Pages
| Key | Action |
|-----|--------|
| `Space` / `Page Down` | Scroll down a page |
| `b` / `Page Up` | Scroll up a page |
| `/pattern` | Search forward for "pattern" |
| `n` | Go to next search match |
| `q` | Quit the man page |

---

## 9. The Filesystem Hierarchy Standard (FHS)

### Concept
The **Filesystem Hierarchy Standard (FHS)** is a specification that defines the directory structure and directory contents used in Unix-like operating systems (Linux distributions, in particular). It ensures consistency, so software and users can predict *where* certain types of files should live.

- **Current widely referenced version:** FHS 3.0, released June 3, 2015
- **Maintained by:** The Linux Foundation
- **Reference:** https://refspecs.linuxfoundation.org/fhs.shtml

### Why It Matters
Without a standard, every Linux distribution could organize files completely differently, making it very hard to:
- Write portable software
- Know where to find configuration files, logs, or binaries
- Administer systems consistently across distros

---

## 10. Traversing the Filesystem Tree

### Concept
The Linux filesystem is organized as an inverted **tree structure**, starting from a single root.

### Key Symbols
| Symbol | Meaning |
|--------|---------|
| `/` | The **root** of the entire filesystem — the top-most directory. Everything branches from here. |
| `/` (also) | Acts as the **delimiter** (separator) between directory names in a path, e.g., `/home/gphani/Desktop` |
| `.` | Refers to the **current directory** |
| `..` | Refers to the **parent directory** (one level up) |

### Absolute vs. Relative Paths

**Absolute Path:** Always starts from `/` (root), fully specifying the location regardless of where you currently are.
```bash
/home/gphani/Desktop
```

**Relative Path:** Specified relative to your **current working directory**.
```bash
# If you're in /home/gphani
cd Desktop        # relative path
cd ../gphani/Desktop   # relative, using parent reference
```

### Example Tree (from the lecture)
```
/
├── bin
└── home
    └── gphani
        ├── Desktop
        ├── Documents
        ├── Downloads
        ├── Pictures
        └── MyCode
```

**Explanation:** Notice that `home/gphani` is a *user's personal directory*, and everything below it (Desktop, Documents, etc.) is a **subdirectory** of `gphani`, which itself is a subdirectory of `home`, which is a subdirectory of `/`.

### Extended Reference Tree (not from the lecture slide — for context only)
The slide only illustrates `/bin` and `/home`. In a real system, root also contains the other top-level directories covered in Section 11 (`/etc`, `/usr`, `/var`, `/tmp`, `/lib`, `/mnt`, `/media`, `/sbin`, etc.). A fuller picture looks like this:
```
/
├── bin
├── boot
├── dev
├── etc
├── home
│   └── gphani
│       ├── Desktop
│       ├── Documents
│       ├── Downloads
│       ├── Pictures
│       └── MyCode
├── lib
├── media
├── mnt
├── opt
├── run
├── sbin
├── srv
├── tmp
├── usr
│   ├── bin
│   ├── sbin
│   ├── share
│   ├── local
│   ├── include
│   └── src
└── var
    ├── cache
    ├── lib
    ├── local
    ├── lock
    ├── log
    ├── run
    └── tmp
```

---

## 11. Important FHS Directories Explained

| Directory | Purpose |
|-----------|---------|
| `/bin` | Essential command binaries needed for basic system functioning (available to all users, even in single-user/recovery mode) |
| `/boot` | Static files needed by the **boot loader** (e.g., kernel images, GRUB config) |
| `/dev` | **Device files** — special files representing hardware devices (e.g., `/dev/sda` for a disk) |
| `/etc` | **Host-specific system configuration** files (e.g., network settings, user account info) |
| `/lib` | Essential **shared libraries** and kernel modules needed by binaries in `/bin` and `/sbin` |
| `/media` | Mount points for **removable media** (USB drives, CDs, etc.) |
| `/mnt` | Mount points for **temporarily mounted filesystems** (used manually by the admin) |
| `/opt` | **Add-on application software packages** (third-party software not part of the default OS) |
| `/run` | Data relevant to running processes since the last boot (runtime data) |
| `/sbin` | **Essential system binaries** — typically administrative commands (e.g., `reboot`, `fdisk`) |
| `/srv` | Data for **services** provided by the system (e.g., web server files) |
| `/tmp` | **Temporary files** — often cleared on reboot |
| `/usr` | **Secondary hierarchy** — contains the majority of user utilities and applications (read-only, shareable data) |
| `/var` | **Variable data** — files whose content is expected to grow/change (logs, mail, caches) |

### Memory Tip
Think of it this way:
- `/bin`, `/sbin`, `/lib` → **Essential** stuff needed to boot and repair the system
- `/etc` → **Configuration** ("et cetera" — miscellaneous settings)
- `/home` → **Personal** user data
- `/var` → Data that **varies**/changes constantly (logs, mail)
- `/usr` → **User-installed software and secondary programs** (like a "read-only" library of apps)
- `/tmp` → **Temporary**, disposable
- `/mnt`, `/media` → **Mounting** external storage

---

## 12. The `/usr` and `/var` Hierarchies

### `/usr` Hierarchy (Secondary Hierarchy)
| Directory | Purpose |
|-----------|---------|
| `/usr/bin` | User commands (the majority of user-facing programs) |
| `/usr/lib` | Libraries used by programs in `/usr/bin` and `/usr/sbin` |
| `/usr/local` | Local hierarchy — for software installed manually by the local admin, separate from distro-managed packages |
| `/usr/sbin` | Non-vital system administration binaries |
| `/usr/share` | Architecture-**independent** shared data (documentation, icons, etc.) |
| `/usr/include` | Header files (`.h` files) included by C programs during compilation |
| `/usr/src` | Source code (e.g., kernel source) |

### `/var` Hierarchy (Variable Data)
| Directory | Purpose |
|-----------|---------|
| `/var/cache` | Application **cache data** (data stored temporarily to speed up operations) |
| `/var/lib` | **Variable state information** — data that programs modify while running (e.g., databases) |
| `/var/local` | Variable data specifically for programs installed in `/usr/local` |
| `/var/lock` | **Lock files** — used to prevent multiple processes from using the same resource simultaneously |
| `/var/log` | **Log files** and directories — system and application logs |
| `/var/run` | Data relevant to currently running processes (often now a symlink to `/run`) |
| `/var/tmp` | Temporary files that are **preserved between reboots** (unlike `/tmp`) |

---

## 13. Sharable vs Static vs Variable Data

### Concept
The FHS also classifies directories along **two axes**: 
1. **Shareable vs. Unshareable** — can this data be shared across multiple machines on a network (e.g., via NFS)?
2. **Static vs. Variable** — does the data change during normal system operation, or does it stay the same until deliberately modified/upgraded?

### Classification Table
| | **Shareable** | **Unshareable** |
|---|---|---|
| **Static** | `/usr`, `/opt` | `/etc`, `/boot` |
| **Variable** | `/var/mail`, (spool data) | `/var/run`, `/var/lock` |

### Explanation
- **Static + Shareable** (`/usr`, `/opt`): These contain program binaries/libraries that don't change often and can be safely shared/mounted read-only across multiple computers on a network.
- **Static + Unshareable** (`/etc`, `/boot`): Configuration and boot files are static but are specific to *that particular machine* — you wouldn't want another machine to use your `/etc`.
- **Variable + Shareable** (`/var/mail`): Mail data changes constantly but can still be centrally shared (e.g., a mail server shared across a network).
- **Variable + Unshareable** (`/var/run`, `/var/lock`): This is runtime-specific data tied to processes running on *that specific machine right now* — sharing it across machines wouldn't make sense.

---

## 14. Summary

- The **command line** is accessed via a **terminal emulator** running a **shell**.
- The **prompt** (`user@machine:~$`) tells you your username, hostname, and current directory.
- `pwd`, `ls`, `ps`, and `uname` are foundational commands for orientation: where am I, what's here, what's running, what system is this.
- `clear`/`Ctrl+L` cleans the screen; `exit`/`Ctrl+D` closes the shell.
- Every command generally follows: `command [options] [arguments]`.
- `man` gives you built-in documentation, organized into **9 sections** by content type.
- The **Filesystem Hierarchy Standard (FHS)** defines a predictable directory layout starting from `/` (root).
- Paths can be **absolute** (from `/`) or **relative** (from current directory), using `.` and `..` for navigation.
- Directories are further classified as **static/variable** and **shareable/unshareable**, which explains *why* the structure is designed the way it is.

---

## 15. Practice Questions

### A. Conceptual Questions
1. What is the difference between a **terminal emulator** and a **shell**?
2. In the prompt `user@machine:~$`, what does the `~` represent, and what would it mean if this were replaced with `/var/log`?
3. What is the difference between `$` and `#` at the end of a shell prompt?
4. Why does the Filesystem Hierarchy Standard (FHS) exist? What problems would arise without it?
5. Explain the difference between an **absolute path** and a **relative path**, with one example each.
6. What do `.` and `..` represent when navigating directories?

### B. Command-Specific Questions
7. Which command would you use to find your current working directory? Write its syntax.
8. What is the difference between running `ls` and `ls -a`? Give an example of output difference.
9. What does the `-l` option do when combined with `ls`? Name at least three pieces of information shown in long listing format.
10. Which command shows currently running processes? What does the `PID` column represent?
11. What is the difference between `ps` (no options) and `ps -e`?
12. Which command would tell you the kernel version of your operating system? Provide the exact syntax.
13. What are two ways to clear your terminal screen?
14. What are two ways to exit a shell session?

### C. `man` Pages
15. What does `man 5 passwd` most likely show you, based on the man page sections table?
16. Why might you need to specify a section number (like `man 1 printf` vs `man 3 printf`)?
17. List any three man page sections and describe what type of content each contains.

### D. Filesystem Hierarchy
18. Where would you expect to find system log files? Give the full path.
19. Where are **essential** command binaries stored that are needed even in single-user mode?
20. What is the difference between `/tmp` and `/var/tmp`?
21. What is stored in `/etc`, and why is this directory classified as "static and unshareable"?
22. What is the purpose of `/opt`, and how is it different from `/usr`?
23. Where would user-installed, locally compiled software typically be placed (as opposed to distro-managed packages)?
24. Classify the following directories into the Static/Variable and Shareable/Unshareable matrix: `/boot`, `/var/lock`, `/usr`, `/var/mail`.

### E. Applied / Scenario-Based
25. You are told a program crashed and you need to check logs. Which directory should you check first, and why?
26. You want to check if a USB drive got mounted. Which directory would most likely contain the mount point?
27. You've written a command `ls -la /home/gphani` — identify the command, the options, and the argument.
28. A colleague says "cd .." moved them up one directory. Explain what `..` means and why this works.
29. If your current directory is `/home/gphani/Documents`, write the **absolute path** and a **relative path** to reach `/home/gphani/Pictures`.
30. Why is it good practice to use `man` before running an unfamiliar command with unfamiliar options?

---

*End of Lecture Notes — Command Line Environment*
