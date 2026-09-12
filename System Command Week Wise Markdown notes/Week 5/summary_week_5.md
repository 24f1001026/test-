# Week 5 — Shell Variables — Revision Summary

> Quick-revision notes covering Lecture 1 (echo, environment vars, `date`, special vars, `ps`, process control, exit codes, flags) and Lecture 2–3 (creating/modifying variables, defaults, string ops, arrays). Edge cases and "gotchas" are folded into each topic so you don't miss them while skimming.

---

## 1. Basics — Creating & Reading Variables

| Action | Syntax | Notes |
|---|---|---|
| Create | `myvar="value"` | No `$` when **setting** |
| Read | `echo $myvar` or `echo ${myvar}` | `$` needed only when **reading** |
| Rule | No spaces around `=` | `myvar = "x"` → **error** |
| Rule | Can't start with a digit | `1var=10` → error; must start with letter/underscore |
| Allowed chars | Letters, digits, underscore | `my_var1=10` ✅ |
| Value types | String, number, command output | Bash variables are **untyped** — everything is really a string unless restricted with `declare` |

**Remember:** Always prefer `${myvar}` (curly braces) when the variable is immediately followed by a letter/digit/underscore that could merge into the name — e.g. `${count}items` vs the broken `$countitems`. `"$file.txt"` is safe (`.` isn't a valid name character) but `"$file_$ext"` silently resolves to empty because Bash reads `file_` as one (unset) variable name.

---

## 2. `echo` and Quoting

| Quoting Style | Variables Expanded? | Example | Output |
|---|---|---|---|
| No quotes | ✅ Yes | `echo $USER` | `john` |
| Double quotes `"..."` | ✅ Yes | `echo "$USER"` | `john` |
| Single quotes `'...'` | ❌ No | `echo '$USER'` | `$USER` |
| Escaped `\$` | ❌ No (literal) | `echo \$USER` | `$USER` |

**Remember:**
- Double quotes expand variables *and* preserve exact spacing — the safest default for mixing text + variables.
- Single quotes suppress **all** expansion, including `$VAR`. This is one of the most exam-tested distinctions.
- An unclosed quote makes Bash wait for more input across multiple lines until it's closed — this is how multi-line `echo` strings get typed interactively.
- Nest quote types (`'...'` inside `"..."` or vice versa) to print literal quote characters.

---

## 3. Common Environment Variables

| Variable | Meaning |
|---|---|
| `$USER` / `$USERNAME` | Current logged-in username (`$USER` is more portable) |
| `$HOME` | Path to home directory |
| `$HOSTNAME` | Machine's network name |
| `$PWD` | Current working directory — **auto-updates on every `cd`**, never set manually |
| `$PATH` | Colon-separated list of dirs searched for executables — consulted every time you type a command |

**View all variables at once:**

| Command | What it lists |
|---|---|
| `printenv` | Exported environment variables (or a named one) |
| `env` | Current environment / run a command in modified env |
| `set` | **All** shell variables (incl. local/non-exported) + functions |

---

## 4. `date` Command

| Command | Format |
|---|---|
| `date` | Human-readable local format |
| `date -R` | RFC 2822 format (used in email headers/logs) |

**Command substitution with `date` (very common exam pattern):**

| Option | Meaning | Example |
|---|---|---|
| `+%Y` | 4-digit year | `2026` |
| `+%m` | Month | `08` |
| `+%d` | Day | `22` |
| `+%Y-%m-%d` | ISO date | `2026-08-22` |
| `+%H:%M:%S` | 24-hr time | `14:05:32` |
| `+%A` | Weekday name | `Saturday` |

```bash
today=$(date +%Y-%m-%d)   # preferred over `date` backticks
```

**Remember:** Prefer `$()` over backticks `` ` ` `` — backticks require escaping (`` \` ``) to nest, `$()` nests directly. Nested example: `filecount=$(ls $(pwd) | wc -l)`.

---

## 5. Bypassing Aliases

| Method | How |
|---|---|
| `\command` | Ignores alias for this one call (still respects functions/builtins) |
| `/full/path/to/command` | Bypasses alias lookup entirely — most explicit/unambiguous |

**Remember:** If `date` is aliased (e.g. `alias date='date +%A'`), only `\date` or `/usr/bin/date` gives you the *original* command back.

---

## 6. Special Shell Variables

| Variable | Meaning |
|---|---|
| `$0` | Name of shell/script currently running |
| `$1`–`$9` | Positional parameters (individual arguments) |
| `$#` | Count of arguments passed |
| `$@` | All arguments, each preserved as a separate word |
| `$$` | PID of current shell |
| `$?` | Exit code of last command (`0` = success, non-zero = failure) |
| `$-` | Currently active shell flags |

**Remember:** `$#` = *how many*, `$@` = *the arguments themselves*, `$1..$9` = *individual access*. Quote `"$@"` when arguments may contain spaces, so each stays a separate word instead of getting split.

---

## 7. `ps` — Process Monitoring

| Command | Shows |
|---|---|
| `ps` | Current terminal/session processes only |
| `ps -e` | All system processes, short format |
| `ps -f` | Current user's processes, full format |
| `ps -ef` | All processes, full format (UID, PPID, start time) |
| `ps --forest` | Parent-child tree view |

**Remember:** `-e` = *every* process, `-f` = *full* detail — they're independent flags you can combine (`-ef`). Plain `ps` with no flags is scoped to your session only, which surprises people expecting a system-wide list.

---

## 8. Process Control

| Tool | Purpose |
|---|---|
| `&` | Run job in background |
| `fg` | Bring background job to foreground |
| `jobs` | List background/suspended jobs |
| `coproc` | Run command as co-process (bidirectional communication) |
| `top` | Live view of processes/resource usage |
| `kill` | Send a signal to a process (default: `SIGTERM`) |

**Remember:** `kill` alone sends **SIGTERM (15)** — a polite request the process can catch/ignore. `kill -9` sends **SIGKILL** — immediate, unignorable termination (ties directly to exit code `137` below).

---

## 9. Exit Codes (`$?`)

| Code | Meaning |
|---|---|
| `0` | Success |
| `1` | General failure |
| `2` | Misuse of shell command (bad syntax/args) |
| `126` | Command found but not executable (e.g. permission denied) |
| `127` | Command not found |
| `130` | Terminated by Ctrl+C (SIGINT) |
| `137` | Terminated by `kill -9` (SIGKILL) |

**Remember:** `130` vs `137` is a favorite exam pair — both mean "killed," but `130` = user interrupt, `137` = forceful kill signal. Check `$?` **immediately** after the command — running any other command in between overwrites it.

---

## 10. Bash Flags (`$-`)

| Flag | Meaning |
|---|---|
| `h` | Command hashing enabled |
| `i` | Interactive shell |
| `m` | Job control enabled (`fg`/`bg`/`jobs`) |
| `B` | Brace expansion enabled |
| `H` | `!`-style history substitution enabled |
| `s` | Reading commands from stdin |
| `c` | Reading commands from `-c` argument |

Typical output: `echo $-` → `himBH`

---

## 11. Command Substitution

```bash
myvar=`command`     # legacy, hard to nest (needs \` escaping)
myvar=$(command)     # preferred, nests directly
```

**Remember:** Always prefer `$()`. Nested backticks are error-prone; `$()` nests cleanly, e.g. `filecount=$(ls $(pwd) | wc -l)`.

---

## 12. Exporting Variables

| Variable Type | Visible in current shell? | Visible in subshell/child? |
|---|---|---|
| Local (not exported) | ✅ | ❌ |
| Exported (`export var`) | ✅ | ✅ |

**Remember — one-way inheritance (heavily tested):** `export` copies the value into child processes, but the **child gets its own copy**. If the child modifies it, the parent is **never** affected, even after the child exits.
```bash
export counter="10"
bash -c 'counter="99"'   # only changes the child's copy
echo "$counter"           # still 10 in the parent
```

---

## 13. Removing / Emptying Variables

| Action | Command | Still exists? | `${myvar}` value | `[[ -v myvar ]]` |
|---|---|---|---|---|
| Remove completely | `unset myvar` | ❌ No | undefined | exit `1` (failure) |
| Remove value only | `myvar=` | ✅ Yes | `""` (empty) | exit `0` (success) |

**Remember:** These are *not* the same thing — `myvar=` keeps the variable "set" (just empty), which matters for `-v` checks and default-value operators below.

---

## 14. Checking Variable Status

| Test | Syntax | Exit `0` means |
|---|---|---|
| Is set? | `[[ -v myvar ]]` | Variable **is** set |
| Is NOT set? | `[[ -z ${myvar+x} ]]` | Variable **is not** set |

**Remember:** In `${myvar+x}`, `x` is just a placeholder string — it's never actually assigned to `myvar`; it only exists briefly during expansion to make `-z` testable.

---

## 15. Default Value Operators

| Operator | If SET | If NOT SET | Modifies variable? |
|---|---|---|---|
| `${myvar:-"default"}` | Shows current value | Shows `"default"` only | ❌ No |
| `${myvar:="default"}` | Shows current value | **Sets** and shows `"default"` | ✅ Yes (only when unset) |
| `${myvar:+"default"}` | **Sets/overwrites** and shows `"default"` | Shows nothing | ✅ Yes (only when set) |
| `${myvar?"error msg"}` | Shows current value | Prints error to stderr and **halts script** | N/A — doesn't substitute, it exits |

**Remember:** `:+` behaves opposite to what people expect at first — it fires (and overwrites!) only when the variable **is already set**, doing nothing when unset. `?` is the *only* one of these four that can stop script execution; the other three never do.

---

## 16. Listing Names & String Length

| Task | Syntax | Example |
|---|---|---|
| List variable names by prefix | `${!H*}` | Lists `HOME HOSTNAME ...` |
| String length | `${#myvar}` | `${#myvar}` → `5` for `"Hello"`; `0` if unset |

---

## 17. Substrings (Slicing)

| Expression | Meaning |
|---|---|
| `${myvar:offset:length}` | From `offset`, take `length` chars |
| `${myvar:5}` | From offset 5 to the end |
| `${myvar: -3:2}` | 3 chars from the **right**, then take 2 |

**Remember — the #1 slicing trap:** A **space is required** before a negative offset: `${myvar: -3:2}`, NOT `${myvar:-3:2}`. Without the space, Bash parses it as the **default-value `:-` operator** (Section 15) instead of slicing — completely different behavior, same-looking syntax.

---

## 18. Pattern Removal (Prefix/Suffix Trimming)

| Operator | Direction | Match Length | Example (`/a/b/c.txt`, pattern `*/`) → Output |
|---|---|---|---|
| `#` | From start | Shortest | `a/b/c.txt` |
| `##` | From start | Longest | `c.txt` |
| `%` | From end | Shortest | (on `file.tar.gz`, pattern `.*`) → `file.tar` |
| `%%` | From end | Longest | `file` |

**Remember:** `#`/`##` strip from the **front**, `%`/`%%` strip from the **back** — easy to mix up under time pressure. To combine a prefix strip and a suffix strip, Bash does **not** allow direct nesting like `${${var#*/}%.*}` — you must use an **intermediate variable** or command substitution (e.g. `basename`) instead.

---

## 19. Pattern Replacement

| Operator | Behavior |
|---|---|
| `${myvar/pattern/string}` | Replace **first** match only |
| `${myvar//pattern/string}` | Replace **all** matches |
| `${myvar/#pattern/string}` | Replace only if match is at the **beginning** |
| `${myvar/%pattern/string}` | Replace only if match is at the **end** |

Example: `myvar="banana"` → `${myvar/a/O}` = `bOnana`, `${myvar//a/O}` = `bOnOnO`.

---

## 20. Changing Case

| Operator | Effect |
|---|---|
| `${myvar,}` | First character → lower |
| `${myvar,,}` | All characters → lower |
| `${myvar^}` | First character → upper |
| `${myvar^^}` | All characters → upper |

**Remember:** Comma operators (`,` `,,`) lower-case; caret operators (`^` `^^`) upper-case. Single symbol = first char only; doubled symbol = whole string.

---

## 21. Restricting Variable Types (`declare`)

| Flag | Restriction | Removable with `+`? |
|---|---|---|
| `-i` | Integer only | ✅ Yes (`declare +i`) |
| `-l` | Lower-case only (auto-converts) | ✅ Yes |
| `-u` | Upper-case only (auto-converts) | ✅ Yes |
| `-r` | Read-only | ❌ **No — permanent for the life of the variable** |

**Remember:** `declare -i num; num="abc"` doesn't error — it silently becomes `0` (non-numeric input treated as `0` in arithmetic context). `-r` is the one restriction you can never undo, even with `+r`.

---

## 22. Arrays

### Indexed Arrays (numeric keys, from 0)

| Operation | Syntax |
|---|---|
| Declare | `declare -a arr` |
| Set element | `arr[0]="value"` (no `$` when setting!) |
| Get element | `${arr[0]}` |
| Count | `${#arr[@]}` |
| List indices | `${!arr[@]}` |
| List values | `${arr[@]}` |
| Delete element | `unset 'arr[2]'` |
| Append | `arr+=("value")` |
| One-shot init | `arr=(a b c)` |

**Remember:** After `unset 'arr[1]'`, the array keeps a **gap** at that index (`${!arr[@]}` shows `0 2`, not `0 1`) — it doesn't re-index automatically.

**Populating from command output:**

| Method | Splits by | Safety |
|---|---|---|
| `arr=($(command))` | Any whitespace | ⚠️ Breaks on filenames with spaces |
| `mapfile -t arr < <(command)` | Newline | ✅ Safer for real-world file lists |
| `readarray -t arr < <(command)` | Newline | Same as `mapfile` (alias) |

### Associative Arrays (string keys, Bash 4.0+)

| Operation | Syntax |
|---|---|
| Declare | `declare -A hash` |
| Set | `hash["key"]="value"` (no `$` when setting!) |
| Get | `${hash["key"]}` |
| Count | `${#hash[@]}` |
| List keys | `${!hash[@]}` |
| List values | `${hash[@]}` |
| Delete | `unset 'hash["key"]'` |

**Remember:** Key/value order in `${!hash[@]}` and `${hash[@]}` is **not guaranteed** for associative arrays (unlike indexed arrays, which stay in numeric order). Also — same trap as everywhere else — assignment never uses a leading `$` (`hash["a"]="value"`, not `$hash["a"]=...`).

---

## Quick Exam-Recall Cheatsheet

| Symbol | Meaning |
|---|---|
| `$?` | Exit code of last command |
| `$$` | PID of current shell |
| `$0` | Script/shell name |
| `$1`–`$9`, `$#`, `$@` | Positional args, count, all args |
| `$-` | Active shell flags |
| `:-` | Show default (no change) |
| `:=` | Set default (permanent) |
| `:+` | Overwrite if set |
| `?` | Error + halt if unset |
| `#` / `##` | Strip prefix (shortest/longest) |
| `%` / `%%` | Strip suffix (shortest/longest) |
| `/` / `//` | Replace first / replace all |
| `,` `,,` `^` `^^` | Lower first / lower all / upper first / upper all |

> *"Zero is success, everything else is a story."* — `$?` tells the story, `$$` tells you the shell's identity (PID), `$0` its name, `$1..$9/$#/$@` what it was handed, `$-` its mood (flags).
