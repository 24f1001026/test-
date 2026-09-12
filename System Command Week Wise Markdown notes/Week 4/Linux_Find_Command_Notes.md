# LINUX `find` COMMAND :

## Table of Contents

1. [Introduction & Concept](#1-introduction--concept)
2. [Basic Syntax](#2-basic-syntax)
3. [Searching by Name](#3-searching-by-name)
4. [Searching by Type](#4-searching-by-type)
5. [Searching by Size](#5-searching-by-size)
6. [Searching by Time (Modification, Access, Change)](#6-searching-by-time-modification-access-change)
7. [Searching by Permissions](#7-searching-by-permissions)
8. [Searching by Owner & Group](#8-searching-by-owner--group)
9. [Limiting Search Depth](#9-limiting-search-depth)
10. [Combining Conditions (Logical Operators)](#10-combining-conditions-logical-operators)
11. [Executing Commands on Results (`-exec` and `-ok`)](#11-executing-commands-on-results--exec-and--ok)
12. [Deleting Files Found](#12-deleting-files-found)
13. [Using `find` with `xargs`](#13-using-find-with-xargs)
14. [Searching by Empty Files/Directories](#14-searching-by-empty-filesdirectories)
15. [Excluding Files/Directories](#15-excluding-filesdirectories)
16. [Case-Insensitive Search](#16-case-insensitive-search)
17. [Searching by Inode Number](#17-searching-by-inode-number)
18. [Printing Results in Custom Formats](#18-printing-results-in-custom-formats)
19. [Combining `find` with Other Commands](#19-combining-find-with-other-commands)
20. [Common Errors & Debugging Tips](#20-common-errors--debugging-tips)
21. [Full Cheat Sheet](#21-full-cheat-sheet)
22. [Summary](#22-summary)

---

## 1. Introduction & Concept

**`find`** is a powerful Linux command-line utility used to **search for files and directories** within a directory hierarchy, based on a wide range of criteria: name, type, size, permissions, timestamps, owner, and more.

**Core Concept:**
- `find` **walks recursively** through a starting directory (and all its subdirectories) and evaluates every file/folder against the conditions (called "**tests**" or "**expressions**") you provide.
- Unlike `locate` (which searches a pre-built database and can be outdated), `find` searches the **live filesystem in real time**, so results are always current — but it can be slower on very large directory trees.
- `find` can not only *locate* files but also **take action on them** directly (delete, move, change permissions, run a command) using options like `-exec` and `-delete`.

**Why learn `find`?**
- Essential for system administration, cleanup scripts, log rotation, backups, and security audits (e.g., finding world-writable files).
- Frequently tested in Linux exams/certifications (RHCSA, LPIC, CompTIA Linux+) because it combines many core Linux concepts: permissions, timestamps, file types, and piping.

---

## 2. Basic Syntax

```bash
find [path] [expression]
```

| Part | Meaning |
|---|---|
| `path` | The directory to start searching from (e.g., `.`, `/home`, `/var/log`). If omitted, defaults to the current directory. |
| `expression` | One or more **tests**, **operators**, and **actions** that determine what's matched and what happens to matches. |

**First Basic Command (Example):**
```bash
find /home/user -name "notes.txt"
```
**Explanation:**
- `find` → the command itself
- `/home/user` → the starting directory to search (searches this folder and all subfolders)
- `-name "notes.txt"` → the test/condition: match files named exactly `notes.txt`

**Sample Output:**
```
/home/user/documents/notes.txt
```

**Simplest possible form — list everything:**
```bash
find .
```
> Searches the **current directory** (`.`) and prints every file and folder found, with no filtering conditions at all.

---

## 3. Searching by Name

### 3.1 Exact Name Match — `-name`
**Syntax:**
```bash
find [path] -name "pattern"
```
```bash
find . -name "report.txt"          # find a file with this EXACT name (case-sensitive)
```

### 3.2 Case-Insensitive Name Match — `-iname`
```bash
find . -iname "report.txt"          # matches Report.txt, REPORT.TXT, report.TXT, etc.
```

### 3.3 Wildcard Patterns
`find` supports shell-style wildcards inside the pattern (must be quoted so the shell doesn't expand them first):
```bash
find . -name "*.txt"               # all files ending in .txt
find . -name "report*"              # all files starting with "report"
find . -name "*.log*"                # all files containing ".log" anywhere in the name
find . -name "file?.txt"              # ? matches exactly ONE character -> file1.txt, fileA.txt, etc.
```

### 3.4 Searching by Regex Pattern
```bash
find . -regex ".*\.\(txt\|log\)$"    # matches files ending in .txt OR .log using POSIX regex
```

---

## 4. Searching by Type

**Syntax:**
```bash
find [path] -type [f|d|l|b|c|p|s]
```

| Flag | Meaning |
|---|---|
| `f` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `b` | Block device |
| `c` | Character device |
| `p` | Named pipe (FIFO) |
| `s` | Socket |

**Examples:**
```bash
find . -type f                    # list only regular FILES
find . -type d                     # list only DIRECTORIES
find . -type l                       # list only SYMBOLIC LINKS
find /dev -type b                      # list block devices (e.g., disk drives) in /dev
```

**Combine type with name:**
```bash
find . -type f -name "*.sh"          # find only files (not folders) ending in .sh
```

---

## 5. Searching by Size

**Syntax:**
```bash
find [path] -size [+|-]N[unit]
```

| Symbol | Meaning |
|---|---|
| `+N` | Greater than N |
| `-N` | Less than N |
| `N` (no sign) | Exactly N |

| Unit | Meaning |
|---|---|
| `c` | Bytes |
| `k` | Kilobytes |
| `M` | Megabytes |
| `G` | Gigabytes |
| `b` | 512-byte blocks (default if no unit given) |

**Examples:**
```bash
find . -size +100M              # files LARGER than 100 MB
find . -size -1k                  # files SMALLER than 1 KB
find . -size 50M                    # files EXACTLY 50 MB (rare, precise match)
find . -type f -size +1G              # large files over 1 GB -- great for freeing disk space
```

---

## 6. Searching by Time (Modification, Access, Change)

`find` can filter based on three different timestamps every file has:

| Test | Timestamp meaning |
|---|---|
| `-mtime` | **M**odification time — content was last changed |
| `-atime` | **A**ccess time — file was last read/opened |
| `-ctime` | **C**hange time — metadata (permissions/owner) was last changed |

**Syntax (time in days):**
```bash
find [path] -mtime [+|-]N
```
| Symbol | Meaning |
|---|---|
| `+N` | More than N days ago |
| `-N` | Less than N days ago |
| `N` | Exactly N days ago |

**Examples:**
```bash
find . -mtime -7                  # files MODIFIED in the last 7 days
find . -mtime +30                   # files NOT modified in over 30 days (good for cleanup)
find . -atime +90                     # files NOT accessed in over 90 days
find . -ctime -1                        # metadata changed within the last 1 day
```

**Minute-level precision** (use `-mmin`, `-amin`, `-cmin` for minutes instead of days):
```bash
find . -mmin -30                  # files modified in the LAST 30 MINUTES
find . -mmin +60                    # files modified MORE than 60 minutes ago
```

**Compare against a reference file — `-newer`:**
```bash
find . -newer reference.txt        # files modified more recently than reference.txt
```

---

## 7. Searching by Permissions

**Syntax:**
```bash
find [path] -perm mode
```

**Examples:**
```bash
find . -perm 755                  # files with EXACTLY permission 755 (rwxr-xr-x)
find . -perm -644                   # files with AT LEAST these permission bits set (- prefix)
find . -perm /222                     # files where ANY write bit is set (owner, group, or other)
find / -perm -4000                      # find SUID (set-user-ID) files -- important for security audits
find / -perm -2000                        # find SGID (set-group-ID) files
find / -type f -perm -o+w                   # world-writable files -- common security check
```
> **Exam distinction:** `-perm 755` (no prefix) = **exact match**; `-perm -755` = **all these bits must be set** (others may also be set); `-perm /755` = **any of these bits** set is enough to match.

---

## 8. Searching by Owner & Group

**Syntax:**
```bash
find [path] -user username
find [path] -group groupname
```

**Examples:**
```bash
find /home -user john               # files owned by user "john"
find /var -group www-data             # files owned by group "www-data"
find / -nouser                          # files with NO valid owner (orphaned -- owner deleted)
find / -nogroup                           # files with NO valid group (orphaned)
find . -uid 1000                            # search using numeric user ID instead of username
```

---

## 9. Limiting Search Depth

**Syntax:**
```bash
find [path] -maxdepth N
find [path] -mindepth N
```

**Examples:**
```bash
find . -maxdepth 1 -type f          # only files in the CURRENT folder, don't go into subfolders
find . -maxdepth 2 -name "*.txt"      # search up to 2 levels deep only
find . -mindepth 2 -type d              # only directories that are AT LEAST 2 levels deep (skip top level)
```
> **Performance tip:** Using `-maxdepth` early in large filesystems significantly speeds up searches by avoiding unnecessary recursion.

---

## 10. Combining Conditions (Logical Operators)

| Operator | Meaning | Syntax |
|---|---|---|
| `-a` or (space, implicit) | AND (both conditions must be true) | `find . -name "*.txt" -a -size +1k` |
| `-o` | OR (either condition true) | `find . -name "*.txt" -o -name "*.log"` |
| `!` or `-not` | NOT (negate a condition) | `find . ! -name "*.txt"` |
| `( )` | Group conditions (must be escaped in bash: `\( \)`) | `find . \( -name "*.txt" -o -name "*.log" \)` |

**Examples:**
```bash
find . -name "*.txt" -size +1k                     # AND is implicit between conditions
find . \( -name "*.txt" -o -name "*.log" \)           # match EITHER .txt OR .log files
find . ! -name "*.txt"                                  # everything EXCEPT .txt files
find . -type f \( -name "*.jpg" -o -name "*.png" \) -size +5M  # image files over 5MB
```

---

## 11. Executing Commands on Results (`-exec` and `-ok`)

**Definition:** `-exec` runs a specified command on every file that `find` matches. `{}` is a placeholder for the matched file path, and the command must end with `\;` (or `+` for batch execution).

**Syntax:**
```bash
find [path] [conditions] -exec command {} \;
```

**Examples:**
```bash
find . -name "*.tmp" -exec rm {} \;                  # delete every matched .tmp file, one at a time
find . -name "*.sh" -exec chmod +x {} \;                # make all matched shell scripts executable
find . -type f -name "*.txt" -exec cat {} \;              # print the content of every matched .txt file
find . -name "*.log" -exec mv {} /var/logs/archive/ \;      # move matched files to another folder
```

**Batch execution with `+` (faster — passes multiple files to one command call instead of one-by-one):**
```bash
find . -name "*.tmp" -exec rm {} +
```

**`-ok` — same as `-exec` but asks for confirmation before each action (safer):**
```bash
find . -name "*.tmp" -ok rm {} \;
# Prompts: "rm ./file.tmp ? (y/n)" before every deletion
```

---

## 12. Deleting Files Found

**Syntax:**
```bash
find [path] [conditions] -delete
```

**Examples:**
```bash
find . -name "*.tmp" -delete                    # delete all matched .tmp files directly
find . -type f -mtime +365 -delete                 # delete files older than a year (cleanup script)
find . -type d -empty -delete                        # delete empty directories
```
> **⚠️ Safety tip:** Always run the same `find` command **without** `-delete` first to preview which files will be affected, before adding `-delete`. This prevents accidentally destroying important data.

---

## 13. Using `find` with `xargs`

**Concept:** `xargs` builds and executes command lines from standard input — often faster than `-exec ... \;` for large numbers of files, since it batches arguments together instead of spawning a new process per file.

**Syntax:**
```bash
find [path] [conditions] | xargs command
```

**Examples:**
```bash
find . -name "*.log" | xargs rm                    # delete all matched .log files
find . -name "*.txt" | xargs grep "error"             # search for "error" inside all matched .txt files
find . -name "*.jpg" -print0 | xargs -0 rm             # safely handle filenames with spaces/special chars
```
> **Exam tip:** Always use `-print0` (find) combined with `xargs -0` when filenames might contain spaces or special characters — this uses a NULL character as a separator instead of whitespace, preventing filename-splitting errors.

---

## 14. Searching by Empty Files/Directories

**Syntax:**
```bash
find [path] -empty
```

**Examples:**
```bash
find . -type f -empty              # find empty FILES (0 bytes)
find . -type d -empty                # find empty DIRECTORIES (no contents)
find . -empty -delete                  # find AND delete all empty files/folders in one step
```

---

## 15. Excluding Files/Directories

**Syntax (using `-prune` to skip a directory entirely):**
```bash
find [path] -path "excluded_path" -prune -o [conditions] -print
```

**Examples:**
```bash
find . -path "./node_modules" -prune -o -name "*.js" -print
# searches for .js files but SKIPS the node_modules folder entirely (much faster on big projects)

find / -path "/proc" -prune -o -name "*.conf" -print
# search the whole filesystem for .conf files, but skip the virtual /proc filesystem

find . -not -path "./.git/*" -name "*.py"
# simpler alternative: exclude .git folder contents without using -prune
```

---

## 16. Case-Insensitive Search

```bash
find . -iname "readme*"           # matches README.md, readme.txt, ReadMe.TXT, etc.
find . -iregex ".*\.(jpg|png)$"     # case-insensitive regex matching
```

---

## 17. Searching by Inode Number

**Concept:** Every file has a unique **inode number** on its filesystem. This is useful for finding hard links or files with ambiguous/broken names.

```bash
ls -i file.txt                     # first, get the inode number of a file, e.g., 123456
find . -inum 123456                  # find all files/hard-links sharing that same inode
```

---

## 18. Printing Results in Custom Formats

**Syntax:**
```bash
find [path] [conditions] -printf "format"
```

**Examples:**
```bash
find . -name "*.txt" -printf "%f\n"            # print only the FILENAME (not full path)
find . -type f -printf "%s %p\n"                  # print SIZE and full PATH for each file
find . -type f -printf "%TY-%Tm-%Td %p\n"           # print last modified DATE and path
```
| Format Code | Meaning |
|---|---|
| `%p` | Full file path |
| `%f` | Filename only |
| `%s` | File size (bytes) |
| `%u` | Owner username |
| `%TY-%Tm-%Td` | Modification date (YYYY-MM-DD) |

---

## 19. Combining `find` with Other Commands

```bash
find . -name "*.txt" | wc -l                     # COUNT how many .txt files were found
find . -type f -name "*.log" | sort                 # sort the list of matched file paths alphabetically
find . -name "*.csv" -exec wc -l {} \;                 # count lines in each matched CSV file
find . -type f -newer file.txt -exec ls -lh {} \;         # detailed listing of files newer than file.txt
du -sh $(find . -type f -name "*.mp4")                      # total size of all matched video files
```

---

## 20. Common Errors & Debugging Tips

| Error / Issue | Cause | Fix |
|---|---|---|
| `find: paths must precede expression` | Path given AFTER the conditions | Always put the path **first**: `find . -name "x"`, not `find -name "x" .` |
| `-exec` command not running | Missing `\;` or `+` terminator | Always end with `\;` (escaped semicolon) or `+` |
| Wildcard expands too early / wrong results | Forgot to quote the pattern | Always quote patterns: `-name "*.txt"`, not `-name *.txt` |
| Search extremely slow on `/` | Searching entire filesystem without limits | Use `-maxdepth`, or exclude virtual filesystems like `/proc` with `-prune` |
| Filenames with spaces break `xargs` | Default whitespace-based splitting | Use `find ... -print0 | xargs -0 ...` |
| Accidentally deleted wrong files with `-delete` | Ran `-delete` without previewing first | Always test the same command WITHOUT `-delete` first |

---

## 21. Full Cheat Sheet

| Category | Command | Example |
|---|---|---|
| Basic | `find [path] [expr]` | `find . -name "*.txt"` |
| By Name | `-name`, `-iname` | `find . -iname "readme*"` |
| By Type | `-type f/d/l` | `find . -type d` |
| By Size | `-size +N/-N` | `find . -size +100M` |
| By Time | `-mtime`, `-atime`, `-ctime`, `-mmin` | `find . -mtime -7` |
| By Permission | `-perm mode` | `find . -perm 755` |
| By Owner | `-user`, `-group`, `-nouser` | `find . -user john` |
| Depth Limit | `-maxdepth`, `-mindepth` | `find . -maxdepth 1` |
| Logic | `-a`, `-o`, `!`, `\( \)` | `find . \( -name "*.jpg" -o -name "*.png" \)` |
| Action | `-exec cmd {} \;`, `-ok` | `find . -name "*.tmp" -exec rm {} \;` |
| Delete | `-delete` | `find . -empty -delete` |
| With xargs | `\| xargs cmd` | `find . -name "*.log" \| xargs rm` |
| Empty | `-empty` | `find . -type f -empty` |
| Exclude | `-prune`, `-not -path` | `find . -path "./node_modules" -prune -o -print` |
| Custom Output | `-printf "format"` | `find . -printf "%s %p\n"` |
| Inode | `-inum N` | `find . -inum 123456` |

---

## 22. Summary

The `find` command is one of the most powerful and versatile tools in Linux for **locating and acting on files/directories** across a filesystem in real time. Its basic syntax — `find [path] [expression]` — combines a **starting location** with one or more **tests** (name, type, size, time, permissions, owner) that every file is checked against.

**Key takeaways:**
- Use `-name`/`-iname` for filename matching (with wildcards or regex), and `-type` to restrict results to files, directories, or links.
- Use `-size`, `-mtime`/`-atime`/`-ctime` to filter by size and by modification/access/change time — essential for cleanup and auditing tasks.
- Use `-perm`, `-user`, and `-group` for permission- and ownership-based searches, common in security audits.
- Control the depth of recursion with `-maxdepth`/`-mindepth`, and skip unwanted directories entirely using `-prune` for performance.
- Combine multiple conditions using `-a` (AND, implicit), `-o` (OR), and `!` (NOT), grouped with escaped parentheses `\( \)`.
- Take direct action on matches using `-exec`/`-ok` (run any command) or `-delete` (remove matches) — always **test without `-delete` first** to avoid accidental data loss.
- Pipe results into `xargs` for faster, batched processing on large result sets, using `-print0`/`xargs -0` when filenames may contain spaces.

Mastering `find` means being comfortable combining these building blocks — search criteria + logical operators + actions — into a single precise command, which is exactly what most exam and real-world sysadmin scenarios require.

---

*End of Notes — Good luck with your exam!*
