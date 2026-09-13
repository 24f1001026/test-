# Linux / Bash Commands — Consolidated Summary

A quick-reference summary of five guides: `shift`/`exec`/`eval`/`getopts`, `crontab`/`at`, `tar`/`gzip`, `sort`, and `uniq`.

---

## 1. Bash Built-ins: `shift`, `exec`, `eval`, `getopts`

| Command | Purpose | Key idiom |
|---|---|---|
| `shift [n]` | Drops the first `n` positional parameters (`$1`, `$2`...), shifting the rest down. Discarded params are gone for good. | `while [ "$#" -gt 0 ]; do ...; shift; done` — loop through args, consuming extras as needed (e.g. `shift 2` for a flag + its value). |
| `exec` | Either **replaces** the current shell process with a command (no new PID, script never resumes), or **redirects file descriptors** for the rest of the script. | `exec > log.txt 2>&1` (redirect all future output); `exec java -jar app.jar` (Docker entrypoint pattern, so the process becomes PID 1 and gets signals directly). |
| `eval` | Concatenates its arguments into a string and re-parses/executes it as shell code — lets stored `\|`, `>`, `&&`, etc. act as real shell syntax instead of literal text. | **Dangerous with untrusted input** — `eval` on unsanitized strings enables command injection (e.g. `; rm -rf ...`). Prefer arrays/associative arrays where possible. |
| `getopts optstring name` | POSIX-standard parser for **single-character** options, including bundled flags (`-vf` = `-v -f`). A colon after a letter (`f:`) means it takes an argument (`$OPTARG`); a leading colon in the optstring enables custom error handling. | Standard pattern: `while getopts ":vf:" opt; do case $opt in ... esac; done; shift $((OPTIND - 1))` to then access remaining positional args. Does **not** support `--long-options` (use manual `shift`/`case` parsing for those). |

**Key gotchas:** `shift` past `$#` can error under strict modes; `exec`'s replacement means code after it never runs; `eval` should never touch untrusted input; `OPTIND` must be reset (`local OPTIND=1`) if `getopts` is called more than once per session.

---

## 2. `crontab` and `at` — Task Scheduling

- **`crontab`** schedules **recurring** jobs; **`at`** schedules **one-time** future jobs.

**crontab basics**
- `crontab -e` / `-l` / `-r` (edit/list/remove your crontab); `-u user` to manage another user's (as root).
- 5-field time syntax: `minute hour day-of-month month day-of-week command` (e.g. `0 9 * * 1-5 cmd` = weekdays at 9 AM).
- Special chars: `*` (any), `,` (list), `-` (range), `/` (step, e.g. `*/15`).
- Shortcuts: `@reboot`, `@daily`, `@weekly`, `@monthly`, `@yearly`, `@hourly`.
- Runs in a **minimal environment** — always use absolute paths and set `PATH`/`SHELL` explicitly; redirect output (`>> log 2>&1`) since cron emails output by default.
- System-wide: `/etc/crontab` (has an extra user field), `/etc/cron.{daily,hourly,weekly,monthly}/`.

**at basics**
- `at TIME` opens an interactive prompt (end with Ctrl+D); accepts natural times (`at 4pm`, `at now + 10 minutes`, `at 5pm tomorrow`).
- `-f FILE` reads commands from a file; `atq`/`at -l` lists pending jobs; `atrm JOBNUM` cancels one; `at -c JOBNUM` views a job's contents.
- `batch` runs a job when system load drops, rather than at a fixed time.
- Requires the `atd` service running (as cron requires `cron`/`crond`).

**Common pitfalls:** minimal cron environment breaking scripts; daemon not running; `crontab -r` has no confirmation (use `-i`); `0` and `7` both mean Sunday.

---

## 3. `tar` and `gzip` — Archiving & Compression

- **`tar`** bundles files/directories into one archive; it does **not** compress by itself.
- **`gzip`** compresses a **single** file (and deletes the original unless `-k` is used).
- Combined (`tar -czvf` / `tar -xzvf`) they form the standard `.tar.gz` workflow.

**tar core flags:** `-c` create, `-x` extract, `-t` list, `-f` filename (almost always needed), `-v` verbose, `-z`/`-j`/`-J` for gzip/bzip2/xz, `-C dir` to extract elsewhere, `--exclude=PATTERN`, `-r` append (uncompressed archives only), `--delete` remove a file from an archive.

```bash
tar -czvf archive.tar.gz folder/        # create compressed archive
tar -xzvf archive.tar.gz -C /dest/      # extract to a directory
tar -tzvf archive.tar.gz                # list contents without extracting
```

**gzip core flags:** `-d`/`gunzip` decompress, `-k` keep original, `-1`–`-9` compression level, `-l` show stats, `-t` test integrity, `-r` recurse through a directory (compressing files individually, not bundling them).

**Common pitfalls:** forgetting `-f`; expecting `gzip` to bundle multiple files (it doesn't — use `tar` first); trying to append to a compressed `.tar.gz` (must decompress first); mismatched extraction flags (though modern GNU tar usually auto-detects compression type).

---

## 4. `sort` — Ordering Lines of Text

- Reads from stdin or file(s); default order is **lexicographic**, so `10` sorts before `2` unless `-n` is used.

| Flag | Effect |
|---|---|
| `-n` / `-h` | Numeric sort / human-readable sizes (2K, 1G) |
| `-r` | Reverse order |
| `-f` | Case-insensitive |
| `-u` | Sort + remove duplicates |
| `-k START[,END]` | Sort by field/column (1-indexed); pair with `-t` to set a delimiter |
| `-M` | Sort by month name |
| `-c` / `-C` | Check if already sorted (loud/silent) |
| `-o FILE` | Write output to a file — safe even if `FILE` is the same as the input (unlike `sort file > file`, which destroys the file) |
| `-R` | Random shuffle |
| `-s` | Stable sort (preserves tie order) |
| `-z` | NUL-delimited (pairs with `find -print0`) |
| `-m` | Merge already-sorted files efficiently |

```bash
sort -t, -k2,2n -k3,3r file.csv   # comma-delimited, field 2 numeric asc, field 3 desc tiebreak
ps aux | sort -k4 -nr | head      # top memory-using processes
sort -o file.txt file.txt         # safe in-place sort
```

**Common pitfalls:** forgetting `-n` for numbers; case sensitivity by default; overwriting a file with `>` instead of `-o`; locale differences (`LC_ALL=C` gives consistent script-safe ordering).

---

## 5. `uniq` — Filtering Adjacent Duplicate Lines

- Only removes **adjacent** duplicates — almost always used after `sort`: `sort file.txt | uniq`.

| Flag | Effect |
|---|---|
| `-c` | Prefix each line with its occurrence count |
| `-d` | Show only duplicated lines (one copy each) |
| `-D` | Show **all** copies of duplicated lines |
| `-u` | Show only lines that appear exactly once |
| `-i` | Case-insensitive comparison |
| `-f N` | Ignore the first N fields when comparing |
| `-s N` | Skip the first N characters when comparing |
| `-w N` | Compare only the first N characters |
| `-z` | NUL-delimited input |

**The classic one-liner** for frequency analysis:
```bash
sort file.txt | uniq -c | sort -nr | head   # most common lines, most frequent first
awk '{print $1}' access.log | sort | uniq -c | sort -nr   # top IPs in a log
```

**`sort -u` vs `sort | uniq`:** `-u` is a quick unique+sorted list; `sort | uniq` (or its variants) is needed for counts, duplicate-only output, or field/character-level comparison logic.

**Common pitfalls:** forgetting to sort first; case sensitivity; confusing `-d` (one copy) with `-D` (all copies); `-f` skips fields, `-s` skips characters — easy to mix up.

---

## Cross-Cutting Patterns Worth Remembering

- **Pipelines over one-off scripts:** `sort | uniq -c | sort -nr` is the standard frequency-count idiom; `tar` + `gzip` is the standard archive-and-compress idiom.
- **Safety first:** `sort -o file file` is safe in-place; `sort file > file` is not. `crontab -r` has no confirmation; `-i` adds one. `eval` on untrusted input is a command-injection risk.
- **Absolute paths & explicit environment** matter for anything run outside an interactive shell (cron jobs, `at` jobs, `exec`'d processes).
- **`shift $((OPTIND - 1))`** is the standard bridge between `getopts` (option parsing) and accessing remaining positional arguments.
