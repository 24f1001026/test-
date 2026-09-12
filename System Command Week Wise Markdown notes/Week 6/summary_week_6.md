# Week 6 — Revision Summary
### Command Line Utilities + Bash Shell Scripting (Part 1)

Quick-revision notes covering both Week 6 lectures. Edge cases and "gotchas" are folded directly into each topic (marked with ⚠️) instead of a separate section, so you see the trap right where the concept is taught.

---

## 1. Software Tools Philosophy

Good scripts/tools should follow these design rules (common theory question):

| Principle | Meaning |
|---|---|
| Do one thing well | Single, focused purpose per tool |
| Process text, not binary | So output of one tool can feed another |
| Use regex | Standard pattern matching across tools |
| Default to stdin/stdout | Read/write standard I/O unless a file is given |
| Don't be chatty | No unnecessary messages — keeps pipelines clean |
| Same output format as input | Enables chaining tools together |
| Reuse existing tools | Don't reinvent functionality |
| Build a specialized tool only if nothing fits | Last resort |

⚠️ **Remember:** This philosophy explains *why* Unix favors small commands joined with `|` instead of one giant program — a common conceptual exam question.

---

## 2. The `find` Command

**Syntax:** `find [pathnames] [conditions]`

| Option | Meaning |
|---|---|
| `-name` | Match filename pattern (wildcards `*`, `?`) |
| `-type` | `f`=file, `d`=directory, `l`=symlink, `c`=char device, `b`=block device, `p`=pipe, `s`=socket |
| `-atime` | Access time filter (`+n` older than n days, `-n` within n days) |
| `-ctime` | Metadata change time filter |
| `-mtime` | Content modification time filter |
| `-size` | Size filter (`c`=bytes, `k`=KB, `M`=MB, `G`=GB); `+n` bigger, `-n` smaller |
| `-regex` | Match full path with regex |
| `-regextype` | Set regex flavor (`posix-basic`, `posix-egrep`) |
| `-exec cmd {} \;` | Run a command on each match |
| `-print` | Print matching paths (default action) |

⚠️ **Edge cases to remember:**
- `-atime`, `-ctime`, `-mtime` look almost identical but test **different timestamps** (access vs metadata-change vs content-change) — exams love mixing these up.
- `+n` vs `-n` is *reversed* from what beginners expect: `+30` means **older than** 30 days, `-2` means **within** 2 days. Get this backwards and the logic flips.
- `-exec {} \;` — the `\;` (escaped semicolon) is mandatory to terminate the command; forgetting the backslash or the space before it is a classic syntax error. (`+` instead of `\;` runs the command once on *all* matches together, which is faster but behaves differently.)
- `find . -name "*.jpg"` **must be quoted** (`"*.jpg"`), otherwise the shell expands the wildcard itself before `find` ever sees it, breaking the search in directories with matching files.
- `-print` is the default action in modern `find`, so omitting it is fine — but exams may still test whether you know it's implicit.

**Key examples:**
```bash
find . -name "*.txt"                      # by name
find /home/user -type d                   # directories only
find . -atime +30                         # not accessed in 30+ days
find . -mtime -2                          # modified in last 2 days
find . -name "*.tmp" -exec rm {} \;       # find & delete
find $HOME -print | wc -l                 # count files (find + wc combo)
find . -size +10M -exec ls -lsh {} \;     # large files, human-readable listing
```

---

## 3. File Packaging & Compression

**Two-step workflow:** `tar` (bundle) → `gzip`/other (compress). Solves the problems of deep directory trees and many small files (block-allocation overhead, slow transfer).

### 3.1 `du` — check size *before* packaging

| Option | Meaning |
|---|---|
| `-s` | Summary total only |
| `-h` | Human-readable (K/M/G) |

```bash
du -sh logfiles/          # total size of one folder
du -sh */ | sort -rh      # largest folders first
```

### 3.2 `tar` — Tape Archive (bundles, does NOT compress by itself)

| Flag | Meaning |
|---|---|
| `-c` | Create archive |
| `-x` | Extract |
| `-t` | List contents |
| `-v` | Verbose |
| `-f` | Specify archive filename (usually placed right before the filename) |
| `-z` | Pipe through gzip |
| `-j` | Pipe through bzip2 |

```bash
tar -cvf logfiles.tar logfiles/     # bundle only
tar -xvf logfiles.tar               # extract
tar -tvf logfiles.tar               # list without extracting
tar -czvf logfiles.tar.gz logfiles/ # bundle + gzip in ONE command
tar -xzvf logfiles.tar.gz           # extract + decompress in ONE command
```

⚠️ **Edge case:** `tar -cvf` alone does **not** shrink file size — it only combines files. People often assume `tar` compresses by default; it doesn't unless you add `-z`/`-j` or run `gzip` separately afterward.

### 3.3 `gzip` — compress a single file

```bash
gzip project_backup.tar        # replaces file with .tar.gz (deletes original!)
gzip -k project_backup.tar     # keep original while compressing
gunzip project_backup.tar.gz   # decompress (or: gzip -d file.gz)
gzip -l project_backup.tar.gz  # check compression ratio without extracting
```

⚠️ **Edge case:** Plain `gzip filename` **deletes the original file** by default and replaces it with `.gz`. Use `-k` if you need to keep the source — this trips people up in exams and in practice.

### 3.4 Other compression tools — speed vs. size trade-off

| Tool | Package | Extension | Speed | Final Size |
|---|---|---|---|---|
| `compress` | ncompress | `.Z` | Fastest | Largest |
| `gzip` | gzip | `.gz` | Fast | Moderate |
| `bzip2` | bzip2 | `.bz2` | Slower | Smaller |
| `xz` | xz-utils | `.xz` | Slowest | Smallest |
| `7z` | p7zip-full | `.7z` | — | High compression, many formats |
| `zip` | — | `.zip` | — | Archives **and** compresses in one native step (unlike tar) |

⚠️ **Edge case:** `tar` and `zip` solve the same problem but differently — `tar` **separates** packing from compressing (needs `-z`/`-j` or a second command), while `zip` does both **natively in one step**. This distinction is a favorite exam comparison.

A `.tar.gz` (or `.tgz`) is called a **"tarball."**

### 3.5 Points to remember while packaging

| Consideration | Explanation |
|---|---|
| Time vs. compression ratio | Better ratio (smaller file) = more time needed (`xz` > `bzip2` > `gzip` in both ratio and time) |
| Portability | `.tar.gz` / `.zip` are most universally supported |
| Unique backup names | Use timestamp (`$(date +%Y%m%d_%H%M%S)`) or PID (`$$`) so backups don't overwrite each other |

```bash
tar -czvf backup_$(date +%Y%m%d_%H%M%S).tar.gz ./data/
tar -cvf temp_$$.tar ./cache/          # $$ = current shell PID
```

---

## 4. The `make` Utility

**Concept:** Reads a `Makefile` and **conditionally** re-runs recipes — only when a target is missing or older than its prerequisites (based on timestamps). Not just for compiling code — great for automating any repetitive task (e.g., dated backups).

```bash
make -f make.file      # -f not required if file is literally "Makefile"/"makefile"
```

| Component | Meaning |
|---|---|
| `# comment` | Ignored by make |
| `VAR = value` | Define a variable/macro |
| `$(VAR)` | Expand/use a variable |
| `.PHONY : target` | Marks target as not a real file — always runs, avoids naming clashes |
| `target : prerequisites` | Build rule |
| Recipe line | Commands to build the target |

⚠️ **Edge case (the #1 Makefile mistake):** Recipe lines **must be indented with a real Tab character**, not spaces. Using spaces causes a `missing separator` error — this is the most common Makefile bug.

⚠️ **Edge case:** Without `.PHONY`, if a real file named `clean` (or `backup`, etc.) exists in the directory, `make` may think the target is already "up to date" and **skip running the recipe** — because it compares against a file of that name. `.PHONY` prevents this.

```makefile
CC = gcc
CFLAGS = -Wall -g

hello: hello.c
	$(CC) $(CFLAGS) -o hello hello.c

.PHONY : clean
clean:
	rm -f *.o hello
```

Backup automation example (combines `make` + `tar`/`gzip` from Section 3):
```makefile
BACKUP = logfiles_$(shell date +%Y%m%d).tar.gz
.PHONY : backup
backup: $(BACKUP)
$(BACKUP): logfiles
	tar -czvf $(BACKUP) logfiles/
```
`make backup` only re-creates the archive if `logfiles/` changed since the last dated backup — avoids redundant work.

---

## 5. Utilities Quick Recap Table

| Utility | Category | Purpose |
|---|---|---|
| `find` | Search | Locate files by name/type/time/size, act on them |
| `du` | Inspection | Report disk usage (check before archiving) |
| `tar` | Packaging | Bundle files/dirs into one archive |
| `gzip`/`bzip2`/`xz`/`compress`/`7z`/`zip` | Compression | Shrink archive size (different speed/ratio trade-offs) |
| `make` | Automation | Conditionally run build/task recipes |

---

## 6. Introduction to Shell Scripts

A script is a text file of commands starting with a **shebang** line telling the OS which interpreter to use.

| Interpreter | Shebang |
|---|---|
| Bash | `#!/bin/bash` |
| awk | `#!/usr/bin/awk -f` |
| sed | `#!/bin/sed -f` |
| Python | `#!/usr/bin/env python3` |
| Ruby | `#!/usr/bin/env ruby` |
| Perl | `#!/usr/bin/perl` |

**Creating a script with `vi`:** `vi s1.sh` → press `i` (insert mode) → type script → `Esc` → `:wq` (save & quit). `:q!` quits **without** saving if you made a mistake.

⚠️ A newly created script is just a text file — it's **not executable** and hasn't been run yet. It must be either **sourced** or **executed** (next section).

---

## 7. Sourced vs Executed Scripts (major exam topic)

| Feature | Sourced (`. script` / `source script`) | Executed (`./script`) |
|---|---|---|
| Permission needed | None | Needs `chmod +x` |
| Process | Runs in current shell — **no new process** | Creates a **new child process** |
| PID (`echo $$`) | **Same** as parent shell | **Different** from parent shell |
| Environment/variables after finishing | **Persist** in current shell | **Lost** once child process ends |
| `$0` inside script | Shows parent shell name (e.g. `bash`) | Shows the script's own name/path |
| Typical use | Setting up environment (vars, aliases) | Running a standalone task |

```bash
echo $$                 # PID of current shell, e.g. 2431
source s1.sh             # PID inside script prints 2431 (same!)
chmod +x s1.sh
./s1.sh                  # PID inside script prints 2587 (different — new process)
ps --forest               # visualize parent-child process tree
```

⚠️ **Edge cases:**
- If a script exports a variable and you ask "will it be visible in my terminal afterward?" — the answer depends entirely on **how it was run**: `source`/`.` → yes; `./script` → no (its whole environment, including the variable, dies with the child process).
- `$0` is a reliable trick to detect invocation style — useful for a config script meant to always be *sourced*, never executed, so it can warn the user if run the wrong way.
- `ps --forest` only shows a separate branch for an **executed** script (new PID); a **sourced** script shows no extra branch since it never left the parent shell.

---

## 8. Script Location & `$PATH`

| Method | Example |
|---|---|
| Absolute path | `/home/user/scripts/backup.sh` |
| Relative path | `./backup.sh` |
| In a `$PATH` directory | `backup.sh` (no `./` needed) |

⚠️ **Edge case:** If two scripts share the same name and both live in directories listed in `$PATH`, Bash runs whichever is found **first** based on directory order in `$PATH` — not necessarily the one you intended.

---

## 9. Login vs Non-Login Shell

| Shell Type | Files Read (in order) |
|---|---|
| Login shell | `/etc/profile` → `~/.bash_profile` → `~/.bash_login` → `~/.profile` (only the **first** of these three found) |
| Non-login shell | `/etc/bash.bashrc` → `~/.bashrc` |

⚠️ **Edge case:** A variable set in `.bash_profile` won't appear in a brand-new terminal tab, because a new GUI terminal tab is usually a **non-login** shell (reads `.bashrc`, not `.bash_profile`). This mismatch is a classic exam/real-world "why isn't my variable there" question.

---

## 10. Output & Input

| Feature | `echo` | `printf` |
|---|---|---|
| Trailing newline | Automatic (use `-n` to suppress) | Must add `\n` manually |
| Format specifiers | Not supported | `%s`, `%d`, `%f`, etc. |
| Complexity | Simple | More control, more verbose |

Input: `read var` (stores typed text into `$var`); `read var1 var2` reads multiple values; `read -p "Prompt: " var` shows a prompt inline.

---

## 11. Script Arguments

| Symbol | Meaning |
|---|---|
| `$0` | Script name |
| `$#` | Number of arguments |
| `$1`, `${1}` | First argument |
| `${11}` | 11th argument — **curly braces required** for args ≥ 10 |
| `$*` / `$@` | All arguments |
| `"$*"` | All args merged into **one** string |
| `"$@"` | Each arg kept **separate** (preserves quoting) |

⚠️ **Edge case (very commonly tested):** With args `"a b"` and `"c"` — `"$*"` gives `"a b c"` as **one item**, while `"$@"` gives `"a b"` and `"c"` as **two separate items**. Always prefer `"$@"` in `for arg in "$@"` loops so multi-word quoted arguments aren't accidentally split.

```bash
for arg in "$@"; do echo "Argument: $arg"; done
if [ $# -eq 0 ]; then echo "No arguments were passed!"; fi
```

---

## 12. Command Substitution

```bash
var=`command`        # legacy backtick form
var=$(command)        # modern, preferred — nests cleanly
```

⚠️ **Edge case:** Backticks **cannot be nested cleanly** (you'd need escaped inner backticks); `$( )` nests naturally, e.g. `echo "Files: $(find . -mtime 0 | wc -l)"`. Prefer `$()` in new scripts.

---

## 13. Loops

| Loop | Runs when | Best for |
|---|---|---|
| `for var in list; do ... done` | Once per item in list | Iterating a known set (space is the default separator; change via `IFS`) |
| `while condition; do ... done` | Condition is **true** | Counters, reading input until false |
| `until condition; do ... done` | Condition is **false** (opposite of while) | Repeating until something becomes true |

```bash
for num in {1..5}; do echo "Number: $num"; done
for file in *.txt; do grep "error" "$file"; done      # globbing
```

⚠️ **Edge case (important, real-world bug source):** Looping over command output with `for line in $(grep "root" /etc/passwd)` splits on **words/spaces**, not lines — a line containing spaces gets broken into multiple loop items. The safe pattern for **whole lines** is:
```bash
grep "bash" /etc/passwd | while read -r line; do echo "$line"; done
```
Use `while read` (piped), not `for`, whenever lines may contain spaces.

⚠️ `IFS` (Internal Field Separator) controls what `for` splits on — default is space; change it (e.g. `IFS=","`) for comma-separated lists, but remember to reset it afterward or it affects the rest of the script.

---

## 14. Conditionals

**`if` statement:**
```bash
if [ $age -ge 18 ]; then
    echo "Eligible"
elif [ $marks -ge 60 ]; then
    echo "Grade B"
else
    echo "Grade C"
fi
```

⚠️ **Edge case (classic beginner error):** Always leave a **space** after `[` and before `]` — `[ $x -eq 5 ]` not `[$x -eq 5]`. `[` is actually a **command**, not just a bracket, so missing spaces cause "command not found" or syntax errors.

**`case` statement** — cleaner than many `if-elif` for one variable against multiple patterns:
```bash
case $fruit in
    apple) echo "red or green" ;;
    banana) echo "yellow" ;;
    *) echo "Unknown fruit" ;;      # default/catch-all, like `default` in switch
esac
```
⚠️ Don't forget the `;;` terminator after each pattern block, and `*)` as the fallback case.

---

## 15. Test Expressions / Conditions

| Condition Type | Syntax | Example |
|---|---|---|
| Command | any command's exit status | `wc -l file` |
| Pipeline | commands joined with `\|` | `who \| grep "joy" > /dev/null` |
| Test expression | `test expr` or `[ expr ]` | `[ -e file ]` |
| Negation | `!` | `! [ -e file ]` |
| Extended test | `[[ expr ]]` (Bash-only, pattern matching) | `[[ $ver == 5.* ]]` |
| Arithmetic | `(( expr ))` | `(( v ** 2 > 10 ))` |

### Numeric comparisons
| Operator | Meaning |
|---|---|
| `-eq` | equal |
| `-ne` | not equal |
| `-gt` | greater than |
| `-ge` | greater or equal |
| `-lt` | less than |
| `-le` | less or equal |

### String comparisons
| Operator | Meaning |
|---|---|
| `=` | equal |
| `!=` | not equal |
| `<`, `>` | lexicographic compare |
| `-z` | zero length (empty) |
| `-n` | non-zero length |

⚠️ **Edge case:** Inside plain `[ ]`, `<` and `>` are interpreted as **redirection operators**, not comparisons — they must be escaped (`\<`, `\>`) or used inside `[[ ]]` instead. Mixing up numeric vs string operators (`-eq` vs `=`) is another very common mistake — `-eq` is for numbers only, `=`/`!=` for strings.

### File comparisons — unary (single file)
| Operator | Meaning |
|---|---|
| `-e` | exists |
| `-d` | is a directory |
| `-f` | is a regular file |
| `-r` | readable |
| `-w` | writable |
| `-x` | executable |
| `-s` | not empty (size > 0) |
| `-O` | owned by current user |
| `-G` | group matches current user |

### File comparisons — binary (two files)
| Operator | Meaning |
|---|---|
| `-nt` | newer than |
| `-ot` | older than |

---

## 16. Functions

```bash
myfunc() {
    commands
}
myfunc      # call
```

⚠️ **Edge case:** A function **must be defined before** it is called in the script — calling it earlier in the file than its definition will fail.

```bash
add_numbers() {
    result=$(( $1 + $2 ))
    echo $result
}
sum=$(add_numbers 5 10)   # capture return "value" via command substitution
```

| Term | Meaning |
|---|---|
| Definition | Where the function body is written |
| Call | Just writing the function's name to run it |

---

## 17. High-Priority Comparison Table (most-tested contrasts)

| Compare | Key Difference |
|---|---|
| `source script` vs `./script` | Same shell/PID, env persists **vs** new process/PID, env lost |
| `"$*"` vs `"$@"` | One merged string **vs** each argument kept separate |
| `while` vs `until` | Runs while true **vs** runs while false |
| `-eq`/`-gt`... vs `=`/`!=` | Numeric comparison **vs** string comparison |
| `tar` vs `zip` | Packs only (needs separate compress step) **vs** packs + compresses natively |
| `-atime` vs `-ctime` vs `-mtime` | Access time vs metadata-change time vs content-modification time |
| Login shell files vs Non-login shell files | `/etc/profile` + `.bash_profile` family **vs** `/etc/bash.bashrc` + `.bashrc` |
| `for` loop vs `while read` (for command output) | Splits by word/IFS **vs** reads whole lines safely |

---

## 18. One-Line Exam Cheat List

- `$$` = current shell/script PID; use it to prove sourced vs executed.
- `chmod +x` is required only for **executing**, never for **sourcing**.
- `${11}` needs braces; `$1`–`$9` don't.
- `"$@"` in loops preserves quoted multi-word arguments — always prefer it over `$*`.
- `$()` beats backticks for nesting.
- `for` splits by `IFS` (default space) — use `while read -r line` for line-safe processing.
- `[ ]` needs spaces around brackets; `<`/`>` need `[[ ]]` or escaping.
- `tar` = pack only; add `-z`/`-j` or a separate `gzip`/`bzip2` step to compress.
- `gzip file` deletes the original — use `-k` to keep it.
- Makefile recipes need a **Tab**, not spaces, or you get `missing separator`.
- `.PHONY` protects targets like `clean`/`backup` from being skipped due to a same-named file.
- Functions must be **defined before** they're called.
