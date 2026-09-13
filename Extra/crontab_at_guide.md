# The `crontab` and `at` Commands — Complete Guide

`crontab` and `at` are both used for **scheduling tasks** on Linux/Unix systems, but they serve different purposes:

- **`crontab`** — schedules **recurring** jobs (run every minute, hour, day, week, etc.)
- **`at`** — schedules a **one-time** job to run once at a specific future time

---

# PART 1: `crontab` (Recurring Jobs)

## 1. What `crontab` Does

`crontab` (cron table) lets each user define scheduled commands/scripts that the `cron` daemon runs automatically at specified times — repeatedly, based on a schedule.

## 2. Basic Syntax

```bash
crontab [OPTIONS]
```

## 3. Core Options

| Option | Description |
|--------|-------------|
| `-e` | Edit the current user's crontab (opens in default editor) |
| `-l` | List the current user's crontab entries |
| `-r` | Remove the current user's entire crontab |
| `-u USER` | Specify another user's crontab (requires root/sudo) |
| `-i` | Prompt before removing crontab (used with `-r`) |

## 4. Editing Your Crontab

```bash
crontab -e
```
Opens your personal crontab file in the default editor (often `vi` or `nano`, configurable via `EDITOR` or `VISUAL` environment variable).

## 5. Viewing Your Crontab

```bash
crontab -l
```

## 6. Removing Your Entire Crontab

```bash
crontab -r
```
> **Warning:** This deletes ALL scheduled jobs for that user with no confirmation. Use `-i` for a safety prompt:
```bash
crontab -ri
```

## 7. Editing Another User's Crontab (as root)

```bash
sudo crontab -u username -e
sudo crontab -u username -l
```

## 8. Crontab Syntax — The 5 Time Fields

```
* * * * * command_to_run
│ │ │ │ │
│ │ │ │ └── Day of week (0–7, both 0 and 7 = Sunday)
│ │ │ └──── Month (1–12)
│ │ └────── Day of month (1–31)
│ └──────── Hour (0–23)
└────────── Minute (0–59)
```

| Field | Allowed Values |
|-------|-----------------|
| Minute | 0–59 |
| Hour | 0–23 |
| Day of Month | 1–31 |
| Month | 1–12 (or Jan–Dec) |
| Day of Week | 0–7 (0 and 7 = Sunday, or Sun–Sat) |

## 9. Special Characters in Cron Syntax

| Symbol | Meaning | Example |
|--------|---------|---------|
| `*` | Every value (wildcard) | `* * * * *` = every minute |
| `,` | List of values | `0 9,17 * * *` = 9 AM and 5 PM |
| `-` | Range of values | `0 9-17 * * *` = every hour from 9 AM to 5 PM |
| `/` | Step values | `*/15 * * * *` = every 15 minutes |
| `?` | No specific value (used in some cron variants, not standard Vixie cron) | — |

## 10. Common Crontab Examples

```bash
# Run every minute
* * * * * /path/to/script.sh

# Run every 5 minutes
*/5 * * * * /path/to/script.sh

# Run every hour, at minute 0
0 * * * * /path/to/script.sh

# Run every day at 2:30 AM
30 2 * * * /path/to/script.sh

# Run every Monday at 9 AM
0 9 * * 1 /path/to/script.sh

# Run on the 1st of every month at midnight
0 0 1 * * /path/to/script.sh

# Run every weekday (Mon–Fri) at 6 PM
0 18 * * 1-5 /path/to/script.sh

# Run twice a day: 9 AM and 9 PM
0 9,21 * * * /path/to/script.sh

# Run every 6 hours
0 */6 * * * /path/to/script.sh

# Run every year on Jan 1st at midnight
0 0 1 1 * /path/to/script.sh
```

## 11. Special Predefined Strings (Shortcuts)

Instead of the 5-field syntax, you can use these shortcuts (supported in most modern cron implementations):

| Shortcut | Equivalent To |
|----------|----------------|
| `@reboot` | Run once at system startup |
| `@yearly` / `@annually` | `0 0 1 1 *` |
| `@monthly` | `0 0 1 * *` |
| `@weekly` | `0 0 * * 0` |
| `@daily` / `@midnight` | `0 0 * * *` |
| `@hourly` | `0 * * * *` |

Example:
```bash
@reboot /path/to/startup_script.sh
@daily /path/to/backup.sh
```

## 12. Redirecting Output / Logging

By default, cron emails output to the user (if mail is set up). It's common to redirect output to a log file instead:

```bash
0 2 * * * /path/to/script.sh >> /var/log/myscript.log 2>&1
```
- `>>` appends stdout to the log file
- `2>&1` redirects stderr into the same log file

To discard all output:
```bash
0 2 * * * /path/to/script.sh > /dev/null 2>&1
```

## 13. Environment Variables in Cron

Cron jobs run with a **minimal environment** (not your full shell environment), which is a very common source of "works in terminal but not in cron" bugs. You can set variables at the top of the crontab:

```bash
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/bin:/bin
MAILTO=you@example.com

0 5 * * * /path/to/script.sh
```
- `MAILTO` — sends job output via email to the specified address (set to empty `MAILTO=""` to disable emails).
- Always use **absolute paths** in cron jobs and scripts, since `PATH` may not match your interactive shell.

## 14. System-Wide Crontab Files

Besides per-user crontabs (managed via `crontab -e`), there are system-level locations:

| Location | Purpose |
|----------|---------|
| `/etc/crontab` | System-wide crontab (includes a username field) |
| `/etc/cron.d/` | Directory for drop-in system cron jobs |
| `/etc/cron.daily/` | Scripts run once a day |
| `/etc/cron.hourly/` | Scripts run once an hour |
| `/etc/cron.weekly/` | Scripts run once a week |
| `/etc/cron.monthly/` | Scripts run once a month |

`/etc/crontab` syntax includes an extra **user** field (since it's not tied to a single user):
```
# minute hour day month weekday user command
0   3   *   *   *   root   /path/to/script.sh
```

## 15. Checking Cron Service Status

```bash
sudo systemctl status cron        # Debian/Ubuntu
sudo systemctl status crond       # RHEL/CentOS/Fedora
```

Restarting the cron service:
```bash
sudo systemctl restart cron       # Debian/Ubuntu
sudo systemctl restart crond      # RHEL/CentOS
```

## 16. Checking Cron Logs

```bash
grep CRON /var/log/syslog          # Debian/Ubuntu
sudo journalctl -u cron            # systemd-based systems
sudo cat /var/log/cron             # RHEL/CentOS
```

## 17. Controlling Who Can Use Cron

| File | Purpose |
|------|---------|
| `/etc/cron.allow` | If it exists, only listed users can use cron |
| `/etc/cron.deny` | If it exists (and `cron.allow` doesn't), listed users are blocked from cron |

---

# PART 2: `at` (One-Time Jobs)

## 18. What `at` Does

`at` schedules a command to run **once** at a specified future time. Unlike cron, it's not recurring — perfect for "remind me" or "run this one time later tonight" type tasks.

## 19. Checking/Installing `at`

```bash
which at
sudo apt install at        # Debian/Ubuntu
sudo yum install at        # RHEL/CentOS
```

Make sure the `atd` service is running:
```bash
sudo systemctl status atd
sudo systemctl enable --now atd
```

## 20. Basic Syntax

```bash
at [TIME]
```
After running this, you get an interactive prompt (`at>`) where you type the command(s) to run, then press `Ctrl+D` to save.

## 21. Basic Example

```bash
at 10:00 PM
at> /path/to/script.sh
at> <Ctrl+D>
```
Output:
```
job 3 at Sat Sep 12 22:00:00 2026
```

## 22. Accepted Time Formats

| Format | Example |
|--------|---------|
| Specific time today | `at 15:30` |
| AM/PM time | `at 4pm` |
| Named times | `at midnight`, `at noon`, `at teatime` (4 PM) |
| Relative time | `at now + 10 minutes` |
| Relative time (other units) | `at now + 2 hours`, `at now + 3 days`, `at now + 1 week` |
| Specific date | `at 10:00 09/20/2026` |
| Specific date (alt format) | `at 10:00am Sep 20` |
| Combined | `at 5pm tomorrow` |

## 23. Examples

```bash
# Run a script 10 minutes from now
at now + 10 minutes
at> /home/user/backup.sh

# Run at 2 AM tomorrow
at 2am tomorrow
at> /home/user/cleanup.sh

# Run at a specific date and time
at 09:00 12/25/2026
at> /home/user/holiday_script.sh
```

## 24. Passing a Command Directly (Without Interactive Prompt)

Use `echo` piped into `at`:
```bash
echo "/home/user/script.sh" | at now + 5 minutes
```

Or use a heredoc:
```bash
at now + 1 hour <<EOF
/home/user/script.sh
EOF
```

## 25. Scheduling from a File

```bash
at -f /path/to/commands.sh now + 30 minutes
```
- `-f FILE` reads commands from a file instead of typing them interactively.

## 26. Listing Pending `at` Jobs

```bash
atq
# or
at -l
```
Output shows job number, scheduled time, and queue:
```
3   Sat Sep 12 22:00:00 2026 a user
```

## 27. Viewing the Contents of a Scheduled Job

```bash
at -c JOB_NUMBER
```
Shows the full script/environment that will run for that job.

## 28. Removing a Scheduled Job

```bash
atrm JOB_NUMBER
# or
at -r JOB_NUMBER
```

## 29. Queues in `at`

`at` supports multiple queues (a-z, A-Z), mainly used to set priority (nice value). Default queue is `a`.

```bash
at -q b now + 1 hour
at> /path/to/script.sh
```
Batch queue `b` is often used by the related `batch` command (see below).

## 30. The `batch` Command (Related to `at`)

```bash
batch
```
Schedules a command to run when system load drops below a certain threshold, rather than at an exact time — useful for resource-heavy tasks that shouldn't run during high load.

```bash
batch
at> /path/to/heavy_script.sh
at> <Ctrl+D>
```

## 31. Output and Notifications

By default, `at` sends job output via **email** to the user (if mail is configured), similar to cron. To capture output manually instead:

```bash
echo "/path/to/script.sh >> /home/user/log.txt 2>&1" | at now + 5 minutes
```

## 32. Controlling Who Can Use `at`

| File | Purpose |
|------|---------|
| `/etc/at.allow` | If it exists, only listed users can use `at` |
| `/etc/at.deny` | If it exists (and `at.allow` doesn't), listed users are blocked |

---

# PART 3: `cron` vs `at` — Quick Comparison

| Feature | `crontab` | `at` |
|---------|-----------|------|
| Purpose | Recurring scheduled jobs | One-time scheduled job |
| Syntax style | 5-field time pattern | Natural language time expressions |
| Runs repeatedly? | Yes | No — runs once, then job is done |
| Good for | Backups, log rotation, periodic checks | Reminders, one-off deferred tasks |
| Managed by | `crontab -e` | `at TIME` interactive prompt or `-f` file |
| Service required | `cron` / `crond` | `atd` |
| List scheduled jobs | `crontab -l` | `atq` |
| Remove scheduled jobs | `crontab -r` (all) | `atrm JOB_NUMBER` (specific job) |

---

## 33. Quick Reference Cheat Sheet

### crontab
```bash
crontab -e                     # edit your crontab
crontab -l                     # list your crontab entries
crontab -r                     # remove your entire crontab
crontab -u user -e             # edit another user's crontab (as root)

* * * * * cmd                  # every minute
*/15 * * * * cmd                # every 15 minutes
0 * * * * cmd                   # every hour
0 0 * * * cmd                   # every day at midnight
0 9 * * 1-5 cmd                 # weekdays at 9 AM
@reboot cmd                     # run once at startup
@daily cmd                      # run once a day
```

### at
```bash
at 10pm                        # schedule job for 10 PM today
at now + 10 minutes            # schedule job 10 minutes from now
at 09:00 12/25/2026             # schedule job for a specific date/time
atq                              # list pending jobs
at -c JOBNUM                     # view job contents
atrm JOBNUM                      # cancel a pending job
batch                            # run when system load is low
```

---

## 34. Common Pitfalls

1. **Cron's minimal environment** — scripts that work fine in your terminal may fail in cron due to missing `PATH` or environment variables. Always use absolute paths and set `PATH` explicitly in the crontab.
2. **Forgetting the `atd`/`cron` service isn't running** — jobs silently never execute if the daemon is stopped.
3. **`crontab -r` has no confirmation** — it deletes everything instantly; use `-i` for a safety prompt.
4. **Mixing up day-of-week values** — both `0` and `7` mean Sunday, which can cause confusion in scripts generated by different tools.
5. **Assuming `at` jobs persist across reboot** — pending `at` jobs are typically preserved (stored in `/var/spool/at`), but always confirm on your specific system/distro.
6. **No output visible** — both `cron` and `at` mail results by default; if mail isn't configured, output can seem to "disappear" unless redirected to a log file.

---

## 35. Related Commands

- `systemctl` — manage the `cron`/`crond` and `atd` services
- `systemd timers` — modern alternative to cron, using `.timer` unit files
- `anacron` — like cron but designed for systems that aren't always on (e.g., laptops)
- `watch` — repeatedly run a command at fixed intervals (foreground, not scheduled in background)
- `nohup` / `disown` — keep a manually started process running after logout (different use case from scheduling)

---

## 36. Summary

`crontab` is the tool of choice for **recurring** scheduled tasks — backups, cleanups, periodic checks — defined using the classic 5-field time syntax. `at` is the tool for **one-time** scheduled tasks, using natural, human-readable time expressions. Together, they cover almost every task-scheduling need on a Linux system, with `systemd timers` as a more modern (but more complex) alternative for advanced use cases.
