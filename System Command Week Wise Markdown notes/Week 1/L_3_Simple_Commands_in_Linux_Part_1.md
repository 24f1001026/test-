
# Week 1 – Lecture 3: Simple Commands in Linux (Part 1)

## Table of Contents

1. [Introduction](#1-introduction)
2. [Basic System Information Commands](#2-basic-system-information-commands)
   - [2.1 date — Display Date and Time](#21-date--display-date-and-time)
   - [2.2 cal — Display Calendar of a Month](#22-cal--display-calendar-of-a-month)
   - [2.3 free — Display Memory Statistics](#23-free--display-memory-statistics)
   - [2.4 groups — Display Groups a User Belongs To](#24-groups--display-groups-a-user-belongs-to)
   - [2.5 file — Determine File Type](#25-file--determine-file-type)
3. [File Types and Permissions](#3-file-types-and-permissions)
   - [3.1 Understanding `ls -l` Output](#31-understanding-ls--l-output)
   - [3.2 File Types in Linux](#32-file-types-in-linux)
   - [3.3 The inode and `ls -i`](#33-the-inode-and-ls--i)
   - [3.4 Permission String Explained](#34-permission-string-explained)
   - [3.5 Numeric (Octal) Permission Values](#35-numeric-octal-permission-values)
4. [File and Directory Management Commands](#4-file-and-directory-management-commands)
   - [4.1 chmod — Change Permissions of a File](#41-chmod--change-permissions-of-a-file)
   - [4.2 touch — Change/Create Modified Timestamp of a File](#42-touch--changecreate-modified-timestamp-of-a-file)
   - [4.3 cp — Create a Copy of a File](#43-cp--create-a-copy-of-a-file)
   - [4.4 mv — Rename or Move a File](#44-mv--rename-or-move-a-file)
   - [4.5 mkdir — Create a Directory](#45-mkdir--create-a-directory)
   - [4.6 rm — Remove a File](#46-rm--remove-a-file)
5. [Summary](#5-summary)
6. [Quick Revision Table](#6-quick-revision-table)

---

## 1. Introduction

This lecture introduces the **Linux command line environment**, focusing on simple, everyday commands used to:

- Check basic **system information** (date, calendar, memory, user groups, file types)
- Understand **file types and permissions** in the Linux filesystem
- Perform basic **file and directory management** (create, copy, move, rename, delete, and change permissions)

These commands form the **foundation of Linux system administration** and are essential building blocks before moving on to more advanced shell operations. It is recommended to learn them in the order presented below — starting with information/inspection commands, then understanding the permission system, and finally the commands that modify files and directories.

---

## 2. Basic System Information Commands

These commands are used to **view** information about the system, time, memory, and users. They do not modify anything, making them safe to practice first.

### 2.1 `date` — Display Date and Time

**Concept:**
The `date` command displays the **current system date and time**. It can also be used to format the output or, with proper privileges, set the system date and time.

**Syntax:**
```bash
date [OPTION] [+FORMAT]
```

**Examples:**
```bash
# Display current date and time
date

# Output: Wed Aug 19 10:32:15 IST 2026

# Display date in custom format (day-month-year)
date +"%d-%m-%Y"
# Output: 19-08-2026

# Display only the current time
date +"%H:%M:%S"
# Output: 10:32:15
```

**Notes:**
- Commonly used in shell scripts for logging and timestamps.
- Format specifiers like `%d` (day), `%m` (month), `%Y` (year), `%H` (hour) allow custom output.

---

### 2.2 `cal` — Calendar of a Month

**Concept:**
The `cal` command displays a **calendar** for the current month by default. It can also display a calendar for a specific month/year or the entire year.

**Syntax:**
```bash
cal [[month] year]
```

**Examples:**
```bash
# Display calendar of the current month
cal

# Display calendar for a specific month and year (August 2026)
cal 8 2026

# Display calendar for the entire year 2026
cal 2026
```

**Notes:**
- Useful for quickly checking days of the week for planning or scripting tasks.
- On some systems, `cal -y` shows the current year's calendar.

---

### 2.3 `free` — Memory Statistics

**Concept:**
The `free` command displays statistics about **system memory usage**, including total, used, free, shared, buffer/cache, and available memory (RAM and swap).

**Syntax:**
```bash
free [OPTION]
```

**Examples:**
```bash
# Display memory usage in default units (KB)
free

# Display memory usage in human-readable format (MB/GB)
free -h

# Display memory usage in megabytes, refreshing every 2 seconds
free -m -s 2
```

**Notes:**
- Very useful for checking whether a system is running low on memory.
- `-h` (human-readable) is the most commonly used option in practice.

---

### 2.4 `groups` — Groups to Which a User Belongs

**Concept:**
The `groups` command displays the **group memberships** of the current user or a specified user. Groups control shared access permissions to files and resources.

**Syntax:**
```bash
groups [username]
```

**Examples:**
```bash
# Show groups of the currently logged-in user
groups

# Output: gphani sudo developers

# Show groups of a specific user
groups gphani
```

**Notes:**
- Useful when troubleshooting **permission-denied** issues, since access often depends on group membership.

---

### 2.5 `file` — What Type of a File It Is?

**Concept:**
The `file` command examines a file and reports its **type** (e.g., text file, directory, executable, image, archive) based on its content, not just its extension.

**Syntax:**
```bash
file [OPTION] filename
```

**Examples:**
```bash
# Check the type of a file
file notes.txt
# Output: notes.txt: ASCII text

# Check type of a compiled program
file a.out
# Output: a.out: ELF 64-bit LSB executable

# Check multiple files at once
file image.png document.pdf
```

**Notes:**
- Very helpful because Linux does not rely on file extensions to determine file type — `file` reads the actual file content/signature.

---

## 3. File Types and Permissions

Before performing file operations, it is essential to understand how Linux represents files, their types, and their permission structure — since almost every file command interacts with these concepts.

### 3.1 Understanding `ls -l` Output

Running `ls -l` gives a **long listing** of files with detailed metadata. Example output:

```
drwxr-xr-x 5 gphani gphani 12288 Nov 25 10:00 Documents
```

This output is broken down as follows:

| Field | Value | Meaning |
|---|---|---|
| File Type | `d` | Directory |
| Permissions | `rwxr-xr-x` | Owner, Group, Others permissions |
| Number of hard links | `5` | Count of hard links to the file |
| Owner | `gphani` | User who owns the file |
| Group | `gphani` | Group that owns the file |
| Size | `12288` | File size (in bytes) |
| Last modified timestamp | `Nov 25 10:00` | Date and time of last modification |
| Filename | `Documents` | Name of the file/directory |

---

### 3.2 File Types in Linux

Every file in Linux has a **type**, indicated by the first character in the `ls -l` output.

| Symbol | File Type |
|---|---|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `c` | Character (device) file |
| `b` | Block (device) file |
| `s` | Socket file |
| `p` | Named pipe (FIFO) |

**Notes:**
- Regular files (`-`) include text files, binaries, and images.
- Character and block files represent hardware devices (e.g., terminals, disks).
- Sockets and named pipes are used for inter-process communication (IPC).

---

### 3.3 The inode and `ls -i`

**Concept:**
An **inode** is an entry in the filesystem table that stores metadata about a file (permissions, owner, size, timestamps, and pointers to data blocks) — essentially information about **where the file is located on the storage media**. Every file has a unique inode number.

**Syntax:**
```bash
ls -i <name>
```

**Example:**
```bash
ls -i notes.txt
# Output: 123456 notes.txt
```

**Notes:**
- The filename itself is not stored in the inode; it is stored separately in the directory entry, which points to the inode.

---

### 3.4 Permission String Explained

A permission string looks like this:

```
rwxr-xr-x
```

It is divided into **three groups of three characters each**:

| Position | Characters | Applies To |
|---|---|---|
| 1st (positions 1–3) | `rwx` | **Owner** permissions |
| 2nd (positions 4–6) | `r-x` | **Group** permissions |
| 3rd (positions 7–9) | `r-x` | **Others** permissions |

Each group can contain:
- `r` = read permission
- `w` = write permission
- `x` = execute permission
- `-` = permission not granted

---

### 3.5 Numeric (Octal) Permission Values

Each permission triplet (`rwx`) can be represented as a **single octal digit** by summing values:

| Permission | Symbol | Value |
|---|---|---|
| Read | `r` | 4 |
| Write | `w` | 2 |
| Execute | `x` | 1 |
| No permission | `-` | 0 |

Common combinations:

| Octal | Symbolic | Meaning |
|---|---|---|
| 7 | `rwx` | Read + Write + Execute |
| 6 | `rw-` | Read + Write |
| 5 | `r-x` | Read + Execute |
| 4 | `r--` | Read only |
| 1 | `--x` | Execute only |

**Example:**
The permission string `rwxr-xr-x` translates to octal **755**:
- Owner: `rwx` = 7
- Group: `r-x` = 5
- Others: `r-x` = 5

---

## 4. File and Directory Management Commands

These commands are used to **create, modify, copy, move, and delete** files and directories. Use them carefully, especially `rm`, as changes are often irreversible.

### 4.1 `chmod` — Change Permissions of a File

**Concept:**
`chmod` (change mode) modifies the **read, write, and execute permissions** of a file or directory for the owner, group, and others. It accepts both **symbolic** and **numeric (octal)** modes.

**Syntax:**
```bash
chmod [OPTIONS] MODE FILE
```

**Examples:**
```bash
# Give the owner full permissions (rwx) using octal mode
chmod 755 script.sh

# Give everyone read and write permission
chmod 666 notes.txt

# Add execute permission for the owner (symbolic mode)
chmod u+x script.sh

# Remove write permission from group and others
chmod go-w report.txt

# Apply permissions recursively to a directory and its contents
chmod -R 755 project/
```

**Notes:**
- `u` = user/owner, `g` = group, `o` = others, `a` = all.
- `+` adds a permission, `-` removes it, `=` sets it exactly.
- Executable permission (`x`) is required to run scripts or enter directories.

---

### 4.2 `touch` — Change/Create Modified Timestamp of a File

**Concept:**
`touch` updates the **access and modification timestamps** of a file to the current time. If the file does not exist, `touch` creates a new, empty file.

**Syntax:**
```bash
touch [OPTION] filename
```

**Examples:**
```bash
# Create a new empty file
touch newfile.txt

# Update the timestamp of an existing file to the current time
touch notes.txt

# Create multiple files at once
touch file1.txt file2.txt file3.txt

# Set a specific timestamp
touch -t 202608191030 notes.txt
```

**Notes:**
- Frequently used to quickly create empty placeholder files.
- Does **not** change file content — only timestamps (or creates the file if absent).

---

### 4.3 `cp` — Create a Copy of a File

**Concept:**
`cp` copies files or directories from a source location to a destination, creating a duplicate while keeping the original intact.

**Syntax:**
```bash
cp [OPTION] SOURCE DESTINATION
```

**Examples:**
```bash
# Copy a file to another name
cp notes.txt notes_backup.txt

# Copy a file into a directory
cp notes.txt Documents/

# Copy a directory and its contents recursively
cp -r Documents/ Documents_Backup/

# Copy while preserving file attributes (timestamps, permissions)
cp -p notes.txt notes_copy.txt
```

**Notes:**
- The `-r` (recursive) option is **mandatory** when copying directories.
- If the destination file already exists, `cp` overwrites it by default (use `-i` for a confirmation prompt).

---

### 4.4 `mv` — Rename or Move a File

**Concept:**
`mv` moves a file or directory to a new location, or renames it if the destination is in the same directory with a different name. Unlike `cp`, it does not leave a copy behind.

**Syntax:**
```bash
mv [OPTION] SOURCE DESTINATION
```

**Examples:**
```bash
# Rename a file
mv oldname.txt newname.txt

# Move a file into a directory
mv notes.txt Documents/

# Move multiple files into a directory
mv file1.txt file2.txt Documents/

# Move and rename a directory
mv Project/ Project_Final/
```

**Notes:**
- `mv` works for both **renaming** and **moving** — the operation depends on whether the destination path changes location, name, or both.
- Use `-i` to get a prompt before overwriting an existing file.

---

### 4.5 `mkdir` — Create a Directory

**Concept:**
`mkdir` (make directory) creates one or more new, empty directories.

**Syntax:**
```bash
mkdir [OPTION] directory_name
```

**Examples:**
```bash
# Create a single directory
mkdir Projects

# Create multiple directories at once
mkdir Docs Images Videos

# Create nested directories in one command
mkdir -p Projects/2026/August

# Create a directory and set permissions simultaneously
mkdir -m 755 SecureFolder
```

**Notes:**
- The `-p` (parents) option creates all necessary parent directories without error if they already exist.

---

### 4.6 `rm` — Remove a File

**Concept:**
`rm` (remove) deletes files or directories permanently. **There is no default Recycle Bin/Trash in the command line** — deleted files are generally not recoverable.

**Syntax:**
```bash
rm [OPTION] filename
```

**Examples:**
```bash
# Remove a single file
rm notes.txt

# Remove multiple files
rm file1.txt file2.txt

# Remove a file with a confirmation prompt
rm -i notes.txt

# Remove a directory and all its contents recursively
rm -r Documents/

# Force remove without confirmation (use with caution)
rm -rf temp_folder/
```

**Notes:**
- ⚠️ **Caution:** `rm -rf` deletes everything without confirmation and cannot be undone — double-check the path before running it.
- Use `-i` (interactive mode) when practicing, to avoid accidental deletion.

---

## 5. Summary

- **Information commands** (`date`, `cal`, `free`, `groups`, `file`) are non-destructive commands used to inspect the system, time, memory, user groups, and file types — a safe starting point for beginners.
- Linux organizes every filesystem object with a **file type** (regular file, directory, symbolic link, character/block device, socket, pipe) identified by the first character of `ls -l` output.
- Every file has an **inode**, which stores its metadata (permissions, owner, size, timestamps) and is identified using `ls -i`.
- **Permissions** are represented as a 9-character string (`rwxr-xr-x`) divided into **owner, group, and others**, and can also be expressed numerically using **octal values** (r=4, w=2, x=1), e.g., `755`.
- **File management commands** allow full control over the filesystem:
  - `chmod` – change permissions
  - `touch` – create files / update timestamps
  - `cp` – copy files or directories
  - `mv` – move or rename files/directories
  - `mkdir` – create directories
  - `rm` – delete files or directories (irreversible — use with caution)
- Mastering these commands provides the essential foundation for working confidently in the Linux command-line environment before progressing to more advanced topics such as piping, redirection, and shell scripting.

---

## 6. Quick Revision Table

| Command | Purpose | Common Example |
|---|---|---|
| `date` | Show date and time | `date +"%d-%m-%Y"` |
| `cal` | Show calendar | `cal 8 2026` |
| `free` | Show memory statistics | `free -h` |
| `groups` | Show user's group memberships | `groups gphani` |
| `file` | Identify file type | `file notes.txt` |
| `ls -l` | Long listing with permissions & metadata | `ls -l Documents/` |
| `ls -i` | Show inode number | `ls -i notes.txt` |
| `chmod` | Change file permissions | `chmod 755 script.sh` |
| `touch` | Create file / update timestamp | `touch newfile.txt` |
| `cp` | Copy files/directories | `cp -r Docs/ Docs_Backup/` |
| `mv` | Move or rename files/directories | `mv old.txt new.txt` |
| `mkdir` | Create directories | `mkdir -p A/B/C` |
| `rm` | Delete files/directories | `rm -rf temp/` |

---

*End of Notes — Week 1, Lecture 2: Simple Commands in Linux (Part 1)*
