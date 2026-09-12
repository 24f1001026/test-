# Automating Scripts — Week 8, Lecture 1
### Topic: Scheduled, Recurring, and Automatic Execution of Scripts in Linux

---

## Table of Contents

1. [Introduction to Script Automation](#1-introduction-to-script-automation)
2. [Cron and `at` Commands](#2-cron-and-at-commands-0030)
   - [2.1 What is Cron?](#21-what-is-cron)
   - [2.2 The `at` Command](#22-the-at-command)
   - [2.3 Cron vs. `at` — Key Differences](#23-cron-vs-at--key-differences)
   - [2.4 Other Related Tools](#24-other-related-tools)
   - [2.5 Cron Script Locations](#25-cron-script-locations)
3. [Job Definition in Cron (02:00)](#3-job-definition-in-cron-0200)
   - [3.1 Field-by-Field Breakdown](#31-field-by-field-breakdown)
   - [3.2 Field Value Reference Table](#32-field-value-reference-table)
   - [3.3 Special Characters Used in Cron Fields](#33-special-characters-used-in-cron-fields)
   - [3.4 Worked Example from the Lecture](#34-worked-example-from-the-lecture)
   - [3.5 Examples of Every Special-Character Type](#35-examples-of-every-special-character-type)
   - [3.6 Additional Practice Examples](#36-additional-practice-examples)
4. [Startup Scripts (05:00)](#4-startup-scripts-0500)
   - [4.1 Concept](#41-concept)
   - [4.2 Startup Script Locations](#42-startup-script-locations)
   - [4.3 How Startup Scripts Actually Run](#43-how-startup-scripts-actually-run)
   - [4.4 Examples](#44-examples)
5. [Runlevel Scripts](#5-runlevel-scripts)
   - [5.1 Concept](#51-concept)
   - [5.2 Runlevel Reference Table](#52-runlevel-reference-table)
   - [5.3 Explanation of Each Runlevel](#53-explanation-of-each-runlevel)
6. [`crontab -e` — Editing Cron Jobs (10:00)](#6-crontab--e--editing-cron-jobs-1000)
   - [6.1 Concept](#61-concept)
   - [6.2 All `crontab` Command Options](#62-all-crontab-command-options)
   - [6.3 Step-by-Step: Using `crontab -e`](#63-step-by-step-using-crontab--e)
   - [6.4 Examples](#64-examples)
7. [Summary](#7-summary)
8. [Quick Revision Cheat-Sheet](#8-quick-revision-cheat-sheet)

---

## 1. Introduction to Script Automation

In Linux system administration, many maintenance tasks (backups, log cleanup, report generation, system checks) need to run **without manual intervention**, either:

- At a **specific scheduled time** (once or repeatedly), or
- **Automatically at system startup/shutdown**, depending on the system's operating state.

Linux provides two broad mechanisms for this:

| Mechanism | Purpose |
|---|---|
| **Cron / at** | Time-based scheduling of scripts/commands (recurring or one-time execution) |
| **Startup / Runlevel Scripts** | Execution of scripts tied to system boot, shutdown, or a specific operating mode (runlevel) |



---

## 2. Cron and `at` Commands

### 2.1 What is Cron?

**Cron** is a background service (daemon) in Linux/Unix systems that allows scripts or commands to be executed **automatically at scheduled times**. It is the standard tool for **time-based, recurring job scheduling** in Linux.

**Key concepts:**
- Cron runs continuously in the background as a daemon called **`crond`**.
- It reads job definitions from special files called **crontabs** (cron tables).
- Each **user** can have their own crontab, and the **system** also has its own (`/etc/crontab`).
- Cron is ideal for tasks that must repeat: hourly, daily, weekly, monthly, or on a fully custom schedule.
- If the system is **powered off** at the scheduled time, a normal cron job is simply **skipped** — it does not run late (this is the gap that `anacron` fills; see Section 2.4).

**How cron works internally (process flow):**
1. The `crond` daemon starts at boot and runs continuously.
2. Every minute, `crond` wakes up and checks all crontabs (`/etc/crontab`, `/etc/cron.d/*`, and each user's personal crontab).
3. If the current minute/hour/date matches any job's schedule, that job's command is executed.
4. Output (if any) is normally emailed to the crontab owner unless redirected (e.g., `>> logfile.txt 2>&1`).

**Basic example — system-wide cron job:**
```
5 2 * * 1-5 root cd /home/scripts/backup && ./mkbackup.sh
```
This runs a backup script as `root`, every weekday at 2:05 AM (fully explained in Section 3.4).

### 2.2 The `at` Command

The **`at`** command schedules a command or script to run **only once**, at a specific future date/time. Unlike cron, it is **not recurring** — after execution, the job is removed automatically.

**Basic syntax:**
```bash
at [TIME]
```
After pressing Enter, you are dropped into an `at>` prompt where you type the command(s) to run, then press **Ctrl+D** to save and exit.

**Example 1 — Run a script at a specific clock time today:**
```bash
at 18:00
at> /home/scripts/run_report.sh
at> <Ctrl+D>
```
> Runs `run_report.sh` once, today at 6:00 PM.

**Example 2 — Run a command after a relative delay:**
```bash
at now + 10 minutes
at> /home/scripts/send_reminder.sh
at> <Ctrl+D>
```
> Runs `send_reminder.sh` once, 10 minutes from now.

**Example 3 — Schedule for a specific future date:**
```bash
at 09:00 25.12.2026
at> /home/scripts/holiday_greeting.sh
at> <Ctrl+D>
```
> Runs the greeting script once, at 9:00 AM on 25 December 2026.

**Example 4 — Piping a command directly (no interactive prompt):**
```bash
echo "/home/scripts/cleanup.sh" | at 23:30
```
> Schedules `cleanup.sh` to run once at 11:30 PM, without opening the interactive `at>` prompt.

**Useful `at`-related commands:**

| Command | Purpose |
|---|---|
| `atq` | List all pending `at` jobs for the current user (the "at queue") |
| `atrm <job_number>` | Remove/cancel a pending `at` job by its job number |
| `at -l` | Alternative way to list pending jobs (same as `atq`) |
| `at -c <job_number>` | Display the full contents/commands of a scheduled `at` job |

**Example — Cancelling a job:**
```bash
atq
# Output: 3   Thu Aug 27 18:00:00 2026 a claude
atrm 3
```
> Lists pending jobs, finds job number `3`, then removes it before it runs.

### 2.3 Cron vs. `at` — Key Differences

| Feature | `cron` | `at` |
|---|---|---|
| Recurrence | Repeats on a schedule | Runs **only once** |
| Best use case | Regular maintenance tasks (backups, log rotation) | One-off future tasks (a single reminder, a one-time report) |
| Configuration file | Crontab (`crontab -e`, `/etc/crontab`) | No persistent file; queued jobs stored internally by the `atd` daemon |
| Daemon | `crond` | `atd` |
| Editing existing job | Edit the crontab entry | Cannot edit — must `atrm` and resubmit |

### 2.4 Other Related Tools

| Tool | Function |
|---|---|
| `at` | Schedules a command/script to run **once** at a specified future time (not recurring) |
| `crontab` | Command used to **create, edit, list, or remove** a user's cron jobs |
| `anacron` | Ensures periodic jobs run even if the system was **powered off** during the scheduled time (useful for laptops/desktops that aren't always on) |
| `logrotate` | Automatically **rotates, compresses, and manages log files** so they don't grow indefinitely; usually triggered via cron |

**Example — `anacron` entry (in `/etc/anacrontab`):**
```
# period(days)  delay(minutes)  job-identifier   command
7               25              weekly-backup    /scripts/weekly_backup.sh
```
> Runs `weekly_backup.sh` roughly every 7 days, with a 25-minute delay after boot if the system was off at the exact scheduled time — ensuring the job still runs eventually instead of being skipped.

**Example — `logrotate` configuration (in `/etc/logrotate.d/myapp`):**
```
/var/log/myapp/*.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
}
```
> Rotates `myapp` log files **daily**, keeps the last **7** rotated copies, **compresses** old logs, and does not error out if a log file is missing or empty.

### 2.5 Cron Script Locations

Cron looks for job definitions in multiple locations depending on scope:

| Location | Purpose |
|---|---|
| `/etc/crontab` | System-wide crontab file; requires specifying the **user** who runs the job |
| `/etc/cron.d/` | Directory for additional system-wide cron job files (often used by installed packages) |
| `/etc/cron.hourly/` | Scripts placed here run **once every hour** |
| `/etc/cron.daily/` | Scripts placed here run **once every day** |
| `/etc/cron.weekly/` | Scripts placed here run **once every week** |
| `/etc/cron.monthly/` | Scripts placed here run **once every month** |

> **Concept:** The `/etc/cron.hourly`, `/etc/cron.daily`, `/etc/cron.weekly`, and `/etc/cron.monthly` directories don't need a schedule written manually — you simply **drop a script inside them**, and the system's cron configuration (via `/etc/crontab` and the `run-parts` utility) executes every script in that folder at the implied frequency.

**Example — Making a script run daily without writing a custom schedule:**
```bash
sudo cp cleanup_temp.sh /etc/cron.daily/
sudo chmod +x /etc/cron.daily/cleanup_temp.sh
```
> Any script placed in `/etc/cron.daily/` must be **executable** (`chmod +x`) or `run-parts` will skip it.

---

## 3. Job Definition in Cron 

### 3.1 Field-by-Field Breakdown

A single line in a crontab (such as `/etc/crontab`) follows this structure:

```
minute   hour   day-of-month   month   day-of-week   user-name   command
```

| Field | Allowed Values | Meaning |
|---|---|---|
| **minute** | 0–59 | Minute of the hour the job should run |
| **hour** | 0–23 | Hour of the day (24-hour format) the job should run |
| **day of month** | 1–31 | Day of the month the job should run |
| **month** | 1–12 or `jan, feb, ...` | Month the job should run (numeric or 3-letter name) |
| **day of week** | 0–6 or `sun, mon, ...` | Day of the week the job should run (0 = Sunday) |
| **user-name** | valid system username | The user account under which the command executes *(only used in `/etc/crontab` and `/etc/cron.d/`, not in per-user `crontab -e`)* |
| **command** | any valid shell command/script path | The actual command or script to execute |

> **Correction/Clarification:** The source material listed month as `(0-12)`. The correct standard cron range for month is **1–12** (there is no month `0`). This has been fixed in the table above to avoid an exam error.

> **Important distinction:** In a **personal** crontab (edited with `crontab -e`), there is **no `user-name` field** — the job automatically runs as the currently logged-in user. The `user-name` field only appears in **system-wide** files like `/etc/crontab` and `/etc/cron.d/*`, because those files can define jobs for *any* user, so cron needs to be told explicitly who should run each line.

### 3.2 Field Value Reference Table

| Field | Numeric Range | Named Equivalent (if supported) |
|---|---|---|
| Minute | 0–59 | — |
| Hour | 0–23 | — |
| Day of Month | 1–31 | — |
| Month | 1–12 | jan, feb, mar, apr, may, jun, jul, aug, sep, oct, nov, dec |
| Day of Week | 0–6 (0 = Sunday, 6 = Saturday; 7 is also accepted as Sunday on some systems) | sun, mon, tue, wed, thu, fri, sat |

### 3.3 Special Characters Used in Cron Fields

| Symbol | Meaning | Example | Interpretation |
|---|---|---|---|
| `*` | "Every" value / any value | `* * * * *` | Every minute of every hour of every day |
| `,` | List of values | `1,15,30 * * * *` | At minutes 1, 15, and 30 of every hour |
| `-` | Range of values | `1-5` in day-of-week | Monday through Friday |
| `/` | Step values (used with `*` or a range) | `*/10 * * * *` | Every 10 minutes |

> These symbols aren't explicitly written in the source slide but are **standard cron syntax** every student should know, since exam questions commonly test range (`-`) and step (`/`) operators — as seen in the lecture's own example (`1-5` for weekdays).

### 3.4 Worked Example from the Lecture

```
5 2 * * 1-5 root cd /home/scripts/backup && ./mkbackup.sh
```

**Field-by-field decoding:**

| Field | Value | Meaning |
|---|---|---|
| minute | `5` | At minute 5 |
| hour | `2` | At hour 2 (2 AM) |
| day of month | `*` | Every day of the month |
| month | `*` | Every month |
| day of week | `1-5` | Monday to Friday (working days) |
| user-name | `root` | Runs with root privileges |
| command | `cd /home/scripts/backup && ./mkbackup.sh` | Navigate to the backup directory, then execute the backup script |

**Plain-English meaning:**
> Run `mkbackup.sh` as the `root` user every working day (Monday–Friday) at **02:05 AM**.

### 3.5 Examples of Every Special-Character Type

Since exam questions often test *each type of operator individually*, here is one clear, standalone example for every special character:

| Operator Type | Cron Line | Meaning |
|---|---|---|
| **`*` (Every value)** | `* * * * * root /scripts/heartbeat.sh` | Runs **every single minute**, every hour, every day |
| **`,` (List)** | `0 9,13,18 * * * root /scripts/attendance.sh` | Runs at **9:00 AM, 1:00 PM, and 6:00 PM** every day |
| **`-` (Range)** | `0 9 * * 1-5 root /scripts/office_open.sh` | Runs at **9:00 AM, Monday through Friday** only |
| **`/` (Step)** | `*/5 * * * * root /scripts/monitor.sh` | Runs **every 5 minutes**, all day, every day |
| **Combined (`,` + `-`)** | `0 8 * * 1-5,0 root /scripts/mixed_days.sh` | Runs at **8:00 AM** on **Mon–Fri and Sunday** (i.e., every day except Saturday) |
| **Combined (`/` + range)** | `0 9-17/2 * * * root /scripts/office_hours_check.sh` | Runs **every 2 hours between 9 AM and 5 PM** (9, 11, 13, 15, 17) |

### 3.6 Additional Practice Examples

| Cron Expression | Meaning |
|---|---|
| `0 0 * * * root /scripts/daily_report.sh` | Run `daily_report.sh` as root every day at **midnight (00:00)** |
| `30 6 * * 0 root /scripts/weekly_cleanup.sh` | Run `weekly_cleanup.sh` as root every **Sunday at 6:30 AM** |
| `0 */2 * * * root /scripts/health_check.sh` | Run `health_check.sh` as root **every 2 hours** |
| `0 9 1 * * root /scripts/monthly_invoice.sh` | Run `monthly_invoice.sh` as root at **9:00 AM on the 1st day of every month** |
| `*/15 * * * * root /scripts/ping_test.sh` | Run `ping_test.sh` as root **every 15 minutes** |
| `0 22 * 12 * root /scripts/year_end_backup.sh` | Run `year_end_backup.sh` as root **every day at 10 PM, but only in December** |

---

## 4. Startup Scripts 

### 4.1 Concept

**Startup scripts** are scripts that Linux executes automatically as part of the **boot process**, before or during the transition into a particular system state (runlevel/target). These handle tasks like starting services, mounting drives, initializing networking, or launching daemons — **without** requiring cron's time-based scheduling, because they are triggered by a system **event** (boot/shutdown) rather than a **clock time**.

Startup scripts answer the question *"What should run **when the system turns on or off**?"*, whereas cron answers *"What should run **at this exact time**, repeatedly?"*

### 4.2 Startup Script Locations

| Location | Purpose |
|---|---|
| `/etc/init/` | Directory used by the **Upstart** init system to store job configuration files (`.conf` files) |
| `/etc/init.d/` | Directory containing traditional **SysV init scripts** used to start, stop, restart, and check the status of services |

> **Concept Note:** Modern distributions largely use **systemd** instead of Upstart/SysV init, but `/etc/init.d/` scripts are often still supported for **backward compatibility** (systemd includes a compatibility layer that can run legacy SysV scripts).

### 4.3 How Startup Scripts Actually Run

1. The **kernel** finishes loading and hands control to the **init system** (SysV init, Upstart, or systemd depending on the distribution).
2. The init system determines the **target runlevel** (see Section 5) the machine should boot into.
3. For that runlevel, init runs the appropriate scripts — typically symbolic links inside `/etc/rcX.d/` that point back to the real scripts in `/etc/init.d/`.
4. Each SysV script in `/etc/init.d/` typically supports standard arguments:

```bash
service_name start     # Start the service
service_name stop      # Stop the service
service_name restart   # Stop then start again
service_name status    # Check whether it's running
```

### 4.4 Examples

**Example 1 — Manually starting a service via its init script:**
```bash
sudo /etc/init.d/apache2 start
```
> Starts the Apache web server using its SysV init script.

**Example 2 — Restarting a network service:**
```bash
sudo /etc/init.d/networking restart
```
> Stops and then restarts networking — commonly needed after changing `/etc/network/interfaces`.

**Example 3 — Checking a service's status:**
```bash
sudo /etc/init.d/ssh status
```
> Reports whether the SSH daemon is currently running.

**Example 4 — An Upstart job file (`/etc/init/myapp.conf`):**
```
description "My custom background application"
start on runlevel [2345]
stop on runlevel [016]
exec /usr/local/bin/myapp
```
> Tells Upstart to **start** `myapp` automatically when entering runlevels 2, 3, 4, or 5, and to **stop** it when entering runlevels 0, 1, or 6.

---

## 5. Runlevel Scripts

### 5.1 Concept

A **runlevel** defines the **operating state** of a Linux system — essentially, which services and features are active. Each runlevel has a **dedicated directory** containing the scripts (as symlinks back to `/etc/init.d/`) that should run when the system enters that state.

### 5.2 Runlevel Reference Table

| Runlevel | Directory | Description |
|---|---|---|
| **0** | `/etc/rc0.d/` | Shutdown and power off |
| **1** | `/etc/rc1.d/` | Single user mode |
| **2** | `/etc/rc2.d/` | Non-GUI multi-user mode **without** networking |
| **3** | `/etc/rc3.d/` | Non-GUI multi-user mode **with** networking |
| **4** | `/etc/rc4.d/` | Non-GUI multi-user mode for special/custom purposes |
| **5** | `/etc/rc5.d/` | GUI multi-user mode with networking |
| **6** | `/etc/rc6.d/` | Shutdown and reboot |

### 5.3 Explanation of Each Runlevel

Since the runlevel table packs several distinct concepts into one row each, here is an **expanded explanation** of every runlevel individually:

| Runlevel | Expanded Explanation |
|---|---|
| **0 – Halt** | The system executes all shutdown scripts in `/etc/rc0.d/`, cleanly stops services, unmounts filesystems, and **powers off** the machine. |
| **1 – Single User Mode** | Also called **rescue mode** or **maintenance mode**. Only the root user can log in, and networking/most services are disabled. Used for system repair, password resets, and emergency maintenance. |
| **2 – Multi-user, No Networking** | Multiple users can log in via terminals, but network services are **not started**. Rarely used as a default in most modern distros. |
| **3 – Multi-user, With Networking (No GUI)** | The standard **text-mode/server** runlevel. Multiple users can log in, and networking is fully active, but no graphical desktop is loaded. Common on Linux servers. |
| **4 – Custom/Undefined** | Reserved for **user-defined** configurations; not standardized across distributions. Rarely used in practice. |
| **5 – Multi-user, With Networking and GUI** | The standard **desktop** runlevel. Full networking plus a graphical login/desktop environment (X11/Wayland) is loaded. |
| **6 – Reboot** | Executes shutdown scripts (similar to runlevel 0) and then **restarts** the system instead of powering it off. |

**Example — Manually changing runlevel (legacy SysV systems):**
```bash
init 3      # Switch to multi-user mode with networking, no GUI
init 5      # Switch to GUI multi-user mode
init 6      # Reboot the system
init 0      # Shut down the system
```

**Example — Checking the current runlevel:**
```bash
runlevel
# Output: N 5
```
> `N` means "no previous runlevel" (system just booted), and `5` is the **current** runlevel (GUI desktop mode).

---

## 6. `crontab -e` — Editing Cron Jobs 

### 6.1 Concept

**`crontab -e`** is the standard, safe way to **create or edit** the cron jobs belonging to the **currently logged-in user**. Rather than manually editing a raw file, `crontab -e` opens the user's personal crontab in a text editor, and — crucially — **automatically validates and installs** the file when saved, so cron picks up the changes immediately without needing a service restart.

**Why use `crontab -e` instead of editing a file directly?**
- It opens the crontab in your **default editor** (commonly `nano` or `vi`, configurable via the `EDITOR` environment variable).
- On saving, `crontab` performs a **basic syntax check** and installs the file to the correct internal location (typically `/var/spool/cron/crontabs/<username>`).
- It avoids the risk of corrupting cron's internal spool directory by editing it directly with a text editor.
- Per-user crontabs edited this way do **not** include the `user-name` field, since the file itself is already scoped to one user.

### 6.2 All `crontab` Command Options

| Command | Purpose |
|---|---|
| `crontab -e` | **Edit** the current user's crontab (creates one if it doesn't exist) |
| `crontab -l` | **List** (display) the current user's crontab contents |
| `crontab -r` | **Remove** the current user's entire crontab |
| `crontab -i` | Remove the crontab, but **prompt for confirmation** first (safer than `-r`) |
| `crontab -u <username> -e` | Edit **another user's** crontab (requires root/sudo privileges) |
| `crontab <filename>` | **Replace** the current crontab with the contents of the given file |

### 6.3 Step-by-Step: Using `crontab -e`

1. **Open the editor:**
   ```bash
   crontab -e
   ```
2. **First-time use** — you may be prompted to choose a default editor:
   ```
   no crontab for claude - using an empty one

   Select an editor.  To change later, run 'select-editor'.
     1. /bin/nano        <---- easiest
     2. /usr/bin/vim.basic
   Choose 1-2 [1]:
   ```
3. **Add a new job line** at the bottom of the file, following the cron syntax from Section 3:
   ```
   0 3 * * * /home/claude/scripts/backup.sh
   ```
   (Notice: **no `user-name` field** here — it's a personal crontab.)
4. **Save and exit:**
   - In `nano`: press `Ctrl+O` (write out), then `Enter`, then `Ctrl+X` (exit).
   - In `vi`/`vim`: press `Esc`, then type `:wq`, then `Enter`.
5. **Confirmation message:**
   ```
   crontab: installing new crontab
   ```
6. **Verify it was saved correctly:**
   ```bash
   crontab -l
   ```

### 6.4 Examples

**Example 1 — Add a job to run a backup script every day at 3 AM:**
```bash
crontab -e
```
Add this line, save, and exit:
```
0 3 * * * /home/claude/scripts/backup.sh
```

**Example 2 — List all current cron jobs for the logged-in user:**
```bash
crontab -l
```
Sample output:
```
0 3 * * * /home/claude/scripts/backup.sh
*/30 * * * * /home/claude/scripts/sync_files.sh
```

**Example 3 — Remove all cron jobs for the current user (with confirmation):**
```bash
crontab -i
```
```
crontab: really delete claude's crontab? (y/n) y
```

**Example 4 — Edit another user's crontab as root:**
```bash
sudo crontab -u deploy -e
```
> Opens the crontab belonging to the `deploy` user — useful when an admin needs to schedule jobs for a service account.

**Example 5 — Restore/replace a crontab from a backup file:**
```bash
crontab -l > my_cron_backup.txt      # Step 1: back up current jobs
crontab my_cron_backup.txt           # Step 2: restore/replace from that file
```
> First exports existing jobs to a file, then reloads them — useful for migrating cron jobs to a new server.

---

## 7. Summary

- **Cron** is the core Linux service for **scheduled, recurring, automatic execution** of scripts, run continuously by the `crond` daemon and configured through **crontab** files.
- The **`at`** command complements cron by scheduling **one-time** future jobs (managed by the `atd` daemon), and is best combined with `atq` (list) and `atrm` (cancel) for job management.
- Supporting tools include **`anacron`** (catches up on missed jobs when a machine was powered off) and **`logrotate`** (automatically manages log file size and history), both of which reinforce how automation reduces manual system maintenance.
- Every cron job line follows a fixed field order — **minute → hour → day of month → month → day of week → [user] → command** — where month correctly ranges from **1–12** (not 0–12) and day of week from **0–6**.
- Cron's flexibility comes from four special characters: **`*`** (any value), **`,`** (list), **`-`** (range), and **`/`** (step) — all demonstrated individually and in combination in Section 3.5.
- **Startup scripts**, located in `/etc/init/` (Upstart) and `/etc/init.d/` (SysV), run automatically at **boot/shutdown events** rather than at scheduled clock times, typically supporting `start`, `stop`, `restart`, and `status` actions.
- **Runlevels (0–6)** define distinct system operating states — from complete shutdown (0) and rescue mode (1), through server mode (3), to full GUI desktop mode (5) and reboot (6) — each governed by its own `/etc/rcX.d/` directory of scripts linked back to `/etc/init.d/`.
- **`crontab -e`** is the safe, standard command for creating and editing a **personal** (per-user) crontab, automatically validating and installing changes — complemented by `-l` (list), `-r` (remove), `-i` (remove with confirmation), and `-u` (edit another user's crontab as root).

---

## 8. Quick Revision Cheat-Sheet

**Cron field order (memorize this):**
```
*     *     *     *     *     command
minute hour day(month) month day(week)
```

**Fast lookup — Tools:**
- `at` → one-time job | `atq` → list pending `at` jobs | `atrm` → cancel an `at` job
- `crontab -e` → create/edit personal cron jobs | `crontab -l` → list | `crontab -r` → remove all
- `anacron` → recurring job that survives power-off
- `logrotate` → manages log file size/rotation
- `/etc/cron.{hourly,daily,weekly,monthly}` → drop-in scripts, no manual schedule needed

**Fast lookup — Special characters:**
- `*` = every | `,` = list | `-` = range | `/` = step

**Fast lookup — Runlevels:**
- **0** = shutdown | **1** = single user (rescue) | **2** = multi-user, no network | **3** = server (CLI + network) | **4** = custom/undefined | **5** = desktop (GUI) | **6** = reboot

**Fast lookup — Startup vs. Cron:**
- Cron/at → triggered by **time**
- Startup/runlevel scripts → triggered by **boot/shutdown events**
