# AWK — Week 9 Revision Summary

Quick-revision sheet for AWK. Every topic includes the concept, the key facts, and the edge cases/traps to remember for that topic (no separate "edge case" section — they're built into each row).

---

## 1. What is AWK & Where It Fits

| Point | Detail |
|---|---|
| Name | Abbreviation of creators' surnames: **A**ho, **W**einberger, **K**ernighan (Bell Labs, 1977) |
| Type | A full **programming language** for text organized into records & fields — not just a command |
| Standard | POSIX, **IEEE 1003.1-2008** |
| Variants | `nawk` (POSIX-close rewrite), `gawk` (GNU, most feature-rich, default on Linux), `mawk` (fast/minimal), `busybox awk`, `goawk`, "one true awk" |
| Where it sits | Between `grep` (match only) / `sed` (simple substitution) and full languages like Python/Perl |

**Edge case:** `gawk` adds extras beyond POSIX (`asort()`, `asorti()`, `strtonum()`, bit-wise functions). A script using these **may fail under `mawk`/`nawk`** — don't assume portability. Also, plain `awk` on Linux is usually a **symlink** to `gawk`; verify with `which awk` then `realpath $(which awk)`.

---

## 2. Execution Model (Read → Process → Write)

| Stage | What happens |
|---|---|
| Read | One record read at a time (default: one line, since `RS = "\n"`) |
| Process | Record checked against every pattern in the script; matching action(s) run |
| Write | `print`/`printf` output sent to stdout (or redirected file/pipe) |

Flow: `Input → split into RECORDS (RS) → split each record into FIELDS (FS) → for each record: check pattern → run action`.

| Edge case | Why it matters |
|---|---|
| Default `FS` is **not** exactly one space | It's *any run* of whitespace (spaces/tabs); multiple consecutive spaces count as one separator — only true when `FS` is left at default |
| Changing `FS` inside `BEGIN` | Applies from the **first** record |
| Changing `FS` mid-script (outside `BEGIN`) | Takes effect only from the **next** record, not the current one |
| `$0` vs `$1` | `$0` = whole record; `$1` = first field only — easy to confuse |
| `RS=""` | Switches to **paragraph mode** — blank line becomes the record separator (common `gawk` trick) |
| `FS` can be a regex | e.g. `-F'[,]+[ ]*'` — not limited to a single character |
| `RS` can be multi-character (gawk extension) | e.g. `RS=";"` |

---

## 3. Running AWK — Command Line vs Script

| Mode | Syntax | Notes |
|---|---|---|
| Command line | `awk -F"delim" '{action}' file` or `cmd \| awk '{action}'` | Quick one-liners |
| Script file | `#!/usr/bin/gawk -f` shebang, then `chmod +x`, run as `./myscript.awk file` | For reusable/longer programs |

**Key options:** `-F fs` (set FS), `-v var=value` (inject variable **before BEGIN runs**), `-f scriptfile` (repeatable, to combine multiple files e.g. a function library + main script).

| Edge case | Detail |
|---|---|
| Typographic quotes | Textbook slides sometimes show curly quotes `" "` — must be replaced with straight `"` or you get a syntax error |
| Pattern with no action | Default action is `{print $0}` — e.g. `awk '/Engineer/' file` behaves like `grep` |
| No pattern, just action | `awk '{print}'` behaves like `cat` |
| Multiple `-f` files | AWK logically concatenates them — lets you keep a function library separate from main logic |

---

## 4. Built-in Variables

### Record / Field / File

| Variable | Meaning |
|---|---|
| `NR` | Record number across **all** files (never resets) |
| `FNR` | Record number **within current file** (resets to 1 per new file) |
| `NF` | Number of fields in current record |
| `FS` / `RS` | Input field / record separator |
| `FILENAME` | Current file being processed |
| `ARGC` / `ARGV` | Count / array of command-line args |
| `ENVIRON` | Associative array of environment variables |

### Output / Matching

| Variable | Meaning |
|---|---|
| `OFS` | Output field separator (used when fields are rejoined by `print`) |
| `ORS` | Output record separator (end of each `print`) |
| `RSTART` / `RLENGTH` | Position / length matched by `match()` |
| `SUBSEP` | Separator for building multi-dimensional array keys |
| `OFMT` | Number output format |
| `$0` / `$n` | Whole record / n-th field |

**Edge case (classic exam trap):** `NR` = global counter, `FNR` = per-file counter. With `file1.txt` (2 lines) then `file2.txt` (2 lines): `NR` goes 1,2,3,4 but `FNR` goes 1,2,1,2 — `FNR` resets when a new file starts.

**Edge case (OFS trap):** Just setting `OFS = "\t"` does **not** rebuild `$0`. `$0` is only regenerated (using the new `OFS`) the moment you assign to a field (`$1="x"`) or to `NF`. Printing `$0` right after changing `OFS` — without touching a field — still shows the **old** separator.

---

## 5. Program Structure — Patterns, BEGIN, END

| Concept | Behavior |
|---|---|
| `pattern { action }` | The core unit — action runs only if pattern is true for that record |
| `BEGIN { }` | Runs **once**, before any input is read (setup: FS, headers, counters) |
| `END { }` | Runs **once**, after all input is processed (totals, summaries) |
| Multiple `BEGIN`/`END` | Allowed — they run **in the order written** |

**Edge case:** `BEGIN`/`END` blocks don't have `$0`/fields available in the usual sense unless you explicitly read (`getline`) — `NF`/`$1` etc. are only meaningful inside per-record blocks (or END, where they hold the last record's values).

---

## 6. Operators

| Category | Examples |
|---|---|
| Arithmetic | `+ - * / % ^` |
| Relational | `< <= > >= == !=` |
| Logical | `&& \|\| !` |
| Assignment | `= += -= *= /= %= ^=` |
| Increment/Decrement | `++ --` (prefix vs postfix) |
| Special | `?:` ternary, `in` (array membership test), `~` / `!~` (regex match / non-match), `$` (field reference), string concatenation by **juxtaposition** (no operator — `a b` means "a concatenated with b") |

**Edge case (string vs numeric comparison — very commonly tested):** AWK decides numeric vs string comparison based on context. If both operands **look numeric** (numeric constants, or numeric-looking values from input), comparison is **numeric**. If either is a plain string constant (quoted), comparison is **lexicographic**. So:
- `"10" < "9"` → **true** (string comparison: `"1"` < `"9"` char by char), even though 10 > 9 numerically.
- The same values coming from a data file (unquoted, numeric-looking) would compare **numerically** instead.

**Edge case (prefix vs postfix):** `++i` increments before returning the value; `i++` returns the old value first — subtle but affects output when used inline in expressions.

---

## 7. Standard Function Library

| Category | Functions |
|---|---|
| Arithmetic | `sqrt, int, rand, srand, sin, cos, log, exp, atan2` |
| String | `substr, split, gsub, sub, length, index, match, sprintf, tolower, toupper, strtonum*, asort*, asorti*` (*gawk extensions) |
| Control flow | `if, for, while, do-while, break, continue, return, exit` |
| I/O | `print, printf, getline, next, nextfile, close, fflush` |
| Extensions | `delete, function, system`, bit-wise: `and, or, xor, lshift, rshift, compl` |

**Edge case:** `rand()` without `srand()` produces the **same sequence every run** — always call `srand()` (often `srand(systime())`... though only seed once, e.g. in `BEGIN`) if you need different random numbers each execution.

---

## 8. Arrays

| Property | Detail |
|---|---|
| Type | **Associative** (like a hash map) — any string can be an index |
| Storage | **Sparse** — only assigned indices are stored, no empty slots |
| Iteration | `for (key in array)` — order is **not guaranteed** |
| Removal | `delete array[key]` |
| Multi-dimensional simulation | Composite keys joined with `SUBSEP`, e.g. `arr[i, j]` internally becomes `arr[i SUBSEP j]` |

**Edge case:** Because arrays are associative and unordered, iterating with `for (k in arr)` can produce keys in **any order** — never assume insertion or numeric order unless you sort explicitly (e.g. with `asort`/`asorti` in gawk, or by collecting keys and sorting them yourself).

---

## 9. Loops & Conditionals

| Construct | Runs | Use case |
|---|---|---|
| `for (a in array)` | Once per array index | Iterating associative arrays |
| `for (i=1;i<n;i++)` | While condition true, with update step | Fixed-count loops |
| `while` | Condition checked **before** each iteration | May run zero times |
| `do...while` | Condition checked **after** — runs **at least once** | Guarantees one execution |
| `if / else`, `if/else if/else` | Only if condition true | Branching |

**Edge case:** `while` vs `do-while` — if the condition is false from the start, `while` never executes the body, but `do-while` still executes it **once**. Pick based on whether "at least one run" is required.

---

## 10. User-Defined Functions

```awk
function name(params) { ...; return value }
```

| Rule | Detail |
|---|---|
| Scalars | Passed **by value** — a copy; changes inside don't affect caller's variable |
| Arrays | Passed **by reference** — changes inside DO affect caller's array |
| Extra parameters | Calling with fewer arguments than declared → remaining parameters act as **local variables** (AWK's only way to simulate `local`) |
| Recursion | Supported |
| Multiple files | Combine a function library + main script via repeated `-f` flags |

**Edge case:** Because AWK has no `local` keyword, the common pattern is `function f(a, b,   localvar1, localvar2)` — note the **extra blank-looking parameters after gap/comma** that are never passed by the caller; they exist purely to act as function-local scratch variables. Forgetting this and reusing a global-sounding name inside a function can silently clobber a global variable.

---

## 11. `printf` — Formatted Output

| Control letter | Meaning |
|---|---|
| `d` / `i` | Integer |
| `f` | Floating point |
| `e` | Scientific notation |
| `g` | Shorter of `e` or `f` |
| `s` | String |
| `c` | ASCII character |
| `o` | Octal |
| `x` / `X` | Hex (lower/upper) |

**Modifiers:** `width` (min field width, pad with spaces or zeros), `.prec` (decimal digits for floats; max chars for strings). `-` flag left-aligns (default is right-align).

**Edge case:** `printf` does **not** add a newline automatically (unlike `print`) — you must include `\n` yourself, or output from consecutive `printf` calls runs together on one line.

---

## 12. Combining AWK with Bash

| Technique | Example |
|---|---|
| Embed in shell script | AWK command/script inside a `.sh` file |
| Heredoc | `awk << 'EOF' ... EOF` — multi-line program without a separate `.awk` file |
| Pass shell variables in | `awk -v t="$THRESHOLD" '...'` |
| Pipe with other tools | `grep "ERROR" file \| awk '{print $1,$NF}' \| sort` |
| Frequency counting | `awk '{print $3}' file \| sort \| uniq -c` |

**Edge case:** In a heredoc, use `<< 'EOF'` (quoted delimiter) if the AWK script contains `$` variables that should be interpreted by **AWK**, not expanded by **bash** first. An unquoted `<< EOF` lets bash substitute `$1`, `$NF`, etc. before AWK ever sees them — usually not what you want.

---

## 13. Why AWK Matters / When to Use It

| Point | Explanation |
|---|---|
| Universally available | Pre-installed on virtually every Unix/Linux system |
| Quick to code | Concise pattern-action syntax |
| Fast | Efficient for large files — streams records with **constant memory** |
| Composable | Works well in pipelines |

| Task | Best tool |
|---|---|
| Simple line filter | `grep` |
| Simple substitution | `sed` |
| Field extraction, math, reports | **AWK** |
| Complex logic, libraries, web calls | Python/Perl |
| Sort/count uniques | `sort`, `uniq` (often piped with AWK) |

**Edge case:** AWK is preferred over spreadsheet apps for very large files (millions of rows) because spreadsheets load everything into memory and enforce row limits, while AWK streams one record at a time.

---

## 14. Worked Example Pattern (Payroll-style Script)

Typical structure combining everything above:

```awk
BEGIN { FS=" "; OFS="\t"; print "Header" }
NF == 0 { next }                     # skip blank lines
{
    total[$2] += $4                  # associative array, grouping by field
    count[$2]++
}
END {
    for (d in total)
        printf "%-10s %10.2f\n", d, total[d]
}
```

| Concept reused here | Where |
|---|---|
| `BEGIN` setup | FS/OFS + header |
| `next` | Skips unwanted records (blank lines) |
| Associative arrays | Grouping/summing by a field value |
| `printf` | Aligned tabular report |
| `END` | Final aggregation/summary |

**Edge case:** `NF == 0 { next }` is the standard idiom to skip blank lines — remember `next` jumps straight to reading the **next record**, skipping any remaining pattern-action blocks for the current one (don't confuse with `nextfile`, which skips to the next **file**).

---

## One-Page Recall Table

| If you're asked about... | Remember |
|---|---|
| Acronym | Aho, Weinberger, Kernighan |
| Standard | POSIX IEEE 1003.1-2008 |
| NR vs FNR | NR = global, FNR = per-file |
| OFS after change | `$0` unchanged until a field/NF is reassigned |
| `"10" < "9"` | True — string comparison, not numeric |
| Arrays | Associative, sparse, unordered iteration |
| Function params | Scalars by value, arrays by reference, extra params = locals |
| `while` vs `do-while` | do-while always runs ≥ 1 time |
| `printf` vs `print` | printf needs manual `\n` |
| `next` vs `nextfile` | next = next record, nextfile = next file |
| Default FS | Any whitespace run, not exactly one space |
| FS change timing | BEGIN → from record 1; mid-script → from next record |
