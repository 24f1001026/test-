# Simple Commands in Linux – Part 2
### Lecture 4 | Week 1

---

## Table of Contents

1. [Multiple Uses of `ls` — `/` Is as Good as One](#1-multiple-uses-of-ls--is-as-good-as-one)
   - [1.1 The Root Folder `/` Is Its Own Parent](#11-the-root-folder--is-its-own-parent)
   - [1.2 Short and Long Forms of Options](#12-short-and-long-forms-of-options)
   - [1.3 Interpretation of Directory as an Argument](#13-interpretation-of-directory-as-an-argument)
   - [1.4 Recursive Listing](#14-recursive-listing)
   - [1.5 Order of Options on the Command Line](#15-order-of-options-on-the-command-line)
2. [Knowing Files Better](#2-knowing-files-better)
   - [2.1 `cat`](#21-cat)
   - [2.2 `more`](#22-more)
   - [2.3 `less`](#23-less)
   - [2.4 `head`](#24-head)
   - [2.5 `tail`](#25-tail)
   - [2.6 `wc`](#26-wc)
3. [Knowing More Commands](#3-knowing-more-commands)
   - [3.1 `man`](#31-man)
   - [3.2 `whatis`](#32-whatis)
   - [3.3 `apropos`](#33-apropos)
   - [3.4 `which`](#34-which)
   - [3.5 `type`](#35-type)
   - [3.6 `help`](#36-help)
   - [3.7 `info`](#37-info)
4. [Multiple Arguments](#4-multiple-arguments)
   - [4.1 Second Argument](#41-second-argument)
   - [4.2 Interpretation of the Last Argument](#42-interpretation-of-the-last-argument)
   - [4.3 Recursion Assumed for `mv`, Not for `cp`](#43-recursion-assumed-for-mv-not-for-cp)
5. [Links](#5-links)
   - [5.1 Hard Links](#51-hard-links)
   - [5.2 Symbolic (Soft) Links](#52-symbolic-soft-links)
6. [File Sizes](#6-file-sizes)
   - [6.1 `ls -s`](#61-ls--s)
   - [6.2 `stat`](#62-stat)
   - [6.3 `du`](#63-du)
   - [6.4 Role of Block Size](#64-role-of-block-size)
7. [In-Memory Filesystems](#7-in-memory-filesystems)
   - [7.1 `/proc`](#71-proc)
   - [7.2 `/sys`](#72-sys)
8. [Summary](#8-summary)

---

## 1. Multiple Uses of `ls` — `/` Is as Good as One

### 1.1 The Root Folder `/` Is Its Own Parent

**Concept:**
In the Linux filesystem hierarchy, `/` (root) is the topmost directory. It has no parent directory above it. When you check the parent of `/` (using `..`), Linux resolves it back to `/` itself — meaning `/` is its own parent. Also, using multiple consecutive slashes (`//`, `///`) in a path is treated exactly the same as a single slash `/`.

**Syntax:**
```bash
ls /
ls //
ls ///home
cd /..
```

**Examples:**
```bash
$ ls /..
# Shows the same content as ls / — because /.. resolves to / itself

$ ls //home///user
# Works exactly like: ls /home/user
# Multiple slashes are collapsed and treated as one

$ cd /../../..
$ pwd
/
# No matter how many times you go "up" from root, you stay at root
```

**Key Point for Exam:** Extra slashes (`/`) anywhere in a path (except as a separator) are ignored by the shell/kernel — "multiple uses of `/` is as good as one."

---

### 1.2 Short and Long Forms of Options

**Concept:**
Most Linux commands accept options in two forms:
- **Short form**: single dash `-` followed by a single letter (can often be combined, e.g., `-la`)
- **Long form**: double dash `--` followed by a full word (more readable, self-explanatory)

Both forms usually do the same thing.

**Syntax:**
```bash
command -X          # short form
command --XXXX       # long form
command -abc         # combined short options
```

**Examples:**
```bash
$ ls -a
$ ls --all
# Both show hidden files (files starting with .)

$ ls -l
$ ls --format=long
# Both show long listing format

$ ls -la
# Combined short options: 'a' (all) + 'l' (long listing)

$ ls -l -a -h
$ ls --format=long --all --human-readable
# Same result, short vs long form
```

**Key Point for Exam:** Short options can be clubbed together after a single `-` (e.g., `-la` = `-l -a`), but long options **cannot** be clubbed — each needs its own `--`.

---

### 1.3 Interpretation of Directory as an Argument

**Concept:**
When a **directory name** is passed as an argument to `ls`, the command lists the **contents inside** that directory (not the directory entry itself). If you want to see the directory itself as an entry (like a file), you use the `-d` option.

**Syntax:**
```bash
ls directory_name
ls -d directory_name
```

**Examples:**
```bash
$ ls Documents
file1.txt  file2.txt  notes.md
# Lists everything INSIDE Documents

$ ls -d Documents
Documents
# Lists the directory entry itself, not its contents

$ ls -d */
# Lists only directories in the current location
```

---

### 1.4 Recursive Listing

**Concept:**
The `-R` (capital R) option lists the contents of a directory **and all its subdirectories**, going down the entire directory tree.

**Syntax:**
```bash
ls -R directory_name
ls -R
```

**Examples:**
```bash
$ ls -R Projects
Projects:
app  docs  README.md

Projects/app:
main.py  utils.py

Projects/docs:
guide.txt
# Shows Projects, then descends into each subfolder automatically

$ ls -Ra
# Recursive + show hidden files
```

**Key Point for Exam:** `-r` (lowercase) means **reverse order** in `ls`, while `-R` (uppercase) means **recursive**. Don't confuse the two — a very common exam trap.

---

### 1.5 Order of Options on the Command Line

**Concept:**
In most Linux commands, the **order of options does not matter** — `ls -l -a` and `ls -a -l` produce the same output. However, the order of **non-option arguments** (like filenames) can matter for some commands (especially for commands like `cp` and `mv`, discussed later).

**Syntax:**
```bash
ls -l -a
ls -a -l
ls -la
ls -al
```

**Examples:**
```bash
$ ls -l -a /home
$ ls -a -l /home
# Identical output — option order is irrelevant here

$ ls -la
$ ls -al
# Same result
```

**Key Point for Exam:** Options are order-independent; arguments (operands) are often order-dependent.

---

## 2. Knowing Files Better

### 2.1 `cat`

**Concept:**
`cat` (concatenate) displays the entire content of one or more files at once, prints it directly to the terminal, and can also be used to combine multiple files or create new files.

**Syntax:**
```bash
cat filename
cat file1 file2
cat file1 file2 > combined.txt
cat > newfile.txt
cat -n filename
```

**Examples:**
```bash
$ cat notes.txt
This is line 1
This is line 2

$ cat file1.txt file2.txt
# Prints contents of both files one after another

$ cat file1.txt file2.txt > merged.txt
# Combines both files into merged.txt

$ cat -n notes.txt
     1  This is line 1
     2  This is line 2
# Displays with line numbers
```

**Limitation:** Not suitable for large files since it dumps everything at once without pausing.

---

### 2.2 `more`

**Concept:**
`more` is a simple **pager** that displays file content one screen (page) at a time. You can scroll **forward only** (in the classic version). Useful for large files where `cat` would flood the screen.

**Syntax:**
```bash
more filename
```

**Examples:**
```bash
$ more longfile.txt
# Displays first screenful; press SPACE for next page, ENTER for next line, q to quit
```

**Common keys inside `more`:**
| Key | Action |
|---|---|
| `Space` | Next page |
| `Enter` | Next line |
| `q` | Quit |

---

### 2.3 `less`

**Concept:**
`less` is an improved pager compared to `more`. It allows **both forward and backward** scrolling, searching within the file, and does not need to load the entire file into memory first — making it faster for very large files. (Famous phrase: "less is more".)

**Syntax:**
```bash
less filename
```

**Examples:**
```bash
$ less bigfile.log
# Opens file for viewing

# Inside less:
# Space / f     -> forward one page
# b             -> backward one page
# /pattern      -> search forward for 'pattern'
# ?pattern      -> search backward for 'pattern'
# n             -> repeat last search
# q             -> quit
```

**Key Point for Exam:** `less` supports backward navigation and search; `more` (classic) does not — this is the main functional difference tested in exams.

---

### 2.4 `head`

**Concept:**
`head` displays the **beginning** (first lines) of a file. By default, it shows the first **10 lines**.

**Syntax:**
```bash
head filename
head -n N filename      # first N lines
head -N filename        # short form
head -c N filename      # first N bytes
```

**Examples:**
```bash
$ head file.txt
# Shows first 10 lines by default

$ head -n 5 file.txt
$ head -5 file.txt
# Shows first 5 lines

$ head -c 100 file.txt
# Shows first 100 bytes

$ head file1.txt file2.txt
# Shows first 10 lines of EACH file, with filename headers
```

---

### 2.5 `tail`

**Concept:**
`tail` displays the **end** (last lines) of a file. By default, it shows the last **10 lines**. It is very useful for monitoring log files as they grow, using the `-f` (follow) option.

**Syntax:**
```bash
tail filename
tail -n N filename      # last N lines
tail -N filename        # short form
tail -f filename        # follow (live updates)
```

**Examples:**
```bash
$ tail file.txt
# Shows last 10 lines by default

$ tail -n 20 file.txt
$ tail -20 file.txt
# Shows last 20 lines

$ tail -f /var/log/syslog
# Continuously displays new lines as they are appended (real-time monitoring)
```

**Key Point for Exam:** `head` and `tail` behave symmetrically — `head` = top of file, `tail` = bottom of file, both default to 10 lines.

---

### 2.6 `wc`

**Concept:**
`wc` (word count) counts the number of **lines, words, and bytes/characters** in a file.

**Syntax:**
```bash
wc filename
wc -l filename     # lines only
wc -w filename     # words only
wc -c filename     # bytes only
wc -m filename     # characters only
```

**Examples:**
```bash
$ wc notes.txt
  12  50  320 notes.txt
# Output format: lines  words  bytes  filename

$ wc -l notes.txt
12 notes.txt
# Only line count

$ wc -w notes.txt
50 notes.txt
# Only word count

$ cat notes.txt | wc -l
# Counting lines from piped input (no filename shown)
```

---

## 3. Knowing More Commands

### 3.1 `man`

**Concept:**
`man` (manual) displays the **full official documentation** (manual pages) for a command, including its description, syntax, all options, and related commands (SEE ALSO section).

**Syntax:**
```bash
man command_name
man -k keyword       # same as apropos
man section_number command_name
```

**Examples:**
```bash
$ man ls
# Opens full manual page for ls (use q to quit, / to search)

$ man 5 passwd
# Opens manual page from Section 5 (file formats) for passwd
```

**Manual Sections (good to remember):**
| Section | Content |
|---|---|
| 1 | User commands |
| 2 | System calls |
| 3 | Library functions |
| 5 | File formats |
| 8 | Admin/system commands |

---

### 3.2 `whatis`

**Concept:**
`whatis` gives a **one-line summary/description** of a command — much shorter than `man`.

**Syntax:**
```bash
whatis command_name
```

**Examples:**
```bash
$ whatis ls
ls (1)  - list directory contents

$ whatis cat
cat (1) - concatenate files and print on the standard output
```

---

### 3.3 `apropos`

**Concept:**
`apropos` searches the manual page **names and short descriptions** for a given keyword and lists all matching commands. Useful when you know what you want to do but not the exact command name.

**Syntax:**
```bash
apropos keyword
```

**Examples:**
```bash
$ apropos copy
cp (1)     - copy files and directories
cpio (1)   - copy files to and from archives

$ apropos "list directory"
ls (1)     - list directory contents
```

**Key Point for Exam:** `whatis` needs the **exact command name**; `apropos` works with a **keyword/topic** and can return multiple results.

---

### 3.4 `which`

**Concept:**
`which` shows the **full path of the executable file** that would run when you type a command name, by searching directories listed in the `$PATH` variable.

**Syntax:**
```bash
which command_name
which -a command_name    # show all matches in PATH
```

**Examples:**
```bash
$ which ls
/usr/bin/ls

$ which python3
/usr/bin/python3

$ which -a python
/usr/bin/python
/usr/local/bin/python
# Shows every matching location in PATH
```

---

### 3.5 `type`

**Concept:**
`type` tells you **how a name would be interpreted** if used as a command — whether it's a shell built-in, an alias, a function, or an external executable file.

**Syntax:**
```bash
type command_name
```

**Examples:**
```bash
$ type cd
cd is a shell builtin

$ type ls
ls is /usr/bin/ls

$ type type
type is a shell builtin
```

**Key Point for Exam:** `which` only finds external executables in `$PATH`; `type` can also detect **built-ins, aliases, and functions** — so `type` is more general than `which`.

---

### 3.6 `help`

**Concept:**
`help` displays information specifically about **shell built-in commands** (commands that are part of the shell itself, like `cd`, `echo`, `pwd`, `alias`), not external programs.

**Syntax:**
```bash
help
help command_name
```

**Examples:**
```bash
$ help cd
cd: cd [-L|[-P [-e]] [-@]] [dir]
    Change the shell working directory...

$ help
# Lists all available shell built-in commands
```

**Key Point for Exam:** `help` works only for **built-ins**; for external commands use `man` instead (e.g., `help ls` will fail/give no useful info since `ls` is an external program).

---

### 3.7 `info`

**Concept:**
`info` provides documentation in a **hyperlinked, menu-driven** format (GNU Info system) — often more detailed and structured than `man` pages, with the ability to navigate between linked topics/nodes.

**Syntax:**
```bash
info command_name
```

**Examples:**
```bash
$ info ls
# Opens an interactive, hyperlinked documentation browser
# Navigate with arrow keys, Enter to follow links, q to quit
```

---

## 4. Multiple Arguments

### 4.1 Second Argument

**Concept:**
Many commands (like `cp`, `mv`) take **at least two arguments**: a source and a destination. The **second argument** typically specifies the **destination** — where the file/content should go.

**Syntax:**
```bash
cp source destination
mv source destination
```

**Examples:**
```bash
$ cp file1.txt file2.txt
# file1.txt content is copied into file2.txt (destination = second argument)

$ mv report.txt /home/user/Documents/
# report.txt is moved to the Documents folder
```

---

### 4.2 Interpretation of the Last Argument

**Concept:**
When a command like `cp` or `mv` is given **more than two arguments**, the **last argument** is always treated as the **destination**, and it must be a **directory** (since multiple source files cannot be merged into a single file).

**Syntax:**
```bash
cp file1 file2 file3 destination_directory
mv file1 file2 file3 destination_directory
```

**Examples:**
```bash
$ cp a.txt b.txt c.txt /home/user/backup/
# a.txt, b.txt, c.txt are all copied INTO the backup/ directory
# backup/ (the LAST argument) must be an existing directory

$ mv *.log /var/log/archive/
# All .log files moved into archive/ (last argument = destination directory)
```

**Key Point for Exam:** If the last argument is not an existing directory (and there are 3+ arguments), the command will throw an error — e.g., `cp: target 'X' is not a directory`.

---

### 4.3 Recursion Assumed for `mv`, Not for `cp`

**Concept:**
- `mv` moves directories **without needing any special option** — recursion is implicit/assumed because moving a directory doesn't require copying data blocks, just changing directory entries.
- `cp`, however, requires the explicit `-r` (or `-R`) option to copy a directory recursively, because copying involves actually duplicating all data.

**Syntax:**
```bash
mv directory destination          # works directly
cp -r directory destination       # -r required for directories
```

**Examples:**
```bash
$ mv ProjectFolder /home/user/Backup/
# Works fine — no -r needed, even though ProjectFolder has subfolders/files

$ cp ProjectFolder /home/user/Backup/
cp: -r not specified; omitting directory 'ProjectFolder'
# FAILS without -r

$ cp -r ProjectFolder /home/user/Backup/
# Works correctly — recursively copies all contents
```

**Key Point for Exam:** This is a classic exam question — remember: **"mv doesn't need -r, cp does."**

---

## 5. Links

### 5.1 Hard Links

**Concept:**
A **hard link** is an additional directory entry that points to the **same inode** (same physical data on disk) as the original file. Both the original file and the hard link are indistinguishable — they are equally "real." Deleting one does not delete the data as long as at least one link (name) still exists. Hard links:
- Cannot link to directories
- Cannot cross filesystem/partition boundaries
- Share the same inode number as the original

**Syntax:**
```bash
ln target_file link_name
```

**Examples:**
```bash
$ ln original.txt hardlink.txt

$ ls -li original.txt hardlink.txt
1234567 -rw-r--r-- 2 user user 100 Aug 19 10:00 original.txt
1234567 -rw-r--r-- 2 user user 100 Aug 19 10:00 hardlink.txt
# Same inode number (1234567) and link count = 2

$ rm original.txt
$ cat hardlink.txt
# Still works! Data is intact because hardlink.txt still references the inode
```

---

### 5.2 Symbolic (Soft) Links

**Concept:**
A **symbolic link (symlink)** is a special type of file that simply **stores the path** to another file or directory, rather than pointing to the same inode. It's like a "shortcut." If the original file is deleted or moved, the symlink becomes **broken/dangling**. Symbolic links:
- Can link to directories
- Can cross filesystem/partition boundaries
- Have their own separate inode

**Syntax:**
```bash
ln -s target_file link_name
```

**Examples:**
```bash
$ ln -s /home/user/original.txt softlink.txt

$ ls -l softlink.txt
lrwxrwxrwx 1 user user 25 Aug 19 10:05 softlink.txt -> /home/user/original.txt
# Note the 'l' at start and the '->' pointing to target

$ rm original.txt
$ cat softlink.txt
cat: softlink.txt: No such file or directory
# Symlink is now broken/dangling
```

**Key Point for Exam — Hard Link vs Symbolic Link:**

| Feature | Hard Link | Symbolic Link |
|---|---|---|
| Points to | Same inode (data) | Path/name of target |
| Works across filesystems | No | Yes |
| Can link directories | No | Yes |
| Broken if original deleted | No | Yes (becomes dangling) |
| Command | `ln target link` | `ln -s target link` |
| File type shown by `ls -l` | Normal `-` | `l` (with `->` arrow) |

---

## 6. File Sizes

### 6.1 `ls -s`

**Concept:**
The `-s` option with `ls` displays the **size (in blocks)** allocated to each file, printed just before the filename.

**Syntax:**
```bash
ls -s
ls -sh          # human-readable sizes
```

**Examples:**
```bash
$ ls -s
total 24
4 file1.txt  8 file2.txt  12 file3.txt
# Numbers represent block-size allocation, not exact byte size

$ ls -sh
total 24K
4.0K file1.txt  8.0K file2.txt  12K file3.txt
```

---

### 6.2 `stat`

**Concept:**
`stat` displays **detailed metadata** about a file: exact size in bytes, number of blocks, inode number, permissions, number of hard links, owner, timestamps (access, modify, change), and more.

**Syntax:**
```bash
stat filename
```

**Examples:**
```bash
$ stat notes.txt
  File: notes.txt
  Size: 320       Blocks: 8          IO Block: 4096   regular file
Device: 802h/2050d Inode: 1234567    Links: 1
Access: (0644/-rw-r--r--)  Uid: (1000/user)   Gid: (1000/user)
Access: 2026-08-19 10:00:00.000000000 +0530
Modify: 2026-08-19 09:55:00.000000000 +0530
Change: 2026-08-19 09:55:00.000000000 +0530
```

**Key Point for Exam:** `stat` gives the **exact/precise byte size**, while `ls -l` and `ls -s` show sizes affected by block allocation/rounding.

---

### 6.3 `du`

**Concept:**
`du` (disk usage) reports the **actual disk space consumed** by files/directories (based on blocks used), which can differ from the apparent file size. It's especially useful for finding how much space a directory tree occupies.

**Syntax:**
```bash
du filename
du -h filename          # human-readable
du -s directory         # summary (total only)
du -sh directory         # summary + human-readable
```

**Examples:**
```bash
$ du notes.txt
4    notes.txt
# 4 KB blocks used

$ du -sh /home/user/Documents
120M    /home/user/Documents
# Total space used by the entire folder, human-readable

$ du -h *
# Disk usage of every item in the current directory
```

**Key Point for Exam — `du` vs `ls -l`:**
| Aspect | `ls -l` | `du` |
|---|---|---|
| Shows | Apparent/logical file size | Actual disk space used (in blocks) |
| Affected by block size rounding | No | Yes |

---

### 6.4 Role of Block Size

**Concept:**
Disk space is allocated to files in fixed-size chunks called **blocks** (commonly 4 KB on many Linux filesystems). Even if a file contains only 1 byte of data, it still occupies at least **one full block** on disk. This is why:
- A file's **apparent size** (from `ls -l`) may be small,
- But its **disk usage** (from `du` or `ls -s`) is rounded up to the nearest block size.

**Example:**
```bash
$ echo "hi" > tiny.txt
$ ls -l tiny.txt
-rw-r--r-- 1 user user 3 Aug 19 10:00 tiny.txt
# Apparent size: 3 bytes

$ du -h tiny.txt
4.0K    tiny.txt
# Actual disk usage: 4 KB (one full block), even though file is only 3 bytes
```

**Key Point for Exam:** Disk usage is always a **multiple of the block size**, so small files "waste" the unused portion of their last block. This explains why `du` totals are usually **larger** than the sum of apparent file sizes from `ls -l`.

---

## 7. In-Memory Filesystems

### 7.1 `/proc`

**Concept:**
`/proc` is a **virtual (pseudo) filesystem** that exists only in memory (RAM), not on disk. It provides a window into the **kernel and running processes** in real time. Every running process gets a numbered directory (`/proc/<PID>`) containing information about that process. Files here are generated dynamically when read — they don't occupy real disk space.

**Syntax:**
```bash
cat /proc/cpuinfo
cat /proc/meminfo
cat /proc/version
ls /proc/<PID>
```

**Examples:**
```bash
$ cat /proc/cpuinfo
# Shows detailed CPU information (model, cores, speed)

$ cat /proc/meminfo
# Shows current RAM usage statistics

$ cat /proc/version
Linux version 6.x.x ...
# Shows kernel version

$ ls /proc/1
# Shows information directory for process with PID 1 (usually init/systemd)

$ cat /proc/1/status
# Detailed status of process 1
```

---

### 7.2 `/sys`

**Concept:**
`/sys` (sysfs) is another **virtual, in-memory filesystem** that exposes information about **devices, drivers, and kernel subsystems** in a structured, hierarchical way. It is mainly used by the kernel to represent the device tree and allows viewing (and sometimes modifying) hardware/driver parameters.

**Syntax:**
```bash
ls /sys
cat /sys/class/... 
```

**Examples:**
```bash
$ ls /sys
block  bus  class  dev  devices  firmware  fs  kernel  module  power

$ cat /sys/class/net/eth0/address
# Shows the MAC address of network interface eth0 (example)

$ cat /sys/class/power_supply/BAT0/capacity
# Shows battery percentage on a laptop (example)
```

**Key Point for Exam — `/proc` vs `/sys`:**
| Feature | `/proc` | `/sys` |
|---|---|---|
| Focus | Processes + kernel info | Devices, drivers, kernel objects |
| Type | Virtual/in-memory | Virtual/in-memory |
| Introduced | Earlier, broader/legacy use | Later, cleaner structured design |
| Example use | `/proc/cpuinfo`, `/proc/<PID>` | `/sys/class/net/...` |

---

## 8. Summary

This lecture (Week 1, Lecture 4 — *Simple Commands in Linux, Part 2*) builds on basic Linux navigation and covers seven major areas:

1. **`ls` and `/`:** The root `/` is its own parent, redundant slashes collapse into one, options come in short (`-x`) and long (`--xxxx`) forms, a directory argument lists its contents (use `-d` to list the entry itself), `-R` gives recursive listing, and option order generally doesn't matter.

2. **File-viewing commands:** `cat` dumps whole files instantly; `more`/`less` page through content (with `less` supporting backward scroll and search, unlike `more`); `head`/`tail` show the first/last 10 lines by default (`-n` to customize, `tail -f` to follow live updates); `wc` counts lines, words, and bytes.

3. **Help/documentation commands:** `man` gives full manual pages; `whatis` gives a one-line summary for an exact command; `apropos` searches by keyword across descriptions; `which` finds the executable path in `$PATH`; `type` identifies whether a name is a built-in, alias, function, or executable; `help` covers shell built-ins only; `info` offers hyperlinked, menu-based documentation.

4. **Multiple arguments:** In commands like `cp`/`mv`, the second argument is the destination when there are two arguments; with 3+ arguments, the **last** argument must be an existing directory. Notably, `mv` handles directories without any special flag, while `cp` requires `-r`/`-R` to copy directories recursively.

5. **Links:** **Hard links** share the same inode as the original (same data, cannot cross filesystems or link directories) whereas **symbolic links** are pointer/path-based shortcuts (can cross filesystems, can link directories, but break if the target is removed).

6. **File sizes:** `ls -s` shows block-based size, `stat` shows exact byte-level metadata, and `du` shows actual disk usage of files/directories. All are influenced by **block size** — actual disk usage is always rounded up to the nearest block, so small files consume more disk space than their apparent size suggests.

7. **In-memory filesystems:** `/proc` and `/sys` are virtual filesystems that exist only in RAM, generated live by the kernel — `/proc` exposes process and kernel-wide info, while `/sys` exposes structured device/driver information.

**Exam Tip:** Pay special attention to command **pairs that are commonly confused** — `more` vs `less`, `which` vs `type`, `whatis` vs `apropos`, `-r` vs `-R` in `ls`, hard link vs symbolic link, and `cp -r` requirement vs `mv`'s implicit recursion. These distinctions are classic exam and viva questions.

---

*End of Notes — Lecture 4, Week 1: Simple Commands in Linux (Part 2)*
