# Week 3 Revision Summary — Linux (Redirection, Software Management, Process Control)

> Covers: Lecture 1–2 (Combining Commands & Redirection), Lecture 3–4 (Software Management), Lecture 5 (Process Management).
> Edge cases are folded into each topic's explanation, not listed separately — read the "Watch out" notes inside each row/section.

---

## 1. Combining Commands (`;`, `&&`, `||`, `()`)

| Operator | Runs command2 when... | Edge case to remember |
|---|---|---|
| `;` | Always, regardless of command1's result | Even if command1 fails or doesn't exist (e.g. a typo like `lss`), command2 **still runs** — `;` never checks exit status. |
| `&&` | Only if command1 **succeeds** (exit code `0`) | If command1 fails, command2 is **skipped entirely** — chain breaks at the first failure. |
| `\|\|` | Only if command1 **fails** (exit code ≠ 0) | If command1 succeeds, command2 **never runs** — it's a fallback, not a second action. |
| `(cmd1; cmd2)` | Groups commands into a **subshell** | `cd`/variable changes made **inside `()` don't leak out** to the parent shell — `pwd` afterward still shows the original directory. Nested `()` create deeper subshells; check depth with `echo $BASH_SUBSHELL` (starts at `0` in the top-level shell, +1 per nesting level). |

**Remember:** `;`, `&&`, `||` can all be mixed with `&` and redirection on the same line — order matters (e.g. `cmd1 & echo "..."` runs cmd1 in background while echo runs immediately, not after).

---

## 2. File Descriptors & Redirection

| FD | Name | Default | Edge case |
|---|---|---|---|
| `0` | stdin | Keyboard | `cat` with no file argument reads from stdin — this is how `cat > file` lets you "type" a file (end with **Ctrl+D**). |
| `1` | stdout | Screen | `>` and `>>` only redirect stdout — **stderr still prints to the screen** unless separately redirected. |
| `2` | stderr | Screen | Piping (`\|`) only carries **stdout** to the next command — stderr of command1 is *not* piped, it still shows on screen. |

| Syntax | Behavior | Watch out |
|---|---|---|
| `cmd > file` | Overwrite stdout to file | **Destroys existing file content** silently — no warning. |
| `cmd >> file` | Append stdout to file | Creates the file if it doesn't exist; safe for logs. Can be chained: `cmd1 >> f; cmd2 >> f`. |
| `cmd 2> file` | Redirect only stderr | stdout still shows on screen. |
| `cmd > out 2> err` | stdout and stderr to **different** files | Both files are overwritten. |
| `cmd > file 2>&1` | Both streams to **same** file | **Order is critical**: `2>&1` must come *after* `> file`. Writing `2>&1 > file` sends stderr to the *old* stdout (terminal), not the file — classic exam trap. |
| `cmd < file` | stdin reads from file instead of keyboard | Command behaves as if you typed the file's contents. |
| `cmd > /dev/null 2>&1` | Fully silence a command | Common in scripts/cron; nothing is saved anywhere — output is gone for good. |

---

## 3. Piping and `tee`

| Command | Purpose | Edge case |
|---|---|---|
| `cmd1 \| cmd2` | Sends cmd1's stdout → cmd2's stdin | stderr of cmd1 bypasses the pipe (goes to screen). |
| `cmd1 \| cmd2 > file` | Pipe then redirect final output | Only cmd2's *final* output is saved; overwrites `file`. |
| `cmd \| tee file` | Show on screen **and** save to file simultaneously | Unlike `>`, output is **not hidden** — `tee` duplicates it. Default **overwrites** `file`; use `tee -a` to append. |
| `cmd \| tee f1 f2 \| cmd2` | Write to multiple files *while* continuing the pipeline | All files get identical copies; `cmd2` still receives the data downstream. |
| `cmd 2>/dev/null \| tee f1 f2 \| cmd2` | Discard errors + tee + continue pipeline | Order of operations: errors dropped first, then stdout is teed, then piped onward. |
| `diff file1 file2` | Compare two files | **No output = identical.** Beginners often expect a "match" message — silence *is* the success signal. |

---

## 4. Package Management — Concepts

| Family | Distros | Low-level tool | High-level tool |
|---|---|---|---|
| **RPM** | Red Hat, CentOS, Fedora, Oracle Linux, SUSE/openSUSE | `rpm` | `yum` (older) / `dnf` (modern) |
| **DEB** | Debian, Ubuntu, Mint, Knoppix | `dpkg`, `dpkg-deb` | `apt`, `aptitude`, `synaptic` |

**Watch out:** `dnf` is the modern replacement for `yum` — if asked "which is used in modern Fedora/RHEL," it's `dnf`, not `yum`.

### Architecture labels
| Label | Meaning |
|---|---|
| `amd64` / `x86_64` | 64-bit Intel/AMD |
| `i386` / `x86` | 32-bit Intel/AMD |
| `arm` | ARM processors |
| `ppc64el` | 64-bit little-endian PowerPC |
| `all` / `noarch` / `src` | Architecture-independent or source package |

Check your machine's architecture with `uname -m` or `arch` **before** manually downloading a `.deb`/`.rpm` — installing the wrong architecture package will fail.

### Package naming
| Format | Pattern | Example |
|---|---|---|
| RPM | `package-version-release.arch.rpm` | `httpd-2.4.6-97.el7.centos.x86_64.rpm` |
| DEB | `package_version-revision_arch.deb` | `firefox_115.0.2-1_amd64.deb` |

**Edge case:** DEB uses `_` as separator and `-revision` for the packaging revision (not the software version); RPM uses `-` throughout. Don't confuse "revision" (packaging change) with "version" (upstream software release).

### Priorities (Debian/Ubuntu)
`required` (never remove) → `important` → `standard` (default install) → `optional` → `extra` (may conflict, install only if needed).

### Permissions
Only **sudoers** can install/remove packages. Edit `/etc/sudoers` **only via `visudo`** — never a plain editor, since a broken sudoers file can lock everyone out of `sudo`. Running package commands without `sudo` gives a lock-file permission error, not a "command not found."

---

## 5. `apt` vs `dpkg` — the Most Important Exam Distinction

| Aspect | `apt` (high-level) | `dpkg` (low-level) |
|---|---|---|
| Dependency resolution | ✅ Automatic | ❌ **None** — you must install dependencies manually |
| Works from | Remote repositories | Local `.deb` files only |
| Recommended for install/remove? | ✅ Yes, always prefer | ⚠️ Only for direct `.deb` installs (`dpkg -i`); **removing packages via `dpkg` is discouraged** |
| Config location | `/etc/apt/sources.list`, `/etc/apt/sources.list.d` | `/var/lib/dpkg` |

**Key `apt-get` commands**
| Command | Effect |
|---|---|
| `apt-get update` | Refresh local package index (does **not** install/upgrade anything by itself — always run before `upgrade`/`install`) |
| `apt-get upgrade` | Upgrade installed packages |
| `apt-get install pkg` | Install a package |
| `apt-get remove pkg` | Remove package, **keep config files** |
| `apt-get purge pkg` | Remove package **and** config files |
| `apt-get autoremove` | Clean up orphaned dependency packages |
| `apt-get clean` | Clear downloaded package cache |

**Edge case:** `remove` vs `purge` is a favorite trick question — `remove` leaves `/etc` config files behind; `purge` deletes them too.

**Key `dpkg` commands**
| Command | Effect |
|---|---|
| `dpkg -l [pattern]` | List installed packages (flags: `ii`=installed, `rc`=removed-but-configs-remain, `un`=unknown, `iU`=unpacked-not-configured) |
| `dpkg -L pkg` | List files a package installed |
| `dpkg -s pkg` | Show status/metadata block |
| `dpkg -S /path` | Find which package owns a file |
| `dpkg -i file.deb` | Install a local `.deb` directly |

**Edge case on status flags:** `rc` does **not** mean "running config" — it means the package was removed but its config files are still on disk (opposite of what people assume). Only `apt-get purge` clears that `rc` state.

### Finding any package property (the universal exam trick)
> Almost every property (version, architecture, priority, section, dependencies, size, status) is just a labelled field inside `apt-cache show pkg` (repo package) or `dpkg -s pkg` (installed package). Pipe to `grep -i "fieldname"`, or extract cleanly with `dpkg-query -W -f='${FieldName}\n' pkg`.

| Property | Best command |
|---|---|
| Version | `apt-cache show pkg \| grep Version` |
| Architecture | `dpkg --print-architecture` (system) or `grep Architecture` (package) |
| Dependencies | `apt-cache depends pkg` |
| Installed files | `dpkg -L pkg` |
| Owner of a file | `dpkg -S /path` |
| Checksum | `md5sum` / `sha256sum` on the `.deb` file |

**Edge case:** `dpkg --print-architecture` reports the **system's** architecture, not a specific package's — don't confuse it with `dpkg -s pkg | grep Architecture`, which reports the package's.

### Logs
`/var/log/apt/history.log` (high-level apt actions) vs `/var/log/dpkg.log` (low-level per-package state changes) — know which log answers "what did apt do" vs "what did dpkg record."

---

## 6. Process Management — Foreground, Background, Jobs

| Command/Key | Effect | Edge case |
|---|---|---|
| `cmd &` | Run in background, shell returns immediately | Shell prints `[job#] PID` right away — note both numbers, `%N` uses the *job number*, `kill` uses the *PID*. |
| `fg [%N]` | Bring a job to foreground | Terminal is now blocked until it finishes or is suspended. |
| `bg [%N]` | Resume a **suspended** job in the background | Only works on jobs already stopped (e.g. via Ctrl+Z). |
| `jobs [-l/-r/-s]` | List jobs of the **current shell session only** | Won't show processes from other terminals — that's what `ps`/`top` are for. `+` = current job, `-` = previous job. |
| `Ctrl+C` | Sends `SIGINT` to the **foreground** process | Terminates it — but only if the program doesn't ignore SIGINT (some do; then you need `kill -9`). |
| `Ctrl+Z` | Sends `SIGTSTP` — **suspends**, does NOT kill | Common confusion: Ctrl+Z ≠ Ctrl+C. Process stays in memory; resume with `fg`/`bg`. |
| `Ctrl+D` | Sends EOF, not a signal | In an interactive shell = exit; in `cat`/`bc` = "no more input." Different from typing `exit`, though effect is similar for a shell. |

**Two ways to kill — comparison**
| | Ctrl+C | `kill` |
|---|---|---|
| Target | Only current foreground process | Any process, any terminal, via PID/job |
| Signal | SIGINT only | Any signal, default SIGTERM (15) |
| Cross-terminal | ❌ No | ✅ Yes |

**Edge case:** to kill a process running in *another* terminal, Ctrl+C won't work at all (keyboard signals only affect that terminal's foreground process) — you must `ps -ef | grep name` to find the PID, then `kill -9 PID`.

### `kill` signals
| Signal | # | Meaning |
|---|---|---|
| SIGHUP | 1 | Reload config |
| SIGINT | 2 | Same as Ctrl+C |
| SIGKILL | 9 | Force kill — **cannot be ignored/caught** |
| SIGTERM | 15 | Default, graceful — **can** be ignored by a program |
| SIGSTOP | 19 | Pause — cannot be ignored |
| SIGCONT | 18 | Resume |

**Edge case:** `kill` doesn't always kill — it *sends a signal*; SIGTERM (default) can be trapped/ignored by well-behaved programs, so a stuck process sometimes needs `kill -9` (SIGKILL, which the OS enforces unconditionally).

### `ps` and `top`
| Command | Scope |
|---|---|
| `ps` | Current shell's processes only |
| `ps -e` / `ps -ef` | **Every** process on the system |
| `ps -e --forest` | Process tree (parent-child) |
| `top` | Live, interactive monitor (`q` quit, `k` kill, `P`/`M` sort by CPU/Mem) |

### Exit codes
| Code | Meaning |
|---|---|
| `0` | Success |
| `1` | General error |
| `2` | Misuse of shell command |
| `126` | Found but not executable |
| `127` | Command not found |
| `130` | Killed by Ctrl+C (SIGINT) |
| `137` | Killed by SIGKILL (128+9) |

**Edge case:** `$?` always reflects the **last executed command** — if you run `echo $?` twice, the second call reports the exit code of the *first* `echo $?` (which is `0`), not the original command anymore. For background/child processes, use `$!` (PID of last background job) with `wait $!` to capture the *child's* exit code into `$?` before it gets overwritten.

### Other shell features
| Feature | Concept | Edge case |
|---|---|---|
| `echo $-` | Shows active shell option flags (e.g. `himBHs`) | `H` = history expansion on (enables `!!`/`!n`); `m` = job control on (enables `fg`/`bg`/`jobs`) — if these flags are missing, those features won't work. |
| Subshell (`bash` or `()`) | Child shell inherits env but changes don't propagate back | Confirm with `echo $$` (PID differs) or `pstree`. |
| `!n` | Re-run history line `n` | Numbers shift as history grows — always re-check with `history` first. |
| `!!` | Re-run last command | Classic use: `sudo !!` after a permission-denied error. |
| Brace expansion `{a,b,c}` / `{1..5}` | Pure **text generation**, happens before execution — not a loop | `{10..0..2}` supports a step and can count *down*. `mkdir proj{Front,Back}` creates two dirs in one line. |
| `bc` | CLI calculator (bash itself only does integer math) | Division truncates by default (`10/3` → `3`); use `scale=2; 10/3` for decimals. |
| `coproc` | Background command with auto-created pipes for 2-way I/O | Advanced/rare — mainly for scripted inter-process communication, not everyday use. |

---

## Quick Cross-Topic Traps to Double-Check Before the Exam

- `2>&1` placement: must come **after** `> file`, or stderr goes to the terminal instead of the file.
- `;` runs everything regardless of failure — `&&`/`||` are the ones that check exit status.
- `remove` keeps config files, `purge` deletes them too.
- `dpkg` never resolves dependencies — always prefer `apt` for install/remove.
- `diff` with no output means files are **identical**, not "nothing happened."
- Ctrl+C kills, Ctrl+Z suspends, Ctrl+D sends EOF — three different signals/behaviors, frequently mixed up.
- `jobs`/`fg`/`bg` only see the **current shell's** jobs — use `ps`/`kill` for processes elsewhere.
- `$?` is overwritten by every command you run, including `echo $?` itself — capture it into a variable immediately if you need it later.
