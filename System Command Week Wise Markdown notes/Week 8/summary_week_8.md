# Week 8 — Full Summary
**Lecture 1:** Scheduled/Recurring Script Automation (Cron, at, Startup, Runlevels)
**Lecture 2:** SED (Stream Editor)

---

# 📘 Lecture 1 — Automating Scripts

Linux automates tasks in two broad ways: **time-based** (cron, at) and **event-based** (startup/runlevel scripts tied to boot/shutdown).

| Mechanism | Triggered By | Repeats? |
|---|---|---|
| **Cron** | Specific clock time | Yes (recurring) |
| **at** | Specific clock time | No (once only) |
| **Startup / Runlevel scripts** | Boot / shutdown event | On every boot/shutdown |

## 1. Cron — Recurring Jobs

Cron is a background daemon (**`crond`**) that checks every minute whether any job in a crontab matches the current time, and runs it if so. Each user has a personal crontab; the system also has `/etc/crontab`.

**⚠️ Edge case:** if the machine is **powered off** at the scheduled time, a normal cron job is simply **skipped** — it does *not* run late. This is exactly the gap `anacron` fills.

| Location | Purpose |
|---|---|
| `/etc/crontab` | System-wide file; **requires** a user field |
| `/etc/cron.d/` | Extra system-wide job files (often from installed packages) |
| `/etc/cron.hourly` `.daily` `.weekly` `.monthly` | Drop a script here and it just runs at that frequency — no schedule line needed |

**⚠️ Edge case:** scripts dropped into `cron.daily`/etc. must be **executable** (`chmod +x`), otherwise `run-parts` silently **skips** them.

### Cron field order

```
minute   hour   day-of-month   month   day-of-week   [user-name]   command
```

| Field | Valid Range | Notes / Edge Cases |
|---|---|---|
| Minute | 0–59 | — |
| Hour | 0–23 | 24-hour format |
| Day of month | 1–31 | — |
| Month | **1–12** | ⚠️ Common exam trap: never 0. Names (jan–dec) also work. |
| Day of week | 0–6 | 0 = Sunday. ⚠️ Some systems also accept **7 as Sunday**. Names (sun–sat) also work. |
| user-name | valid username | ⚠️ Only appears in `/etc/crontab`/`cron.d`. A **personal** crontab (`crontab -e`) has **no user field**. |
| command | shell command/path | Output emailed to owner unless redirected (`>> log 2>&1`) |

### Special characters

| Symbol | Meaning | Example | Result |
|---|---|---|---|
| `*` | Every value | `* * * * *` | Every minute |
| `,` | List | `0 9,13,18 * * *` | 9 AM, 1 PM, 6 PM |
| `-` | Range | `0 9 * * 1-5` | 9 AM, Mon–Fri |
| `/` | Step | `*/5 * * * *` | Every 5 minutes |
| `,` + `-` | Mixed | `0 8 * * 1-5,0` | 8 AM, every day except Saturday |
| `/` + range | Mixed | `0 9-17/2 * * *` | Every 2 hrs between 9 AM–5 PM |

**Worked example:** `5 2 * * 1-5 root cd /home/scripts/backup && ./mkbackup.sh` → runs as `root`, **02:05 AM, Mon–Fri**.

| Tool | Function | Edge case |
|---|---|---|
| `anacron` | Runs periodic jobs even if the machine was off at the scheduled time | Good for laptops/desktops not always on |
| `logrotate` | Rotates/compresses log files | `missingok`/`notifempty` prevent errors on missing/empty logs |

## 2. `at` — One-Time Jobs

`at` schedules a command to run **only once**; after it runs, the job is auto-removed.

```bash
at 18:00
at> /home/scripts/run_report.sh
at> <Ctrl+D>
```
Also works with relative time (`at now + 10 minutes`), a future date, or piped input (`echo "cmd" | at 23:30`).

| Command | Purpose |
|---|---|
| `atq` (or `at -l`) | List pending `at` jobs |
| `atrm <job_number>` | Cancel a pending job |
| `at -c <job_number>` | View a job's full contents |

**⚠️ Edge case:** `at` jobs **cannot be edited** in place — you must `atrm` and resubmit. There's no persistent config file like crontab; `atd` stores jobs internally.

| Feature | `cron` | `at` |
|---|---|---|
| Recurs? | Yes | No — once only |
| Daemon | `crond` | `atd` |
| Config file | Crontab | None (internal queue) |
| Editing | Edit crontab entry | Not possible — `atrm` + resubmit |

## 3. Startup Scripts

Startup scripts run automatically **on boot or shutdown**, not at a clock time.

| Location | Init system | Purpose |
|---|---|---|
| `/etc/init/` | Upstart | `.conf` job files |
| `/etc/init.d/` | SysV | Traditional start/stop/restart/status scripts |

**⚠️ Edge case:** most modern distros use **systemd**, but `/etc/init.d/` scripts still work via systemd's **compatibility layer**.

Standard usage: `service_name start | stop | restart | status` (e.g. `sudo /etc/init.d/apache2 start`)

**Flow:** kernel → init system picks target **runlevel** → runs symlinked scripts in `/etc/rcX.d/` that point back to `/etc/init.d/`.

## 4. Runlevels

| Runlevel | Directory | Meaning | Edge case |
|---|---|---|---|
| **0** | `/etc/rc0.d/` | Shutdown / power off | — |
| **1** | `/etc/rc1.d/` | Single-user / rescue mode | Only root can log in |
| **2** | `/etc/rc2.d/` | Multi-user, no networking | Rarely used as default today |
| **3** | `/etc/rc3.d/` | Multi-user, networking, no GUI | Standard **server** mode |
| **4** | `/etc/rc4.d/` | Custom / undefined | ⚠️ Not standardized across distros — rarely used |
| **5** | `/etc/rc5.d/` | Multi-user, networking + GUI | Standard **desktop** mode |
| **6** | `/etc/rc6.d/` | Reboot | Runs shutdown scripts, then restarts |

Manual switching: `init 3/5/6/0`. Check current: `runlevel` → e.g. `N 5` (`N` = no previous runlevel).

## 5. `crontab -e`

The **safe** way to create/edit your personal crontab: opens `$EDITOR`, auto-validates and installs on save — no restart needed.

**⚠️ Edge case:** a personal crontab has **no `user-name` field** (unlike `/etc/crontab`).

| Command | Purpose |
|---|---|
| `crontab -e` | Edit (creates one if none exists) |
| `crontab -l` | List current jobs |
| `crontab -r` | Remove entire crontab (⚠️ no confirmation!) |
| `crontab -i` | Remove **with confirmation** (safer than `-r`) |
| `crontab -u <user> -e` | Edit another user's crontab (needs root/sudo) |
| `crontab <filename>` | Replace current crontab with a file's contents |

Backup/migrate: `crontab -l > backup.txt` then `crontab backup.txt` on the new machine.

### 🔑 Lecture 1 Cheat-Sheet

| Category | Key Points |
|---|---|
| Field order | `minute hour day-of-month month day-of-week [user] command` |
| Month range | 1–12 (never 0) |
| Day-of-week | 0–6, 0=Sunday (7 also = Sunday on some systems) |
| Special chars | `*` every, `,` list, `-` range, `/` step |
| Cron vs at | Cron = recurring; at = once, can't be edited |
| Startup vs cron | Event-triggered (boot/shutdown) vs. time-triggered |
| Runlevels | 0 shutdown · 1 rescue · 2 multi-user no net · 3 server · 4 rare/custom · 5 desktop · 6 reboot |

---

# 📗 Lecture 2 — SED (Stream Editor)

`sed` is a POSIX **stream editor** — a non-interactive text-processing language that reads input line-by-line and applies scripted transformations, unlike interactive editors like `vi`/`nano`.

**⚠️ Edge case (exam favorite):** `sed` predates `awk`; `sed` = simple line-based transformations, `awk` = fuller scripting/reporting language.

## 1. Execution Model

Each line goes through a cycle: **read → load into Pattern Space → run all script statements → auto-print (unless `-n`) → next line**.

| Buffer | Purpose | Default Content | Lifetime |
|---|---|---|---|
| **Pattern Space** | Active buffer holding the current line | Current input line | Refreshed every cycle (unless `N` used) |
| **Hold Space** | Auxiliary storage | Empty at start | Persists across cycles until changed |

**⚠️ Edge case:** the Hold Space is **not shown in output by default** — you must explicitly move data out of it (`g`, `G`, `x`) to see it.

## 2. Invoking SED

| Mode | Flag | Use Case |
|---|---|---|
| Command line | `-e` (optional if only one script) | Short, one-off transforms |
| Script file | `-f script.sed` | Long/reusable/complex scripts |
| Standalone executable | `#!/usr/bin/sed -f` shebang + `chmod +x` | Self-contained mini-program |

**⚠️ Edge case:** `sed -e "" file` (empty script) just prints the file unchanged, because auto-print is on by default — this is often used to *demonstrate* the default behavior before introducing `-n`.

## 3. Key Command-Line Options

| Option | Meaning | Edge case to remember |
|---|---|---|
| `-n` | Suppress auto-print — only explicit `p` output appears | Forgetting `-n` with `p` is the **#1 sed mistake** — `sed '3p' file` prints line 3 **twice** |
| `-E` / `-r` | Extended regex (ERE) — `+ ? | ()` work unescaped | Default (BRE) needs escaping: `\+`, `\?`, `\|`, `\(\)` |
| `-i` | Edit file **in place** | ⚠️ Always test without `-i` first; use `-i.bak` to keep a backup |

## 4. Statement Structure & Grouping

```
address pattern action
```
`;` separates commands · `,` defines a range · `!` negates a match · `:label` marks a branch target.

**Grouping:** `address { cmd; cmd; }` — needed whenever multiple actions share one address.
Example: `sed -n '2,4{s/foo/bar/; p}' file`

## 5. Addressing (heavily tested)

| Category | Syntax | Description |
|---|---|---|
| Exact line | `5` | Single specific line |
| Last line | `$` | Last line of file/stream |
| Step (GNU ext.) | `1~3` | Every 3rd line starting at 1 |
| Regex match | `/regexp/` | Any line matching a pattern |
| Numeric range | `5,15` | Between two fixed line numbers |
| Regex + offset | `/regexp/,+4` | Matched line plus next 4 lines |
| Regex-to-regex | `/re1/,/re2/` | Between two pattern matches — ⚠️ **inclusive of both boundaries** |
| Line-to-regex | `5,/regexp/` | From line number until pattern match |
| Regex-to-step-end | `/regexp/,~2` | Match until next line number multiple of N |
| Negation | `!` added to any address | Inverts the selection (e.g. `/error/!p` = lines without "error") |

**⚠️ Edge case:** if `START` appears multiple times in `/START/,/END/`, sed **restarts** the range logic after each completed range in the same pass.

## 6. Basic Actions

| Action | Description | Edge case |
|---|---|---|
| `p` | Print pattern space | Pair with `-n` or you get duplicates |
| `d` | Delete pattern space (skip auto-print) | No `-n` needed — deletion already prevents auto-print |
| `s/pat/repl/[flags]` | Substitute | Delimiter `/` can be swapped for any char (e.g. `#`) — handy for paths |
| `=` | Print current line number | Prints number **before** the line's own auto-printed content |
| `#` | Comment | ⚠️ `#n` as the **very first line** of a script = same as `-n` flag |
| `i` | Insert text **before** line | Original line **kept** |
| `a` | Append text **after** line | Original line **kept** |
| `c` | Change/replace line entirely | Original line **removed**; ⚠️ applied to a **range**, replaces the whole range with the text **once**, not once per line |

**`s` flags:** `g` = all occurrences · `p` = print if substituted · `i`/`I` = case-insensitive · `N` = replace only Nth match · `w file` = write result if substituted.

## 7. Programming Commands (Advanced)

| Command | Description | Edge case |
|---|---|---|
| `b label` | Unconditional branch | No label → jumps to end of script; can cause **infinite loops** if not paired with a condition |
| `:label` | Label definition | Target for `b`/`t`/`T` |
| `N` | Append next line to pattern space (joined by `\n`) | Gateway to multi-line processing (joining lines, pairs) |
| `q` | Quit immediately | Auto-prints current pattern space first unless `-n` |
| `t label` | Branch **if** last `s` succeeded | Used to build loops that **terminate** (unlike plain `b`) |
| `T label` | Branch **if** last `s` failed | Complement of `t` |
| `w filename` | Write pattern space to file | Writes **in addition to** normal output |
| `x` / `h` / `H` / `g` / `G` | Hold-space family: exchange / copy-to-hold / append-to-hold / copy-from-hold / append-from-hold | `h`/`g` **overwrite**; `H`/`G` **append** (adds `\n` first) |

**⚠️ Edge case:** `sed ':a; s/aa/a/; ba' file` loops **forever** because `b` is unconditional — use `t` instead for loops that must stop.

Classic idioms:
- Reverse a file (like `tac`): `sed -n '1!G;h;$p' file`
- Join all lines: `sed ':a;N;$!ba;s/\n/ /g' file`
- Print first 5 lines (like `head -n5`): `sed '5q' file`

## 8. SED with Bash

| Method | Use Case |
|---|---|
| Inside a shell script | Automate repetitive edits |
| Heredoc (`sed -f - file <<'EOF' ... EOF`) | Multi-line script inline, no separate `.sed` file |
| Pipe (`cat file \| sed '...' \| sort`) | Chain with other Unix tools |

## 9. Debugging

| Technique | Purpose |
|---|---|
| `sed --debug 'script' file` | GNU-only; shows each command executed + pattern/hold space state per cycle |
| `sed -n 'l' file` | Reveals hidden/non-printing chars (e.g. `\t`) — useful for invisible-whitespace bugs |
| Test without `-i` first | Preview before destructive edit |
| Build scripts incrementally | Easier to debug branch-heavy scripts |

**⚠️ Edge case:** `--debug` is a **GNU sed extension**, not POSIX — mention this if asked about portability.

### 🔑 Lecture 2 Cheat-Sheet

| Category | Key Symbols/Commands |
|---|---|
| Invocation | `-e` inline · `-f` script file · `-n` suppress auto-print · `-i` in-place |
| Address types | `5`, `$`, `1~3`, `/regexp/`, `5,15`, `/re1/,/re2/`, `5,/re/`, `/re/,+4`, `/re/,~2` |
| Basic actions | `p`, `d`, `s///`, `=`, `#`, `i`, `a`, `c` |
| Flow control | `b`, `:label`, `N`, `q`, `t`, `T`, `w`, `x` |
| Buffers | Pattern space (active) vs. Hold space (auxiliary, persists) |
| Special chars | `;` separator, `,` range, `!` negation, `{ }` grouping |
| Regex mode | BRE (default, escape `+?|()`) vs ERE (`-E`/`-r`, no escaping) |
