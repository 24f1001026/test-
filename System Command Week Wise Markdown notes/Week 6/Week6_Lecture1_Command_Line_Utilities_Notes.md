# Week 6 — Lecture 1: Important Command Line Utilities
### Tools That Can Augment Your Productivity

---

## Table of Contents

1. [Introduction to Utilities](#1-introduction-to-utilities)
2. [The `find` Command](#2-the-find-command)
   - 2.1 [Syntax](#21-syntax)
   - 2.2 [Options / Conditions Explained](#22-options--conditions-explained)
   - 2.3 [Examples](#23-examples)
3. [File Packaging & Compression](#3-file-packaging--compression)
   - 3.1 [Why We Need Packaging](#31-why-we-need-packaging)
   - 3.1.1 [Checking Disk Usage First — `du`](#311-checking-disk-usage-first--du)
   - 3.2 [`tar` — Tape Archive](#32-tar--tape-archive)
   - 3.3 [`gzip` — GNU Zip](#33-gzip--gnu-zip)
   - 3.4 [Other Compression Utilities](#34-other-compression-utilities)
   - 3.5 [Points to Remember While Packaging](#35-points-to-remember-while-packaging)
4. [The `make` Utility](#4-the-make-utility)
   - 4.1 [Syntax](#41-syntax)
   - 4.2 [Makefile Structure Explained](#42-makefile-structure-explained)
   - 4.3 [Examples](#43-examples)
5. [Additional Utility Covered: `du` (Disk Usage)](#5-additional-utility-covered-du-disk-usage)
6. [Quick Reference Tables](#6-quick-reference-tables)
7. [Summary](#7-summary)

---

## 1. Introduction to Utilities

Command line utilities are small, powerful programs that help increase efficiency while working in a Unix/Linux environment. This lecture covers three major categories of utilities:

| Utility | Purpose |
|---|---|
| `find` | Locating files based on conditions and processing them |
| `tar`, `gzip`, etc. | Packaging and compressing collections of files |
| `make` | Automating conditional actions/build tasks |

> **Learning order suggestion:** Learn `find` first (used daily for searching files), then move to `tar`/`gzip` (used for backup and sharing), and finally `make` (used for automation and builds — slightly more advanced).

---

## 2. The `find` Command

### 2.1 Syntax

```bash
find [pathnames] [conditions]
```

- **pathnames** — one or more directories where the search should begin (e.g., `.` for current directory, `/home/user` for a specific path).
- **conditions** — a combination of options/flags that filter which files are matched, and optional actions to perform on matches.

### 2.2 Options / Conditions Explained

| Option | Description |
|---|---|
| `-name` | Matches filenames against a given pattern (supports wildcards like `*`, `?`). |
| `-type` | Filters by file type code — e.g., `f` for regular file, `d` for directory, `l` for symbolic link, `c` for character device, `b` for block device. |
| `-atime` | Filters files by **access time**. `+n` = accessed more than *n* days ago, `-n` = accessed less than *n* days ago. |
| `-ctime` | Filters files by **change time** (metadata change). `+n` = changed more than *n* days ago, `-n` = changed less than *n* days ago. |
| `-mtime` | Filters files by **modification time** (content change). `+n` = modified more than *n* days ago, `-n` = modified less than *n* days ago. *(Note: this is the corrected form of the shorthand `-m` used verbally in the lecture — the actual flag is `-mtime`.)* |
| `-size` | Filters files by **size**. `+n` = larger than *n*, `-n` = smaller than *n*. Suffixes: `c` (bytes), `k` (KB), `M` (MB), `G` (GB). Example: `-size +10M` means larger than 10 MB. |
| `-regex` | Matches the full path using a regular expression instead of a simple filename pattern. |
| `-regextype` | Used along with `-regex` to specify the regex flavor, e.g., `posix-basic`, `posix-egrep`. |
| `-exec` | Executes a specified command on each matched file; `{}` is used as a placeholder for the filename, and the command must end with `\;` or `+`. |
| `-print` | Prints the full pathname of each matching file (this is the default action in most modern implementations). |

#### 2.2.1 File Type Codes for `-type` (Expanded)

| Code | File Type |
|---|---|
| `f` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `c` | Character special file (device) |
| `b` | Block special file (device) |
| `p` | Named pipe (FIFO) |
| `s` | Socket |

### 2.3 Examples

**Example 1 — Find by name:**
```bash
find . -name "*.txt"
```
Finds all files ending in `.txt` in the current directory and its subdirectories.

**Example 2 — Find directories only:**
```bash
find /home/user -type d
```
Lists all directories under `/home/user`.

**Example 3 — Find files modified/accessed more than 30 days ago:**
```bash
find . -atime +30 -print
```
Prints paths of files not accessed in the last 30 days — useful for identifying stale files.

**Example 4 — Find files changed within the last 2 days:**
```bash
find . -ctime -2
```
Finds files whose metadata changed less than 2 days ago.

**Example 5 — Find and delete matching files using `-exec`:**
```bash
find . -name "*.tmp" -exec rm {} \;
```
Finds all `.tmp` files and deletes each one; `{}` is replaced by the matched filename, and `\;` terminates the `-exec` command.

**Example 6 — Find using a regular expression:**
```bash
find . -regextype posix-egrep -regex ".*/(report|summary)[0-9]+\.pdf"
```
Finds files whose full path matches the given extended POSIX regular expression (e.g., `report1.pdf`, `summary23.pdf`).

**Example 7 — Combine multiple conditions:**
```bash
find /var/log -type f -name "*.log" -ctime +7 -exec gzip {} \;
```
Finds all `.log` files in `/var/log` older than 7 days (by change time) and compresses each using `gzip`.

**Example 8 — Count the total number of files in a directory:**
```bash
find $HOME -print | wc -l
```
`find $HOME -print` lists every file/folder under the home directory, and piping it into `wc -l` (word count, line mode) counts the total number of lines — i.e., the total number of files and directories found.

**Example 9 — Find files modified in the last 2 days:**
```bash
find . -mtime -2
```
Lists files whose **content was last modified less than 2 days ago**.

**Example 10 — Find files modified more than 30 days ago:**
```bash
find . -mtime +30
```
Lists files whose content has **not been modified in over 30 days** — useful for identifying old/unused files before archiving or cleanup.

**Example 11 — Find specific directories using a wildcard pattern:**
```bash
find /usr -type d -name "man?" -print
```
Searches `/usr` for **directories** (`-type d`) whose name matches the pattern `man?` (where `?` matches exactly one character) — this is commonly used to locate manual page directories such as `man1`, `man2`, `man3`, etc.

**Example 12 — Find large files and list them with human-readable sizes:**
```bash
find . -size +10M -exec ls -lsh {} \;
```
Finds all files larger than 10 MB in the current directory tree, and for each match runs `ls -lsh` (long listing, with size shown, in human-readable form like `12M`, `1.2G`) to display details.

**Example 13 — Find all JPG images and show their sizes:**
```bash
find . -name '*.jpg' -exec ls -sh {} \;
```
Finds every `.jpg` file recursively and lists its human-readable size using `ls -sh` — handy for quickly auditing image files before compressing/archiving a folder.

---

## 3. File Packaging & Compression

### 3.1 Why We Need Packaging

Two common problems make file transfer and storage inefficient:

| Problem | Explanation |
|---|---|
| Deep file hierarchies | Directories nested many levels deep are cumbersome to copy/transfer one by one. |
| Large number of tiny files | Many small files consume more disk space (due to block allocation overhead) and are slow to transfer individually. |

**Solution approach (two-step process):**

1. **`tar`** — collects an entire file hierarchy (files + folders) into a **single archive file**.
2. **`gzip`** — **compresses** that single file to reduce its size.

**Common applications:**
- Backup
- File sharing
- Reducing disk utilization

### 3.1.1 Checking Disk Usage First — `du`

**Concept:** Before packaging/compressing a folder, it is often useful to check **how much disk space it currently occupies**. The `du` (disk usage) command reports this.

**Syntax:**
```bash
du [options] [file/directory]
```

| Option | Meaning |
|---|---|
| `-s` | Summarize — show only the total for each argument (don't list every subdirectory) |
| `-h` | Human-readable format (shows sizes as `K`, `M`, `G` instead of raw bytes) |

**Examples:**

```bash
# Show total size of a folder in human-readable form
du -sh logfiles/

# Show sizes of all subfolders in the current directory
du -sh */

# Combine with sort to find the largest folders
du -sh */ | sort -rh
```
`du -sh` is typically run **before** creating an archive, so you know roughly how large the resulting `.tar`/`.tar.gz` file might be, and to decide which compression tool is worth using.

### 3.2 `tar` — Tape Archive

**Concept:** `tar` bundles multiple files and directories into one archive file (commonly with extension `.tar`), preserving the directory structure, permissions, and metadata. It does **not** compress by default — it only "packs."

**Syntax:**
```bash
tar [options] archive_name.tar [file(s)/directory]
```

**Common options:**

| Option | Meaning |
|---|---|
| `-c` | Create a new archive |
| `-x` | Extract files from an archive |
| `-t` | List/table the contents of an archive |
| `-v` | Verbose — show files being processed |
| `-f` | Specify the archive filename (must usually be the last flag before the filename) |
| `-z` | Filter the archive through `gzip` (compress/decompress) while archiving |
| `-j` | Filter through `bzip2` |

**Examples:**

```bash
# Create an archive of a directory
tar -cvf project_backup.tar ./project/

# Extract an archive
tar -xvf project_backup.tar

# List contents without extracting
tar -tvf project_backup.tar

# Create a compressed archive directly (tar + gzip together)
tar -czvf project_backup.tar.gz ./project/

# Extract a .tar.gz archive
tar -xzvf project_backup.tar.gz
```

**Example 5 — Lecture walkthrough: packaging a `logfiles` folder:**
```bash
tar -cvf logfiles.tar logfiles/
```
This creates a single archive called `logfiles.tar` containing everything inside the `logfiles/` directory. At this stage the archive is **only bundled, not compressed** — its size will be roughly equal to the sum of the original files.

**Example 6 — Extracting the `logfiles.tar` archive back:**
```bash
tar -xvf logfiles.tar
```
Extracts the contents of `logfiles.tar` into the current directory, restoring the original `logfiles/` folder structure.

**Example 7 — Combined comparison: `tar` vs `zip` vs `unzip`:**

```bash
# STEP 1: Package + compress the same folder three different ways

# (a) tar alone — archive only, no compression
tar -cvf logfiles.tar logfiles/

# (b) tar + gzip together — archive AND compress in one command
tar -czvf logfiles.tar.gz logfiles/

# (c) zip — archive AND compress in one command (alternative to tar+gzip)
zip -r logfiles.zip logfiles/


# STEP 2: Reverse each of the above

# (a) Extract a plain .tar archive
tar -xvf logfiles.tar

# (b) Extract a .tar.gz archive in one step
tar -xzvf logfiles.tar.gz

# (c) Extract a .zip archive
unzip logfiles.zip
```

**Explanation:**
- `tar -cvf` only **bundles** files together — it does **not** shrink their size. You need a separate compression step (`gzip`, `bzip2`, etc.) afterward, or the combined `-z`/`-j` flags.
- `tar -czvf` does **both steps in one command** — `c` (create), `z` (gzip-compress), `v` (verbose), `f` (filename) — producing a `.tar.gz` file directly.
- `zip -r` is a **self-contained alternative**: unlike `tar`, it packages *and* compresses in a single native step without needing a second tool, which is why it's commonly preferred when sharing files with Windows/macOS users (who have built-in zip support).
- To reverse the process: use `tar -xvf` for a plain `.tar`, `tar -xzvf` for a `.tar.gz` (single command extracts and decompresses together), and `unzip` for a `.zip` file.
- **Exam tip:** Remember that `tar` and `zip` solve the *same problem* (packaging + compression) but with different philosophies — `tar` traditionally separates "packing" from "compressing" (hence needing `-z`/`-j` flags or a second command like `gzip`), while `zip` does both natively in one step.

### 3.3 `gzip` — GNU Zip

**Concept:** `gzip` compresses a single file to reduce its size (replacing the original file with a `.gz` version by default). It is commonly used **after** `tar` to compress the packaged archive.

**Syntax:**
```bash
gzip [options] filename
gunzip [options] filename.gz     # to decompress
```

**Examples:**

```bash
# Compress a file (project_backup.tar becomes project_backup.tar.gz)
gzip project_backup.tar

# Decompress a .gz file
gunzip project_backup.tar.gz

# Keep the original file while compressing (don't delete source)
gzip -k project_backup.tar

# Check compression ratio without extracting
gzip -l project_backup.tar.gz
```

**Example 5 — Lecture walkthrough: compressing the `logfiles.tar` archive:**
```bash
gzip logfiles.tar
```
This replaces `logfiles.tar` with a compressed file named `logfiles.tar.gz`. Combined with the earlier `tar -cvf` step, this completes the standard **package-then-compress** workflow.

**Example 6 — Decompressing back to `.tar`:**
```bash
gunzip logfiles.tar.gz
# OR equivalently:
gzip -d logfiles.tar.gz
```
Both commands restore `logfiles.tar` from `logfiles.tar.gz`. After this, `tar -xvf logfiles.tar` can be used to fully extract the original files (see Example 6 in section 3.2).

### 3.4 Other Compression Utilities

The lecture mentions several alternative packaging/compression tools, each provided by different packages:

| Tool | Provided by (package) | Notes |
|---|---|---|
| `tar` | — | Archiving only (no compression by default) |
| `zip` | — | Archives **and** compresses together (popular for cross-platform sharing) |
| `compress` | `ncompress` | Older Unix compression utility (`.Z` extension) |
| `gzip` | `gzip` | Fast, widely used, moderate compression (`.gz`) |
| `bzip2` | `bzip2` | Better compression ratio than gzip, slower (`.bz2`) |
| `xz` | `xz-utils` | Very high compression ratio, slower speed (`.xz`) |
| `7z` | `p7zip-full` | High compression, supports many formats (`.7z`) |

**Combined naming convention:** A `tar` archive that is also `gzip`-compressed is often called a **"tarball"** and conventionally named with a `.tgz` or `.tar.gz` extension.

```bash
# Example of naming convention
myproject.tgz      # equivalent to myproject.tar.gz
```

> **Note:** A full worked example comparing `tar`, `tar+gzip`, and `zip`/`unzip` side by side — with explanation — is provided at the end of Section 3.2 (Example 7).

**Example — Lecture walkthrough: comparing `bzip2` and `compress` on the same archive:**

```bash
# Compress with bzip2 — produces a smaller file, but takes more time
bzip2 logfiles.tar
# Result: logfiles.tar.bz2 (smaller, slower to create)

# Compress with compress — produces a larger file, but is faster
compress logfiles.tar
# Result: logfiles.tar.Z (larger, faster to create)
```

**Decompressing each format:**
```bash
# Decompress a .bz2 file
bzip2 -d logfiles.tar.bz2
# OR
bunzip2 logfiles.tar.bz2

# Decompress a .Z file
uncompress logfiles.tar.Z
```

> **Key exam point:** `bzip2` trades **more compression time** for a **smaller final file size**, while `compress` is **faster** but leaves a **larger final file size**. This directly illustrates the *time vs. compression-ratio trade-off* discussed in section 3.5 below.

### 3.5 Points to Remember While Packaging

| Consideration | Explanation |
|---|---|
| Time vs. memory trade-off | Higher compression ratio (smaller size) generally requires **more time** to compress/decompress (e.g., `xz` > `bzip2` > `gzip` in ratio, but also in time taken). |
| Portability | Not all systems support all formats by default; `.tar.gz`/`.zip` are the most portable choices. |
| Unique naming for backups | Use identifiers such as a **timestamp** or **process ID (PID)** in the backup filename to avoid overwriting previous backups and to ensure uniqueness. |

**Example — creating a uniquely named backup using a timestamp:**
```bash
tar -czvf backup_$(date +%Y%m%d_%H%M%S).tar.gz ./data/
# Produces something like: backup_20260823_143015.tar.gz
```

**Example — using process ID for a temporary unique archive:**
```bash
tar -cvf temp_$$.tar ./cache/
# $$ is replaced by the current shell's process ID
```

---

## 4. The `make` Utility

**Concept:** `make` is a build-automation tool that reads instructions from a file (conventionally called a `Makefile`) and performs actions **conditionally** — that is, it only re-executes a recipe (set of commands) when the target is missing or its prerequisites have changed (based on file timestamps). It is widely used to compile programs, but can be used to automate *any* repetitive shell task.

### 4.1 Syntax

```bash
make -f make.file
```

- `-f` specifies the name of the makefile to use (if the file is named exactly `Makefile` or `makefile`, this flag is not required — `make` will detect it automatically).

### 4.2 Makefile Structure Explained

A basic Makefile is composed of the following building blocks:

```makefile
# comments
TMP_FILES = *.o *.aux

.PHONY : clean

target : prerequisites
	recipe
clean:
	rm -f $(TMP_FILES)
```

| Component | Explanation |
|---|---|
| `# comments` | Lines starting with `#` are comments and are ignored by `make`. |
| `VARIABLE = value` | Defines a **variable** (macro) that can be reused later using `$(VARIABLE)`. Example: `TMP_FILES = *.o *.aux`. |
| `.PHONY : target` | Declares a target (like `clean`) as **phony**, meaning it does not correspond to an actual file — this prevents conflicts if a file with the same name as the target exists, and ensures the recipe always runs when called. |
| `target : prerequisites` | Defines a **rule**: `target` is what you want to build; `prerequisites` are the files/targets it depends on. |
| `recipe` | The shell command(s) to execute to build the target. **Must be indented with a Tab character**, not spaces (a very common source of errors). |
| `$(VARIABLE_NAME)` | Syntax used to **reference/expand** a previously defined variable within a recipe or elsewhere in the Makefile. |

> **Important correction/clarification:** In the recipe line, `$(OPTION_NAME)` refers to expanding a variable named `OPTION_NAME` that would have been defined earlier in the Makefile (similar to how `$(TMP_FILES)` is used in the `clean` target). It is a placeholder to show that recipes can use variables — it is **not** a fixed keyword.

### 4.3 Examples

**Example 1 — Basic compilation Makefile:**
```makefile
CC = gcc
CFLAGS = -Wall -g

hello: hello.c
	$(CC) $(CFLAGS) -o hello hello.c
```
Run with:
```bash
make -f Makefile
```
This compiles `hello.c` into an executable named `hello`, but **only if** `hello.c` is newer than `hello` (or `hello` doesn't exist yet).

**Example 2 — Using `.PHONY` and a `clean` target:**
```makefile
TMP_FILES = *.o *.aux

.PHONY : clean

all: myprogram

myprogram: main.o utils.o
	gcc -o myprogram main.o utils.o

main.o: main.c
	gcc -c main.c

utils.o: utils.c
	gcc -c utils.c

clean:
	rm -f $(TMP_FILES) myprogram
```
Run with:
```bash
make            # builds "all" (the default first target)
make clean      # removes temporary/object files and the executable
```

**Example 3 — Multiple targets with dependencies:**
```makefile
report.pdf: report.tex
	pdflatex report.tex

report.tex: data.csv
	python3 generate_report.py data.csv > report.tex
```
Here, running `make report.pdf` will automatically regenerate `report.tex` first (if `data.csv` changed), and then rebuild `report.pdf` — this is the essence of **conditional, dependency-based automation**.

**Example 4 — Lecture use case: using `make` to automate backups:**

`make` is not limited to compiling code — it can automate **any** repetitive shell workflow, including taking backups using the `tar`/`gzip` commands learned in Section 3.

```makefile
SRC_DIR = logfiles
BACKUP  = logfiles_$(shell date +%Y%m%d).tar.gz

.PHONY : backup clean

backup: $(BACKUP)

$(BACKUP): $(SRC_DIR)
	tar -czvf $(BACKUP) $(SRC_DIR)/
	@echo "Backup created: $(BACKUP)"

clean:
	rm -f logfiles_*.tar.gz
```

Run with:
```bash
make backup      # creates a fresh, uniquely-dated backup only if needed
make clean        # removes old backup archives
```

**Explanation:**
- `SRC_DIR` and `BACKUP` are **variables** — `BACKUP` uses `$(shell date +%Y%m%d)` to embed today's date directly into the filename, giving each backup a **unique name** (the same idea covered in Section 3.5).
- `backup: $(BACKUP)` means the `backup` target's real job is to ensure the file named in `$(BACKUP)` exists — so `make` checks: does today's dated `.tar.gz` already exist? If yes, it does nothing (saving time); if no, it runs the recipe.
- The actual recipe `tar -czvf $(BACKUP) $(SRC_DIR)/` reuses the exact `tar` + `gzip` combined-flag command taught in Section 3.2 — this is where `make` and the packaging utilities connect directly.
- `.PHONY : backup clean` ensures both `backup` and `clean` always run their recipes on request, even if a file or folder with those exact names happens to exist.
- **Exam tip:** This is the clearest illustration of *why* `make` matters beyond compiling code — it turns a manual, repeatable shell task (creating dated backups) into a single, safe, conditional command (`make backup`) that avoids redundant work.

**Why this is powerful:** Because `backup`'s prerequisite is the `logfiles` directory, `make` will only recreate the archive if the source folder has changed since the last backup — avoiding unnecessary, repeated compression of unchanged data. This combines the **conditional execution** idea from `make` with the **packaging workflow** from Section 3.

---

## 5. Additional Utility Covered: `du` (Disk Usage)

> This topic was introduced in the lecture alongside packaging (to check folder sizes) but is documented here as its own quick-reference entry since it is a standalone utility in its own right.

**Concept:** `du` reports how much disk space files and directories are consuming — useful for deciding what to archive, compress, or clean up.

**Syntax:**
```bash
du [options] [path]
```

| Option | Meaning |
|---|---|
| `-s` | Summary — total only, no per-subdirectory breakdown |
| `-h` | Human-readable sizes (`K`, `M`, `G`) |

**Examples:**
```bash
du -sh logfiles/            # total size of one folder, human-readable
du -sh *                    # size of every item in the current directory
du -sh */ | sort -rh | head # top 5 largest folders (combined with sort/head)
```

---

## 6. Quick Reference Tables

### 6.1 All Utilities at a Glance

| Utility | Category | Main Purpose |
|---|---|---|
| `find` | Search | Locate files by name, type, time, or pattern, and act on them |
| `tar` | Packaging | Bundle multiple files/directories into a single archive |
| `gzip` | Compression | Compress a single file to reduce its size |
| `compress`, `bzip2`, `xz`, `7z`, `zip` | Compression | Alternative compression tools with different speed/ratio trade-offs |
| `make` | Automation | Conditionally execute build/task recipes based on file dependencies |
| `du` | Inspection | Report disk space used by files/directories |

### 6.2 `find` Options Recap

| Option | Function |
|---|---|
| `-name` | Match filename pattern |
| `-type` | Match file type (`f`, `d`, `l`, `c`, `b`, `p`, `s`) |
| `-atime` | Filter by last access time (in days) |
| `-ctime` | Filter by last metadata change time (in days) |
| `-mtime` | Filter by last modification time (in days) |
| `-size` | Filter by file size (`c`, `k`, `M`, `G` suffixes) |
| `-regex` | Match full path via regex |
| `-regextype` | Set regex flavor (`posix-basic`, `posix-egrep`, etc.) |
| `-exec ... {} \;` | Run a command on each match |
| `-print` | Print matching pathnames |

### 6.3 `tar` Options Recap

| Option | Function |
|---|---|
| `-c` | Create archive |
| `-x` | Extract archive |
| `-t` | List archive contents |
| `-v` | Verbose output |
| `-f` | Specify archive filename |
| `-z` | Compress/decompress using gzip |
| `-j` | Compress/decompress using bzip2 |

### 6.4 Compression Tools: Speed vs. Size Recap

| Tool | Relative Speed | Relative Final Size | Decompress Command |
|---|---|---|---|
| `compress` | Fastest | Largest | `uncompress file.Z` |
| `gzip` | Fast | Moderate | `gunzip file.gz` / `gzip -d file.gz` |
| `bzip2` | Slower | Smaller | `bunzip2 file.bz2` / `bzip2 -d file.bz2` |
| `xz` | Slowest | Smallest | `unxz file.xz` |

### 6.5 Makefile Components Recap

| Element | Function |
|---|---|
| `VAR = value` | Define a variable |
| `$(VAR)` | Use/expand a variable |
| `.PHONY` | Mark a target as not a real file |
| `target : prerequisites` | Define build rule |
| Tab-indented recipe | Commands executed to build the target |

---

## 7. Summary

- **`find`** is used to **search for files** in a directory hierarchy based on criteria such as name (`-name`), type (`-type`), access time (`-atime`), modification time (`-mtime`), change time (`-ctime`), or size (`-size`); it can also **act on results directly** using `-exec`, or simply display them using `-print`. Regular expressions (`-regex`, `-regextype`) allow more advanced pattern matching. It can also be combined with utilities like `wc -l` (to count matches) and `ls -lsh` (to inspect matched files).

- **`du`** complements `find` by reporting **disk space usage** (`du -sh`) — typically checked before deciding what to archive or clean up.

- **File packaging and compression** solve the problems of deep, complex directory structures and large numbers of small files. The standard workflow is:
  1. Use **`tar`** to bundle everything into one archive file (e.g., `tar -cvf logfiles.tar logfiles/`).
  2. Use **`gzip`** (or alternatives like `bzip2`, `compress`, `xz`, `7z`, `zip`) to compress that archive, producing a "tarball" (e.g., `.tar.gz` / `.tgz`).
  3. Reverse the process with `tar -xvf` to extract, and `gunzip`/`bunzip2`/`uncompress` to decompress.
  - Key trade-offs to remember: **compression ratio vs. time taken** (`compress` = fastest/largest, `bzip2`/`xz` = slower/smaller), **portability** across systems, and giving backups **unique names** (using timestamps or process IDs) to avoid overwriting.

- **`make`** automates repetitive tasks — not just compiling code, but also routine operations like **backups** — using a **Makefile**, which defines **targets**, their **prerequisites**, and the **recipe** (commands) to build them. `make` is *conditional* — it only rebuilds a target when necessary, based on file modification times. `.PHONY` targets (like `clean` or `backup`) are used for actions that don't correspond to a single, predictable output file.

**Exam takeaway:** Focus first on memorizing the exact flags for `find` (`-name`, `-type`, `-atime`, `-mtime`, `-ctime`, `-size`, `-exec`), then the tar/gzip/bzip2/compress workflow and the meaning of a "tarball," then `du -sh` for disk usage checks, and finally the structure of a Makefile (variable definition, `.PHONY`, target-prerequisite-recipe pattern, and tab-indentation rule).
