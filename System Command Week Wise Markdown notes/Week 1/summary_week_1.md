# Week 1 Summary — Linux Command Line (Quick Revision)

Covers: Lecture 2 (Command Line Environment), Lecture 3 (Simple Commands Part 1), Lecture 4 (Simple Commands Part 2)

---

## 1. Terminal, Shell & Prompt

| Term | Meaning |
|---|---|
| Terminal | The window/app (Terminal, Konsole, xterm) — just the front end |
| Shell | The program that reads & runs commands (bash, zsh) |
| Prompt | `user@machine:~$` → username, hostname, current dir (`~` = home) |
| `$` vs `#` | `$` = normal user, `#` = **root** — be extra careful, root commands run with no confirmation |

---

## 2. Anatomy of a Command

```
command [options] [arguments]
```

| Rule | Detail |
|---|---|
| Short options | Single `-`, combinable → `-la` = `-l -a` |
| Long options | Double `--`, NOT combinable → each needs its own `--` |
| Option order | Doesn't matter → `-la` = `-al` |
| Argument order | Often DOES matter (esp. `cp`/`mv`) |

---

## 3. Orientation Commands

| Command | What it does | ⚠️ Edge case |
|---|---|---|
| `pwd` | Prints absolute path of current dir | — |
| `ls` | Lists directory contents | Hides dotfiles by default — need `-a` to see `.`, `..`, `.bashrc` etc. |
| `ls -l` | Long listing (perms, owner, size, date) | — |
| `ps` | Shows running processes | Plain `ps` = **current session only**, not whole system → use `ps -e` / `ps aux` |
| `uname -a` | Full system/kernel info | Use `-s -r -n -m` for specific parts |
| `clear` / `Ctrl+L` | Clears visible screen | Does **not** erase command history |
| `exit` / `Ctrl+D` | Closes the shell | In nested shells, only closes the innermost one |

---

## 4. `man` and Help Commands

| Command | Use case | ⚠️ Don't confuse with |
|---|---|---|
| `man cmd` | Full documentation | Some names span multiple sections (e.g. `printf` in §1 and §3) → use `man 3 printf` to force a section |
| `whatis` | One-line summary | Needs the **exact** command name |
| `apropos` | Keyword search across all docs | Works with a topic, can return multiple commands ← easy to mix up with `whatis` |
| `which` | Path of an **external** executable in `$PATH` | Won't find built-ins |
| `type` | Identifies builtin / alias / function / executable | More general than `which` — prefer this when unsure |
| `help cmd` | Docs for **shell built-ins only** (`cd`, `pwd`) | Fails on external commands like `ls` → use `man` instead |
| `info` | Hyperlinked, menu-based docs | — |

**Man page sections to memorize:**

| Section | Content |
|---|---|
| 1 | User commands |
| 2 | System calls |
| 3 | Library functions |
| 5 | File formats |
| 8 | Admin/system commands |

---

## 5. Filesystem Hierarchy Standard (FHS)

| Concept | Detail |
|---|---|
| `/` | Root — **is its own parent**; `cd /../../..` still lands you at `/` |
| Repeated slashes | `///home` = `/home` (collapsed, ignored) |
| Absolute path | Always starts with `/` |
| Relative path | Resolved from current directory |

**Key directories — remember by purpose:**

| Directory | Purpose | ⚠️ Edge case |
|---|---|---|
| `/etc` | Config files | Static + **unshareable** — machine-specific, never share across machines |
| `/var` | Data that changes constantly (logs, mail, cache) | — |
| `/tmp` | Temp files | **Cleared on reboot** |
| `/var/tmp` | Temp files | **Survives reboot** — easy to mix up with `/tmp` |
| `/usr` | Secondary hierarchy, mostly read-only software | Static + shareable |
| `/bin`, `/sbin`, `/lib` | Essential binaries | Must work even in single-user/recovery mode |
| `/proc`, `/sys` | Virtual, in-memory filesystems | No real disk space; generated live by kernel; gone on reboot |

**Static/Variable × Shareable/Unshareable matrix:**

| | Shareable | Unshareable |
|---|---|---|
| **Static** | `/usr`, `/opt` | `/etc`, `/boot` |
| **Variable** | `/var/mail` | `/var/run`, `/var/lock` |

⚠️ Common trap: people assume "shareable" and "variable" are opposites — they're independent axes (`/var/mail` is both).

---

## 6. Basic System Information Commands (safe, read-only)

| Command | Purpose | Example |
|---|---|---|
| `date` | Current date/time | `date +"%d-%m-%Y"` |
| `cal` | Calendar | `cal 8 2026` |
| `free -h` | Memory stats, human-readable | `free -h` |
| `groups` | User's group memberships | Useful for debugging "permission denied" errors |
| `file` | Detects type from **content**, not extension | Renaming `photo.png` → `photo.txt` won't fool it |

---

## 7. File Types, Inodes & Permissions

**File type symbol (1st char of `ls -l`):**

| Symbol | Type |
|---|---|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `c` | Character device |
| `b` | Block device |
| `s` | Socket |
| `p` | Named pipe (FIFO) |

| Concept | Detail |
|---|---|
| Inode | Stores metadata (perms, owner, size, timestamps) — **not the filename** (that's in the directory entry) |
| `ls -i` | Shows inode number |
| Permission string | `rwxr-xr-x` → owner / group / others (3 chars each) |
| Octal values | r=4, w=2, x=1 → e.g. `rwxr-xr-x` = `755` |
| ⚠️ Directory permissions | Needs **execute (x)**, not just read, to `cd` into or list it — `644` on a dir won't let you enter it |

---

## 8. File & Directory Management Commands

| Command | Purpose | ⚠️ Edge case |
|---|---|---|
| `chmod` | Change permissions (symbolic `u+x` or octal `755`) | `-R` applies recursively |
| `touch` | Update timestamp | **Creates an empty file if it doesn't exist**; never touches content |
| `cp` | Copy files | `-r` is **mandatory** for directories — plain `cp` fails on a dir |
| `cp` (overwrite) | Copies over existing destination | Overwrites **silently** by default — use `-i` for a prompt |
| `mv` | Move/rename | Handles directories **without** `-r` — this asymmetry with `cp` is a classic exam Q |
| `mkdir -p` | Create nested dirs | Does **not** error if parents already exist (plain `mkdir` does) |
| `rm -rf` | Force recursive delete | **Irreversible**, no confirmation — always double-check the path/wildcard first |

**Multi-argument rule for `cp`/`mv`:**

| # of arguments | Behavior |
|---|---|
| 2 | 2nd argument = destination |
| 3+ | Last argument **must be an existing directory**, or command errors out |

---

## 9. More `ls` Behavior

| Behavior | Detail |
|---|---|
| `ls dirname` | Lists dir's **contents**, not the dir entry itself |
| `ls -d dirname` | Lists the dir entry itself (handy: `ls -d */` → only subdirectories) |
| `ls -R` (uppercase) | **Recursive** listing into subdirectories |
| `ls -r` (lowercase) | **Reverse** order |
| ⚠️ | `-R` vs `-r` is a classic mix-up — case matters! |

---

## 10. Viewing File Contents

| Command | Purpose | ⚠️ Edge case |
|---|---|---|
| `cat` | Dumps whole file instantly | Bad for large files — no pausing |
| `more` | Pages forward only | Classic version — no backward scroll |
| `less` | Pages forward **and backward**, supports `/search` | "less is more" (more features than `more`) |
| `head` | First 10 lines by default | `-n N` or `-N` to change count |
| `tail` | Last 10 lines by default | `tail -f` follows live updates (for logs) |
| `wc` | Counts lines/words/bytes (`-l -w -c`) | Works on piped input too — no filename shown then |

---

## 11. Links: Hard vs Symbolic

| Feature | Hard Link | Symbolic (Soft) Link |
|---|---|---|
| Points to | Same **inode** (same data) | **Path/name** of target |
| Command | `ln target link` | `ln -s target link` |
| Works across filesystems | ❌ No | ✅ Yes |
| Can link directories | ❌ No | ✅ Yes |
| If original deleted | Data survives (link count > 0) | Becomes **dangling/broken** |
| `ls -l` shows | Normal `-` | `l` with `->` arrow |

⚠️ These are opposite failure modes — hard links never break but can't cross filesystems/link dirs; symlinks can do both but break if target moves/deletes.

---

## 12. File Sizes: `ls -s`, `stat`, `du`

| Command | Shows | ⚠️ Edge case |
|---|---|---|
| `ls -l` | Apparent/logical size | — |
| `ls -s` | Size in blocks | Rounded, not exact |
| `stat` | **Exact** byte size + full metadata | The only one giving true unrounded size |
| `du -sh` | Actual disk space used | Always rounds **up** to nearest block (commonly 4 KB) |

⚠️ A 3-byte file can still show `4.0K` in `du` — disk usage ≠ content size. `du` totals for a folder are usually **larger** than the sum of `ls -l` apparent sizes.

---

## 13. Master Quick-Reference Table

| Command | Purpose | Watch out for |
|---|---|---|
| `pwd` | Show current directory | — |
| `ls -a` | Show hidden files | Plain `ls` hides dotfiles |
| `ls -R` vs `ls -r` | Recursive vs reverse | Case matters! |
| `ps` vs `ps -e` | Session vs all processes | Plain `ps` is session-only |
| `man N cmd` | Docs for a specific section | Needed when a name spans sections |
| `cp -r` | Copy directories | `-r` mandatory, unlike `mv` |
| `mv` | Move/rename (dirs need no flag) | Overwrites destination silently |
| `rm -rf` | Force recursive delete | Irreversible — check path first |
| `mkdir -p` | Create nested dirs | Won't error if they already exist |
| `chmod` | Change permissions | Dir needs `x` bit to `cd`/list into it |
| `less` vs `more` | Both directions vs forward only | `less` also supports search |
| `head`/`tail -n` | First/last N lines | Default is 10 |
| `which` vs `type` | External-only vs also builtins/aliases | `type` is more general |
| `whatis` vs `apropos` | Exact name vs keyword search | — |
| Hard link vs symlink | Same inode vs path pointer | Symlinks can dangle; hard links can't cross filesystems |
| `ls -l`/`ls -s` vs `du`/`stat` | Apparent vs actual/exact size | Disk usage rounds up to block size |
| `/tmp` vs `/var/tmp` | Cleared vs preserved on reboot | Easy to mix up |
| `/proc`, `/sys` | Virtual, in-memory filesystems | No real disk space; generated live |

---

*Prepared as a consolidated, table-based revision summary for Week 1 (Lectures 2–4).*
