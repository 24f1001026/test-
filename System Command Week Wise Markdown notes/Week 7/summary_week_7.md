# Week 7 — Advanced Bash Scripting | Quick Revision Summary

*(Lectures 1–3 — Debugging, Arithmetic, Regex, Loops, Control Flow, exec/eval, getopts, select)*

---

## 1. Debugging (`set -x`, `bash -x`)

`bash -x ./script.sh` traces the **whole script** from the command line without editing it — every command is printed **after variable substitution**, prefixed with `+`. Inside a script, `set -x` turns tracing on from that line and `set +x` turns it off, so you can trace just a suspicious section. Remember: traced output shows the *actual values* used, not `$varname` as written in the source — that's the whole point of using it over just reading the code.

| Command | Scope |
|---|---|
| `bash -x script.sh` | Entire script, no edits needed |
| `set -x` ... `set +x` | Only the section between them |

---

## 2. Combining Conditions (`&&`, `||`)

Two independent `[ ]` tests can be chained with `&&` (AND, both must succeed) or `||` (OR, either succeeds) — this works because `&&`/`||` are chaining **exit statuses** of separate commands, not just tests (e.g. `mkdir x && cd x` only runs `cd` if `mkdir` succeeded). Inside a single `[[ ]]`, you can write `&&`/`||` **directly in one bracket**, and `[[ ]]` also lets you use unescaped `<`/`>` for string comparison and native regex — something `[ ]` can't do safely.

**Remember:** `[ ]` and `[[ ]]` are *not* interchangeable — `[[ ]]` is safer (no word-splitting issues, needs no variable quoting) but `[ ]` is more POSIX-portable (works in `sh`/`dash`). If you see `<` or `>` unescaped inside `[ ]`, that's a bug — they'd be read as redirection operators.

---

## 3. Shell Arithmetic — 6 Methods

| Method | Syntax | Integer-only? | Notes |
|---|---|---|---|
| `(( ))` | `(( total = 10+20 ))` | Yes | Evaluates as a *command*; exit status 0=true if result ≠ 0 |
| `let` | `let a=$1+5` | Yes | Quote whole expr if it has spaces: `let "a = $1 + 5"` |
| `expr` | `expr $a + 20` | Yes | **External command** — needs spaces between every token |
| `$(( ))` | `b=$(( a+10 ))` | Yes | ✅ Best practice — modern standard, full C-operator support |
| `$[ ]` | `b=$[ a+10 ]` | Yes | ❌ Deprecated legacy form of `$(( ))` |
| `bc` | `echo "10/3" \| bc -l` | **No** | Only tool with decimal/float support; use `scale=N` for precision |

**Edge cases to remember:**
- `expr $a+20` (no spaces) is treated as **one literal string** and just echoes `5+20` back — `expr` is token-based, so spacing is mandatory.
- `*`, `<`, `>` inside `expr` must be escaped (`\*`, `\<`, `\>`) or Bash will treat them as a wildcard/redirection operator before `expr` ever sees them.
- All of `(( ))`, `let`, `expr`, `$(( ))`, `$[ ]` are **integer-only** — decimals silently truncate or error. Only `bc` (ideally with `-l` for the math library) handles floating point.
- `expr` is an external process, so it's noticeably **slower in loops** than `$(( ))` (confirmed via `time` comparisons) — prefer `$(( ))` for performance.
- `expr` operators table (all need spacing/escaping): `+ - * / %` arithmetic; `> >= < <= =` relational (`\>`, `\<` escaped); `| &` logical (return first non-null/non-zero operand or fallback); `!=` not-equal; `\( \)` grouping for precedence.

---

## 4. Pattern Matching & Regex

| Mechanism | Regex flavor | Anchoring | Notes |
|---|---|---|---|
| `expr "$str" : "regex"` | BRE | **Anchored at start** (like implicit `^`) | Returns matched length, or `0` if no match; returns the captured substring if pattern has `\( \)` |
| `expr match "$str" "regex"` | BRE | Anchored at start | Identical to `str : reg`, just a readable alias |
| `[[ $str =~ regex ]]` | ERE | **Unanchored** unless you add `^`/`$` yourself | Captures go into `BASH_REMATCH[]` array |
| `expr substr "$str" n m` | — | — | Extract `m` chars starting at 1-indexed position `n`; if fewer chars exist, just returns what's available (no error) |
| `expr index "$str" "chars"` | — | — | Position of **first** occurrence of *any* char in `chars`; `0` if none found |
| `expr length "$str"` | — | — | Total character count |

**Edge cases to remember:**
- The most commonly confused pair on exams: `str : reg` (BRE via `expr`, **anchored** at the start) vs `str =~ reg` (ERE via `[[ ]]`, **unanchored**). `expr "HelloWorld" : "World"` returns `0` because the match must start at position 1 — even though "World" clearly appears in the string.
- BRE special characters like `+ ? |` need escaping (`\+ \? \|`) to work as regex metacharacters with `expr`; ERE inside `[[ =~ ]]` uses them unescaped.
- Classic validation pattern: `^[0-9]+$` — anchored both ends, requires the *entire* string to be digits only. Forgetting either `^` or `$` lets partial matches slip through as "valid."
- Regex with `*` (zero-or-more) can silently reject inputs you'd expect to match — e.g. `^[oO]ctav[aeiou]*$` matches "Octavia" but **not** "Octavio", because the trailing part must be pure vowels, and the consonant `v` before `io` breaks the match. Always test boundary cases by hand.
- The regex pattern can also be stored in a variable and used with `expr "$str" : "$regex"` for dynamic patterns.

---

## 5. Heredoc (`<<`, `<<-`)

Feeds multi-line text directly into a command's stdin without a separate file — most often used to send multi-line math to `bc`. The closing marker (any word, not just `EOF`) must appear **alone on its own line** and match the opener exactly (case-sensitive). `<<-` strips **leading TAB characters only** (not spaces) from each body line, so indentation inside functions/loops stays readable.

**Edge case to remember:** if you indent heredoc body lines with **spaces** instead of tabs, `<<-` will **not** strip them — mixing tabs/spaces is a classic silent-failure bug that breaks the heredoc closing marker recognition too.

---

## 6. `IFS` (Internal Field Separator)

Controls how Bash splits a string into words/fields — default is space, tab, newline. Temporarily set `IFS=:` (or `,`, etc.) to parse delimited data (like `/etc/passwd`-style lines or CSV) inside a `for` loop or with `read -ra`.

**Edge case to remember:** always save the original with `OLDIFS=$IFS` before changing it, and restore with `IFS=$OLDIFS` afterward — **forgetting to restore `IFS` silently breaks normal space-based word-splitting for the rest of the script**, causing hard-to-diagnose bugs later on.

---

## 7. Conditionals — `if` / `case`

`if-elif-else-fi` evaluates top to bottom and stops at the **first** true condition (good for range/threshold checks like grading). `case` matches a variable against patterns (wildcards allowed), using `|` to group multiple values into one branch and `*` as the catch-all default — cleaner than a long `elif` chain for discrete/enumerated values (menus, file extensions, y/n prompts).

**Edge case to remember:** in `case`, the `*)` default branch only fires if *no* earlier pattern matched — if your patterns overlap or you order them wrong, an earlier broad pattern can shadow a later specific one, since `case` executes the **first** match, not the best match.

---

## 8. Loops — C-style `for (( ))`

```bash
for (( init; condition; increment )); do ...; done
```
Supports **multiple comma-separated variables** in the initializer/increment (e.g. `for (( a=1, b=10; a<10; a++, b-- ))`), but Bash's C-style `for` allows only **ONE** terminating condition regardless of how many variables you're tracking — this is a very common "spot the bug" exam trap (people try to write `a < 10 && b > 0` expecting both to control the loop, but syntactically only one condition slot exists).

Redirecting `done > file` or `done >> file` applies to the **combined output of the entire loop**, treating it as one unit — not per-iteration. `tmp.$$` is a common pattern for a unique temp filename, since `$$` is the current process's PID.

---

## 9. `time` — Measuring Execution

Reports `real` (wall-clock), `user` (CPU time in user code), `sys` (CPU time in kernel calls).

**Edge case to remember:** `real` is **not simply `user + sys`** — on multi-core systems with parallel work, `real` can be *less* than `user+sys`; when a process waits on I/O, `real` can be *greater*. This distinction is a common short-answer question.

---

## 10. Loop Control — `break` / `continue` (with `n`)

| Statement | Effect |
|---|---|
| `break` | Exits the **innermost** loop only |
| `break n` | Exits `n` levels of nested loops at once, counted from innermost outward |
| `continue` | Skips rest of current iteration, jumps to next iteration check of the **innermost** loop |
| `continue n` | Skips to the next iteration of the loop `n` levels up |

**Edge case to remember:** using plain `break`/`continue` when you actually needed `break 2`/`continue 2` in nested loops is one of the most common mistakes — plain `break` only ever affects the loop it's physically written inside, never an outer one, no matter how the logic "feels" like it should exit everything.

---

## 11. `shift` — Positional Parameters

`shift` moves `$1, $2, $3...` left by one (discarding old `$1`); `shift n` moves by `n` at once. `$#` decreases accordingly. Classic pattern: `while [ -n "$1" ]; do ... shift; done` to process an unknown number of arguments, or `action=$1; shift; echo "$@"` to peel off a command name before handling remaining flags.

---

## 12. `exec` vs `eval`

| | `exec` | `eval` |
|---|---|---|
| Effect | **Replaces** the current shell process entirely | Builds a string and executes it as a command, **returns control** normally |
| After success | Nothing after it in the script runs | Script continues normally afterward |
| After failure | Script **continues** with the next line (only a failed exec lets you proceed) | N/A — always returns |
| Also used for | Permanently redirecting the shell's own I/O (`exec > output.log`) | Dynamic variable names/commands built at runtime |

**Edge cases to remember:**
- People often forget that code placed *after* a successful `exec` call **never executes** — this is only true on success; a *failed* `exec` (bad path/command) lets execution fall through to the next line.
- `eval` executing raw/untrusted user input is a real security risk (arbitrary command execution) — always validate/sanitize input before passing it to `eval`.

---

## 13. `getopts` — Parsing Flags

```bash
while getopts "ab:c:" options
do
  case "${options}" in
    a) ;;
    b) barg=${OPTARG};;
    c) carg=${OPTARG};;
    *) echo "Usage: ...";;
  esac
done
```
A trailing `:` after an option letter (e.g. `b:`) means that flag **requires a value**, captured into `${OPTARG}`. `$OPTIND` tracks how many arguments `getopts` has consumed so far — `shift $(( OPTIND - 1 ))` is the standard way to access any remaining non-flag arguments afterward.

**Edge case to remember:** forgetting the colon after an option letter is a very common silent bug — without `:`, `getopts` assumes that flag takes **no argument**, so `OPTARG` stays empty even if the user supplied a value, and everything after it gets misparsed.

---

## 14. `select` — Interactive Menus

```bash
select variable in list_of_options
do
  commands
done
```
Displays a numbered menu, prompts via `PS3` (customizable, default is `#?`), stores the chosen text in the loop variable, and **loops forever** unless the body explicitly calls `break` (or `exit`).

**Edge case to remember:** this is a classic "spot the bug" question — a `select` loop with no `break` anywhere in its `case`/`if` body will keep re-prompting indefinitely, even after a valid selection, because `select` has no automatic exit condition of its own.

---

## Full Exam-Pitfalls Cheat Table (All Topics)

| # | Mistake | Fix |
|---|---|---|
| 1 | `expr $a+20` (no spaces) | Always space out operators: `expr $a + 20` |
| 2 | `expr $a * $b` unescaped | Escape: `expr $a \* $b` |
| 3 | Two conditions in a multi-variable `for (( ))` | Only ONE condition allowed, e.g. `a < finish` |
| 4 | `select` with no `break` | Always include a `break`/`exit` path |
| 5 | `break` used where `break 2` was needed | Use `break n` for correct nesting depth |
| 6 | `continue` used where `continue 2` was needed | Use `continue n` for correct nesting depth |
| 7 | Confusing `str : reg` (BRE, anchored) with `str =~ reg` (ERE, unanchored) | Know which tool = which regex flavor/anchoring |
| 8 | Forgetting to restore `IFS` | `OLDIFS=$IFS` before, `IFS=$OLDIFS` after |
| 9 | Expecting code after successful `exec` to run | Only a **failed** exec lets the next line execute |
| 10 | Missing `:` after option letter in `getopts` | Use `"b:"` if the flag needs a value |
| 11 | Assuming `(( ))`/`let`/`expr`/`$(( ))`/`$[ ]` handle decimals | Only `bc -l` handles floating point |
| 12 | Misreading `time` output (`real` ≠ `user+sys` always) | `real`=wall-clock; can be less (parallel) or more (I/O wait) |
| 13 | Heredoc body indented with spaces + `<<-` used | `<<-` strips **tabs only**, not spaces |
| 14 | `eval` on untrusted input | Sanitize/validate first — command-injection risk |
| 15 | Confusing `$[ ]` with `$(( ))` | `$[ ]` is deprecated legacy syntax; recognize it, don't write it |

---

### ✅ Final Revision Focus
1. Trace nested-loop output by hand (`break n` / `continue n`).
2. Distinguish BRE (`expr`) vs ERE (`[[ =~ ]]`) regex anchoring behavior.
3. Know which arithmetic tools are integer-only vs decimal-capable (`bc`).
4. Remember `exec` (replaces process) vs `eval` (returns control).
5. `select` and multi-variable `for` loops are the two most common "spot the bug" trap questions.

*End of Week 7 Revision Summary*
