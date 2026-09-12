# Week 4 — Pattern Matching: Regex & grep
### (Lecture 1 & Lecture 2 — Complete Exam Notes)

---

## Table of Contents

1. [Introduction to Regex](#1-introduction-to-regex)
   - 1.1 [What is Regex?](#11-what-is-regex)
   - 1.2 [POSIX Standard](#12-posix-standard)
   - 1.3 [BRE vs ERE](#13-bre-vs-ere)
2. [Why Learn Regex?](#2-why-learn-regex)
3. [Basic Usage & Syntax](#3-basic-usage--syntax)
   - 3.1 [grep Command Syntax](#31-grep-command-syntax)
   - 3.2 [Switching to ERE (egrep / grep -E)](#32-switching-to-ere-egrep--grep--e)
4. [Special Characters — Common to BRE & ERE](#4-special-characters--common-to-bre--ere)
5. [Special Characters — BRE Only](#5-special-characters--bre-only)
6. [Special Characters — ERE Only](#6-special-characters--ere-only)
7. [Character Classes (POSIX Bracket Expressions)](#7-character-classes-posix-bracket-expressions)
8. [Backreferences](#8-backreferences)
9. [Operator Precedence](#9-operator-precedence)
   - 9.1 [BRE Operator Precedence](#91-bre-operator-precedence)
   - 9.2 [ERE Operator Precedence](#92-ere-operator-precedence)
10. [Complete Reference — Everything About `grep`](#10-complete-reference--everything-about-grep)
    - 10.1 [What is `grep`?](#101-what-is-grep)
    - 10.2 [General Syntax](#102-general-syntax)
    - 10.3 [The Three Grep "Flavors"](#103-the-three-grep-flavors)
    - 10.4 [Complete Options Table](#104-complete-options-table)
    - 10.5 [Option-by-Option Examples](#105-option-by-option-examples)
    - 10.6 [Exit Status of `grep`](#106-exit-status-of-grep)
    - 10.7 [Combining Multiple Flags](#107-combining-multiple-flags)
    - 10.8 [grep Options — Quick Reference Table](#108-grep-options--quick-reference-table)
11. [Lecture 1 — grep Practical Walkthrough (Anchors, Quantifiers, Backreferences, ERE)](#11-lecture-1--grep-practical-walkthrough)
12. [Lecture 2 — Character Classes, Real-World Patterns & Practical Filtering](#12-lecture-2--character-classes-real-world-patterns--practical-filtering)
13. [The `cut` Command — Syntax and Examples](#13-the-cut-command--syntax-and-examples)
14. [Summary](#14-summary)

---

## 1. Introduction to Regex

### 1.1 What is Regex?

A **regular expression (regex)** is a pattern template used to search, match, and filter text based on defined rules rather than exact strings.

- A regex describes a *set of strings* that match a particular pattern.
- Two POSIX-defined regex engines exist:
  - **BRE** — POSIX **Basic** Regular Expression engine
  - **ERE** — POSIX **Extended** Regular Expression engine

### 1.2 POSIX Standard

Regex syntax used by Linux/Unix tools follows the **POSIX standard**:

| Reference | Detail |
|---|---|
| Standard | IEEE 1003.1-2001 |
| Full Name | IEEE Standard for Information Technology – Portable Operating System Interface (POSIX™) |
| Link | https://standards.ieee.org/standard/1003_1-2001.html |

### 1.3 BRE vs ERE

| Feature | BRE (Basic) | ERE (Extended) |
|---|---|---|
| Grouping | `\( \)` | `( )` |
| Repetition range | `\{n,m\}` | `{n,m}` |
| One or more | Not available (only via `\{1,\}`) | `+` |
| Zero or one | Not available (only via `\{0,1\}`) | `?` |
| Logical OR | Not available | `\|` |
| Default engine for `grep` | ✅ Yes | ❌ No (needs `-E`) |

---

## 2. Why Learn Regex?

Regex is a *universal* skill because it is used across:

- **Languages:** Java, Perl, Python, Ruby, etc.
- **Tools:** `grep`, `sed`, `awk`, etc.
- **Applications/Databases:** MySQL, PostgreSQL, etc.

> **Exam Tip:** Regex is not tied to one tool — the same pattern logic (BRE/ERE) reappears across shell tools and programming languages, so mastering the syntax once saves time everywhere.

---

## 3. Basic Usage & Syntax

### 3.1 grep Command Syntax

**Syntax:**
```bash
grep 'pattern' filename
command | grep 'pattern'
```

- `grep` searches `filename` (or piped input) line-by-line and prints every line that **matches** the given pattern.
- **Default engine used by `grep` is BRE** (Basic Regular Expression).

**Example:**
```bash
grep 'error' server.log
```
Prints every line in `server.log` containing the word `error`.

### 3.2 Switching to ERE (egrep / grep -E)

To use the more powerful **Extended Regular Expression** syntax (with `+`, `?`, `|`, unescaped `( )`, `{ }`):

**Syntax:**
```bash
egrep 'pattern' filename
# OR (equivalent, preferred modern form)
grep -E 'pattern' filename
```

**Example:**
```bash
grep -E 'cat|dog' pets.txt
```
Matches any line containing **either** `cat` or `dog`.

---

## 4. Special Characters — Common to BRE & ERE

These metacharacters behave the same in **both** BRE and ERE:

| Symbol | Meaning |
|---|---|
| `.` | Any single character except null or newline |
| `*` | Zero or more of the preceding character/expression |
| `[ ]` | Any one of the enclosed characters; hyphen (`-`) indicates a character range |
| `^` | Anchor for beginning of line, OR negation when used inside `[ ]` |
| `$` | Anchor for end of line |
| `\` | Escape character — used to treat a special character literally, or to activate special meaning of a normal character (BRE) |

### Detailed Explanation Table

| Symbol | Concept | Example Pattern | What It Matches |
|---|---|---|---|
| `.` | Wildcard for one character | `c.t` | `cat`, `cot`, `c9t` |
| `*` | Repetition (0 or more of preceding element) | `ab*c` | `ac`, `abc`, `abbbc` |
| `[ ]` | Character set / range | `[a-c]` | any one of `a`, `b`, `c` |
| `^` | Start-of-line anchor | `^Error` | lines that *begin with* `Error` |
| `^` (inside brackets) | Negation | `[^0-9]` | any character that is **not** a digit |
| `$` | End-of-line anchor | `done$` | lines that *end with* `done` |
| `\` | Escape metacharacter | `\.` | a literal dot `.` (not "any character") |

---

## 5. Special Characters — BRE Only

| Symbol | Meaning |
|---|---|
| `\{n,m\}` | Range of occurrences of the preceding pattern — at least `n`, at most `m` times |
| `\( \)` | Grouping of regular expressions |

**Example:**
```bash
grep 'lo\{2,4\}l' file.txt
```
Matches `lool`, `loool`, `looool` (2 to 4 `o`'s between `l` and `l`).

---

## 6. Special Characters — ERE Only

| Symbol | Meaning |
|---|---|
| `{n,m}` | Range of occurrences of the preceding pattern — at least `n`, at most `m` times |
| `( )` | Grouping of regular expressions |
| `+` | One or more of the preceding character/expression |
| `?` | Zero or one of the preceding character/expression |
| `\|` | Logical OR over the patterns |

**Example:**
```bash
grep -E '(ma)+' file.txt
```
Matches `ma`, `mama`, `mamama` (one or more repetitions of the group `ma`).

---

## 7. Character Classes (POSIX Bracket Expressions)

POSIX character classes are used **inside** bracket expressions (`[[:class:]]`) to match categories of characters.

| Class | Meaning | Class | Meaning |
|---|---|---|---|
| `[[:print:]]` | Printable characters | `[[:blank:]]` | Space / Tab |
| `[[:alnum:]]` | Alphanumeric characters | `[[:space:]]` | Whitespace (space, tab, newline, etc.) |
| `[[:alpha:]]` | Alphabetic characters | `[[:punct:]]` | Punctuation |
| `[[:lower:]]` | Lower case letters | `[[:xdigit:]]` | Hexadecimal digits |
| `[[:upper:]]` | Upper case letters | `[[:graph:]]` | Non-space (visible) characters |
| `[[:digit:]]` | Decimal digits | `[[:cntrl:]]` | Control characters |

### Expanded Notes

- Character classes **must** be used inside a bracket expression: `[[:digit:]]` is correct; `:digit:` alone is invalid.
- They are **locale-aware** — e.g., `[[:alpha:]]` may include accented letters depending on system locale.
- Multiple classes can be combined inside one bracket: `[[:upper:][:digit:]]` matches an uppercase letter **or** a digit.
- `[[:punct:]]` and `[[:graph:]]` overlap conceptually — `[[:graph:]]` = all printable characters **except space**, whereas `[[:print:]]` = printable characters **including space**.

**Example:**
```bash
grep '[[:digit:]]' file.txt
```
Prints only the lines that contain **at least one digit**.

---

## 8. Backreferences

- Backreferences allow a regex to refer back to text matched by an **earlier parenthesized group**.
- Syntax: `\1` through `\9`
- `\n` matches whatever text was matched by the **n-th earlier parenthesized subexpression**.

**Example — matching a line with two occurrences of "hello":**
```bash
grep '\(hello\).*\1' file.txt
```
- `\(hello\)` — captures `hello` as group 1
- `.*` — any characters in between
- `\1` — requires the *same* captured text (`hello`) to reappear again

> **Exam Tip:** Backreferences refer to *matched text*, not to the pattern itself — so `\(a*\)` matched against `aaa` sets `\1 = "aaa"`, and `\1` in the rest of the expression will only match `aaa` again, not `a`, `aa`, or any other number of `a`'s.

---

## 9. Operator Precedence

Understanding precedence tells you which operators are evaluated first when multiple metacharacters appear together in one pattern.

### 9.1 BRE Operator Precedence

*(highest priority at top → lowest at bottom)*

| Priority | Operator(s) | Purpose |
|---|---|---|
| 1 (Highest) | `[..]` `[==]` `[::]` | Character collation (collating symbols, equivalence classes, character classes) |
| 2 | `\metachar` | Escaped metacharacter |
| 3 | `[ ]` | Bracket expression |
| 4 | `\( \)` and `\n` | Subexpressions and backreferences |
| 5 | `*` `\{ \}` | Repetition of the preceding single-character regex |
| 6 | (concatenation) | Joining adjacent expressions |
| 7 (Lowest) | `^` `$` | Anchors |

### 9.2 ERE Operator Precedence

*(highest priority at top → lowest at bottom)*

| Priority | Operator(s) | Purpose |
|---|---|---|
| 1 (Highest) | `[..]` `[==]` `[::]` | Character collation |
| 2 | `\metachar` | Escaped metacharacter |
| 3 | `[ ]` | Bracket expansion |
| 4 | `( )` | Grouping |
| 5 | `*` `+` `?` `{ }` | Repetition of the preceding regex |
| 6 | (concatenation) | Joining adjacent expressions |
| 7 | `^` `$` | Anchors |
| 8 (Lowest) | `\|` | Alternation |

> **Exam Tip:** In ERE, alternation (`|`) has the **lowest** precedence — meaning `ab|cd` matches `ab` OR `cd` as whole tokens, not `a(b|c)d`. Use grouping `(ab|cd)` to control scope explicitly.

---

## 10. Complete Reference — Everything About `grep`

> This section covers `grep` itself as a command-line tool (its purpose, general syntax, and full option set) **before** moving into the practical, pattern-by-pattern coding walkthrough from the lectures (Sections 11–12 below). Think of this as the "theory of the tool," and Sections 11–12 as "the tool in action."

### 10.1 What is `grep`?

**`grep`** stands for **G**lobal **R**egular Expression **P**rint. It is a command-line utility that searches one or more files (or standard input) line-by-line for text matching a given pattern, and prints the matching lines.

- Originates from the Unix `ed` editor command `g/re/p` (**g**lobally search for a **r**egular **e**xpression and **p**rint matching lines) — this is literally where the name comes from.
- It is one of the most fundamental text-processing tools in Linux/Unix, used constantly for log analysis, searching source code, filtering command output, and validating data.

### 10.2 General Syntax

**Syntax:**
```bash
grep [OPTIONS] PATTERN [FILE...]
```

| Part | Meaning |
|---|---|
| `OPTIONS` | Flags that modify behavior (case-insensitivity, inverting, counting, etc.) |
| `PATTERN` | The regular expression (or literal string) to search for |
| `FILE...` | One or more files to search. If omitted, `grep` reads from **standard input** (e.g., from a pipe) |

**Example (no file — reads from stdin):**
```bash
ps aux | grep firefox
```

**Example (with file):**
```bash
grep 'firefox' processes.txt
```

**Example (multiple files):**
```bash
grep 'error' log1.txt log2.txt log3.txt
```
When searching multiple files, `grep` automatically **prefixes each output line with the filename** it was found in.

### 10.3 The Three Grep "Flavors"

| Command | Regex Engine Used | Equivalent To |
|---|---|---|
| `grep` | BRE (Basic Regular Expression) | `grep -G` |
| `egrep` | ERE (Extended Regular Expression) | `grep -E` |
| `fgrep` | Fixed string (no regex at all — pattern treated literally) | `grep -F` |

> **Exam Tip:** `egrep` and `fgrep` are historically separate binaries, but on modern systems they are usually just aliases/wrappers for `grep -E` and `grep -F` respectively. GNU documentation now considers `egrep`/`fgrep` deprecated in favor of explicitly using `-E`/`-F`.

### 10.4 Complete Options Table

| Option | Long Form | Meaning |
|---|---|---|
| `-i` | `--ignore-case` | Case-insensitive matching |
| `-v` | `--invert-match` | Print lines that **do NOT** match |
| `-c` | `--count` | Print only a **count** of matching lines (not the lines themselves) |
| `-n` | `--line-number` | Prefix each matching line with its **line number** |
| `-l` | `--files-with-matches` | Print only the **names of files** that contain a match |
| `-L` | `--files-without-match` | Print only the names of files that **do NOT** contain a match |
| `-w` | `--word-regexp` | Match only **whole words** (pattern must be surrounded by word boundaries) |
| `-x` | `--line-regexp` | Match only if the pattern matches the **entire line** |
| `-o` | `--only-matching` | Print only the **matched portion** of each line, not the whole line |
| `-E` | `--extended-regexp` | Use **ERE** syntax (same as `egrep`) |
| `-F` | `--fixed-strings` | Treat the pattern as a **literal string**, not a regex (same as `fgrep`) |
| `-G` | `--basic-regexp` | Use **BRE** syntax (this is the default) |
| `-P` | `--perl-regexp` | Use **Perl-Compatible Regular Expressions** (PCRE) — supports lookaheads/lookbehinds |
| `-e PATTERN` | `--regexp=PATTERN` | Specify a pattern explicitly — useful for **multiple patterns** in one call |
| `-f FILE` | `--file=FILE` | Read patterns from a file (one pattern per line) |
| `-r` / `-R` | `--recursive` | Search **recursively** through directories |
| `-A NUM` | `--after-context=NUM` | Show `NUM` lines **after** each match |
| `-B NUM` | `--before-context=NUM` | Show `NUM` lines **before** each match |
| `-C NUM` | `--context=NUM` | Show `NUM` lines **before and after** each match |
| `-q` | `--quiet` / `--silent` | Suppress all output; only the **exit status** signals a match |
| `-s` | `--no-messages` | Suppress error messages (e.g., for missing files) |
| `-H` | `--with-filename` | Always print the filename, even for a single file |
| `-h` | `--no-filename` | Never print the filename (useful with multiple files) |
| `-m NUM` | `--max-count=NUM` | Stop reading a file after `NUM` matches |
| `-z` | `--null-data` | Treat input as a set of lines terminated by a null character, not newline |
| `-a` | `--text` | Treat binary files as if they were text |
| `--color` | `--color=auto` | Highlight the matching portion of text in color |

### 10.5 Option-by-Option Examples

**`-i` — Case-Insensitive**
```bash
grep -i 'error' log.txt
```
Matches `error`, `Error`, `ERROR`, `ErRoR`.

**`-v` — Invert Match**
```bash
grep -v 'debug' log.txt
```
Prints every line that does **not** contain `debug`.

**`-c` — Count Matches**
```bash
grep -c 'error' log.txt
```
Output: a single number, e.g. `42` — the count of matching lines (not the lines themselves).

**`-n` — Line Numbers**
```bash
grep -n 'error' log.txt
```
Output:
```
15:connection error occurred
102:disk error detected
```

**`-l` — Filenames Only**
```bash
grep -l 'error' log1.txt log2.txt log3.txt
```
Output: only the names of files that contain at least one match, e.g.:
```
log1.txt
log3.txt
```

**`-w` — Whole Word Match**
```bash
grep -w 'cat' file.txt
```
Matches the standalone word `cat`, but **not** `category` or `concatenate`.

**`-x` — Whole Line Match**
```bash
grep -x 'done' status.txt
```
Matches only lines where the **entire line** is exactly `done` — not `task done` or `done now`.

**`-o` — Only the Matched Text**
```bash
echo "order id: 12345" | grep -o '[[:digit:]]\+'
```
Output: `12345` (only the matched digits, not the whole line).

**`-e` — Multiple Patterns**
```bash
grep -e 'error' -e 'warning' log.txt
```
Matches lines containing **either** `error` or `warning` (equivalent to `grep -E 'error|warning'`).

**`-r` / `-R` — Recursive Search**
```bash
grep -r 'TODO' ./project/
```
Searches every file inside `./project/` (and its subdirectories) for the word `TODO`.

**`-A`, `-B`, `-C` — Context Lines**
```bash
grep -A 2 'Exception' app.log     # 2 lines AFTER each match
grep -B 2 'Exception' app.log     # 2 lines BEFORE each match
grep -C 2 'Exception' app.log     # 2 lines BEFORE and AFTER each match
```
This is extremely useful for debugging — seeing the surrounding context of an error, not just the error line itself.

**`-q` — Quiet Mode (for Scripts)**
```bash
if grep -q 'error' log.txt; then
    echo "Errors found!"
fi
```
No output is printed by `grep` itself — only its **exit status** (`0` if a match was found, `1` if not) is used.

**`-m` — Limit Number of Matches**
```bash
grep -m 3 'error' log.txt
```
Stops after finding the **first 3** matching lines, even if more exist further in the file.

**`-P` — Perl-Compatible Regex (Lookaheads/Lookbehinds)**
```bash
echo "price: $500" | grep -oP '(?<=\$)\d+'
```
Output: `500` — extracts only the digits that come **after** a `$` sign, using a lookbehind (a feature not available in plain BRE/ERE).

### 10.6 Exit Status of `grep`

`grep`'s exit status is often used in shell scripts to make decisions:

| Exit Code | Meaning |
|---|---|
| `0` | At least one line matched |
| `1` | No lines matched |
| `2` | An error occurred (e.g., file not found, invalid pattern) |

**Example:**
```bash
grep -q 'root' /etc/passwd && echo "Found" || echo "Not found"
```

### 10.7 Combining Multiple Flags

Flags can be combined/stacked, since most are single-letter short options.

**Example:**
```bash
grep -inv 'error' log.txt
```
This applies **three flags at once**:
- `-i` → case-insensitive
- `-n` → show line numbers
- `-v` → invert match (show lines that do **not** contain the pattern, case-insensitively)

**Example (real-world combo):**
```bash
grep -rniw 'password' ./src/
```
Recursively (`-r`) searches `./src/`, case-insensitively (`-i`), for the **whole word** (`-w`) `password`, and shows line numbers (`-n`) — a common security-auditing style command.

### 10.8 grep Options — Quick Reference Table

| Category | Key Flags |
|---|---|
| Case handling | `-i` |
| Matching logic | `-v` (invert), `-w` (whole word), `-x` (whole line), `-o` (only match) |
| Output control | `-c` (count), `-n` (line no.), `-l`/`-L` (filenames), `-H`/`-h` (show/hide filename), `--color` |
| Regex engine | `-E` (ERE), `-G` (BRE), `-F` (fixed string), `-P` (Perl regex) |
| Multiple patterns | `-e`, `-f` |
| File traversal | `-r` / `-R` |
| Context | `-A`, `-B`, `-C` |
| Scripting | `-q` (quiet), `-s` (no error msgs), exit status |
| Limits | `-m` |

---

## 11.  grep Practical Walkthrough

> Now that the full `grep` command reference (options, syntax, exit codes) has been covered above, this section moves into the **practical, pattern-by-pattern coding walkthrough** from the lecture — applying regex concepts directly with `grep`. Corresponds to lecture timestamps **14:03 → 37:58**. Each command below is explained with concept, syntax, and a worked example. (Note: `ˆ` in the original slides is a rendering artifact of `^`, and `\.\*` is a rendering artifact of `.*` — both corrected below.)

### 11.1 Basic Pattern Match — `grep 'pattern' file`

**Syntax:**
```bash
grep 'pattern' file
```
Searches `file` and prints every line containing `pattern`.

**Example:**
```bash
grep 'apple' fruits.txt
```
Input (`fruits.txt`):
```
apple pie
banana bread
green apple
```
Output:
```
apple pie
green apple
```

### 11.2 Piping Input — `cat file | grep pattern`

**Syntax:**
```bash
cat file | grep pattern
```
Same result as `grep pattern file`, but the file content is streamed through a pipe into `grep`. Useful when chaining multiple commands.

**Example:**
```bash
cat fruits.txt | grep apple
```
Output: (same as above)
```
apple pie
green apple
```

### 11.3 Two Words Concatenated — `grep 'pattern.pattern'`

**Concept:** The `.` matches **exactly one** character, so `pattern.pattern` looks for the literal word `pattern`, then any single character, then `pattern` again.

**Example:**
```bash
grep 'cat.cat' file.txt
```
Matches lines with `cat` + any 1 character + `cat`, e.g., `catXcat`, `cat1cat`, `cat cat` (space counts as one character).

### 11.4 End-of-Line Anchor — `grep 'pattern$'`

**Syntax:**
```bash
grep 'pattern$'  file
```
`$` anchors the match to the **end of the line** — the line must *end with* `pattern`.

**Example:**
```bash
grep 'done$' status.txt
```
Input:
```
task done
done deal
job done
```
Output:
```
task done
job done
```

### 11.5 Escaping the Dot — `grep '\.'`

**Concept:** `.` normally matches any character. To match a **literal dot**, escape it with `\`.

**Example:**
```bash
grep '\.' filenames.txt
```
Matches only lines containing an actual period, e.g., `report.txt`, `v1.0`.

### 11.6 Start-of-Line Anchor — `grep '^pattern'`

**Syntax:**
```bash
grep '^pattern' file
```
`^` anchors the match to the **start of the line**.

**Example:**
```bash
grep '^Error' log.txt
```
Matches only lines that **begin with** `Error`.

### 11.7 Word Boundary — `grep 'pattern\b'`

**Concept:** `\b` (a GNU `grep` extension, not core POSIX) matches a **word boundary** — the position between a word character and a non-word character. `pattern\b` ensures `pattern` ends exactly at a word boundary (not glued to more letters).

**Example:**
```bash
grep 'cat\b' file.txt
```
Matches `cat`, `cat.`, `cat,` but **not** `category` (because `t` is followed by `e`, not a boundary).

### 11.8 Bracket Expression — `grep 'patt[ern]'`

**Concept:** `[ern]` matches **any one** of the characters `e`, `r`, or `n` (not the literal sequence "ern").

**Example:**
```bash
grep 'patt[ern]' file.txt
```
Matches `patte`, `pattr`, `pattn` — i.e., `patt` followed by exactly one of `e`, `r`, or `n`.

### 11.9 Dot-Star Wildcard — `grep 'pat.*tern'`

**Concept:** `.*` = "any character, zero or more times" → matches anything (or nothing) between `pat` and `tern`.

**Example:**
```bash
grep 'pat.*tern' file.txt
```
Matches `pattern`, `pat123tern`, `pat tern`, and even `patterntern`.

### 11.10 Word Boundary + Wildcard — `grep '\bpat.*tern'`

**Concept:** Combines a word-boundary anchor at the start with the `.*` wildcard, ensuring the match starts at the beginning of a word.

**Example:**
```bash
grep '\bpat.*tern' file.txt
```
Matches `pattern` at the start of a word, but not `repattern` (since `pat` there isn't at a word boundary — it's preceded by `re`).

### 11.11 Character Range — `grep 'pat[1-5]tern'`

**Concept:** `[1-5]` matches any **one digit** in the range 1 to 5.

**Example:**
```bash
grep 'pat[1-5]tern' file.txt
```
Matches `pat1tern`, `pat3tern`, `pat5tern`, but **not** `pat7tern` or `pat9tern`.

### 11.12 Negated Character Range — `grep 'pat[^1-5]tern'`

**Concept:** `^` inside `[ ]` negates the set — matches any character **except** 1–5.

**Example:**
```bash
grep 'pat[^1-5]tern' file.txt
```
Matches `patXtern`, `pat9tern`, `pat8tern` — but **not** `pat3tern`.

### 11.13 BRE Interval — `grep 'pattern\{2,4\}'`

**Concept:** `\{2,4\}` repeats the **immediately preceding single character** (here, `n`) between 2 and 4 times — it does **not** repeat the whole word "pattern".

**Example:**
```bash
grep 'pattern\{2,4\}' file.txt
```
Matches `patternn`, `patternnn`, `patternnnn` (i.e., `patter` + 2–4 `n`'s).

> **Exam Tip:** A very common exam mistake is assuming `\{ \}` repeats the whole preceding word. It only repeats the **single character or group** immediately before it. To repeat an entire word, wrap it in a group first: `\(pattern\)\{2,4\}`.

### 11.14 Literal Repetition — `grep 'patternpattern'`

**Concept:** Simple literal string search — no metacharacters, matches the exact literal text `patternpattern`.

**Example:**
```bash
grep 'patternpattern' file.txt
```
Matches only lines containing the exact literal substring `patternpattern`.

### 11.15 Backreference Match — `grep 'patternpattern.*\1'`

**Concept:** This requires a **capturing group** to use `\1` — as written, this pattern only works correctly if `pattern` is wrapped in `\( \)`, e.g., `\(pattern\)pattern.*\1`. It demonstrates using a backreference to require the captured group to reappear later in the line.

**Example (corrected form):**
```bash
grep '\(pattern\)pattern.*\1' file.txt
```
Matches lines where `patternpattern` appears, followed (anywhere later) by another occurrence of `pattern`.

### 11.16 Backreference + Interval — `grep 'patternpattern\{2,3\}'`

**Concept:** Combines literal repetition with an interval on the last character. `\{2,3\}` applies only to the final `n` in the second `pattern`.

**Example:**
```bash
grep 'patternpattern\{2,3\}' file.txt
```
Matches `patternpatternn` or `patternpatternnn` (2–3 `n`'s at the very end).

### 11.17 ERE One-or-More — `egrep 'M+'`

**Concept:** `+` (ERE only) means "one or more of the preceding character."

**Example:**
```bash
egrep 'M+' file.txt
```
Matches any line containing one or more consecutive `M`'s: `M`, `MM`, `MMM`, etc.

### 11.18 Anchored One-or-More — `egrep '^M+'`

**Concept:** Combines `^` (start anchor) with `M+` — line must **begin** with one or more `M`'s.

**Example:**
```bash
egrep '^M+' file.txt
```
Matches lines starting with `M`, `MM`, `MMorning`, but not `AM`.

### 11.19 Anchored Zero-or-More — `egrep '^M*'`

**Concept:** `*` means zero or more — so `^M*` technically matches **every line** (even zero `M`'s satisfies it), but is typically used/demonstrated alongside other patterns to show the contrast with `+`.

**Example:**
```bash
egrep '^M*' file.txt
```
Matches all lines (since 0 occurrences of `M` also satisfies `M*`) — highlighting why `+` (one-or-more) is usually the more *useful* choice when you want to **require** at least one occurrence.

### 11.20 `M*a` vs `M.*a` — Comparison

**Concept:**
- `M*a` → zero or more **`M` characters only**, immediately followed by `a`.
- `M.*a` → an `M`, followed by **any characters**, followed by `a`.

**Example:**
```bash
egrep 'M*a' file.txt      # matches: a, Ma, MMa   (NOT "Mxa")
egrep 'M.*a' file.txt     # matches: Ma, Mxa, M123a
```

| Pattern | Meaning | Matches |
|---|---|---|
| `M*a` | 0+ literal `M`, then `a` | `a`, `Ma`, `MMa` |
| `M.*a` | `M`, then 0+ any character, then `a` | `Ma`, `Mxa`, `M123a` |

### 11.21 Grouped Repetition (One or More) — `egrep '(ma)+'`

**Concept:** `+` applied to a **group** `(ma)` repeats the whole group one or more times.

**Example:**
```bash
egrep '(ma)+' file.txt
```
Matches `ma`, `mama`, `mamama`.

### 11.22 Grouped Repetition (Zero or More) — `egrep '(ma)*'`

**Concept:** Same as above but with `*` — zero or more repetitions of the group `(ma)`.

**Example:**
```bash
egrep '(ma)*' file.txt
```
Matches empty string, `ma`, `mama`, etc. (technically matches almost everything since 0 repetitions is valid).

### 11.23 Alternation — `egrep '(ED|ME)'`

**Concept:** `|` = logical OR. Matches lines containing **either** `ED` or `ME`.

**Example:**
```bash
egrep '(ED|ME)' file.txt
```
Input:
```
LOVED
NAMED
SOME
XYZ
```
Output:
```
LOVED
NAMED
SOME
```

---

## 12. Character Classes, Real-World Patterns & Practical Filtering

> Corresponds to lecture timestamps **01:12 → 25:32**.

### 12.1 Real-World Use — `dpkg-query | grep`

**Concept:** `dpkg-query` lists installed Debian/Ubuntu packages; piping it through `grep` filters the list for a specific package name — a very common sysadmin use case for regex.

**Syntax:**
```bash
dpkg-query -l | grep 'pattern'
```

**Example:**
```bash
dpkg-query -l | grep 'python'
```
Prints only the installed packages whose names/descriptions contain `python`.

### 12.2 Alphabetic Characters — `grep '[[:alpha:]]'`

**Syntax:**
```bash
grep '[[:alpha:]]' file
```
Matches lines that contain **at least one alphabetic character**.

**Example:** Filters out lines that are purely numeric or symbolic, keeping only lines with letters.

### 12.3 Alphanumeric Characters — `grep '[[:alnum:]]'`

**Syntax:**
```bash
grep '[[:alnum:]]' file
```
Matches lines containing **at least one letter or digit**.

### 12.4 Digits Only — `grep '[[:digit:]]'`

**Syntax:**
```bash
grep '[[:digit:]]' file
```
Matches lines containing **at least one numeric digit**.

### 12.5 Control Characters — `grep '[[:cntrl:]]'`

**Syntax:**
```bash
grep '[[:cntrl:]]' file
```
Matches lines containing non-printable **control characters** (e.g., tab, carriage return, backspace — ASCII 0–31 and 127).

### 12.6 Inverted Match (No Control Characters) — `grep -v '[[:cntrl:]]'`

**Syntax:**
```bash
grep -v '[[:cntrl:]]' file
```
The `-v` flag **inverts** the match — prints lines that **do NOT** contain the pattern. Here, it prints only "clean" lines with no control characters.

> **Exam Tip:** `-v` is one of the most commonly tested `grep` flags — remember it inverts the *entire line match*, not individual characters.

### 12.7 Punctuation — `grep '[[:punct:]]'`

**Syntax:**
```bash
grep '[[:punct:]]' file
```
Matches lines containing punctuation symbols (`. , ! ? ; : ' " ( ) -` etc.).

### 12.8 Lowercase Letters — `grep '[[:lower:]]'`

**Syntax:**
```bash
grep '[[:lower:]]' file
```
Matches lines containing at least one **lowercase** letter.

### 12.9 Uppercase Letters — `grep '[[:upper:]]'`

**Syntax:**
```bash
grep '[[:upper:]]' file
```
Matches lines containing at least one **uppercase** letter.

### 12.10 Printable Characters — `grep '[[:print:]]'`

**Syntax:**
```bash
grep '[[:print:]]' file
```
Matches lines with any printable character (letters, digits, punctuation, **and space**).

### 12.11 Blank (Space/Tab) — `grep '[[:blank:]]'`

**Syntax:**
```bash
grep '[[:blank:]]' file
```
Matches lines containing a space or a tab character.

### 12.12 Whitespace — `grep '[[:space:]]'`

**Syntax:**
```bash
grep '[[:space:]]' file
```
Matches lines with **any** whitespace character (space, tab, newline, vertical tab, form feed, carriage return).

### 12.13 Non-Space (Visible) Characters — `grep '[[:graph:]]'`

**Syntax:**
```bash
grep '[[:graph:]]' file
```
Matches lines containing any **visible, non-space** character (i.e., printable characters excluding the space itself).

### Quick Comparison of Related Classes

| Class | Includes Space? | Includes Letters/Digits? | Includes Punctuation? |
|---|---|---|---|
| `[[:print:]]` | ✅ Yes | ✅ Yes | ✅ Yes |
| `[[:graph:]]` | ❌ No | ✅ Yes | ✅ Yes |
| `[[:blank:]]` | ✅ Yes (space & tab only) | ❌ No | ❌ No |
| `[[:space:]]` | ✅ Yes (all whitespace) | ❌ No | ❌ No |

### 12.14 Skip All Empty Lines

**Concept:** To filter out blank lines from output, either invert-match a blank-line pattern or require at least one character.

**Syntax:**
```bash
grep -v '^$' file      # explicitly excludes lines with nothing between start and end
# OR
grep '.' file           # requires at least one character on the line
```

**Example:**
```bash
grep -v '^$' notes.txt
```
Removes every completely empty line from the output.

### 12.15 Matching a 12-Digit Number — `egrep '[[:digit:]]{12}' file`

**Concept:** Combines a character class with an ERE interval to match exactly 12 consecutive digits (e.g., validating an ID number format).

**Syntax:**
```bash
egrep '[[:digit:]]{12}' file
```

**Example:**
```bash
egrep '[[:digit:]]{12}' ids.txt
```
Matches lines containing a 12-digit numeric sequence, e.g., `123456789012`.

### 12.16 Matching a 6-Digit Number with Word Boundaries — `egrep '\b[[:digit:]]{6}\b' file`

**Concept:** Word boundaries (`\b`) ensure the 6-digit number is **standalone** (not part of a longer number).

**Syntax:**
```bash
egrep '\b[[:digit:]]{6}\b' file
```

**Example:**
```bash
egrep '\b[[:digit:]]{6}\b' addresses.txt
```
Matches standalone 6-digit numbers such as postal/PIN codes (e.g., `221005`), but **not** the `6` digits embedded inside `1234567` (which is 7 digits, so boundaries fail).

### 12.17 Matching Roll Numbers — `egrep '\b[[:alpha:]]{2}[[:digit:]]{2}[[:alpha:]][[:digit:]]{2}\b' file`

**Concept:** A composite pattern that models a structured ID format: 2 letters + 2 digits + 1 letter + 2 digits, bounded by word boundaries so it matches the ID exactly (not as a substring of something longer).

**Syntax:**
```bash
egrep '\b[[:alpha:]]{2}[[:digit:]]{2}[[:alpha:]][[:digit:]]{2}\b' file
```

**Example:**
```bash
egrep '\b[[:alpha:]]{2}[[:digit:]]{2}[[:alpha:]][[:digit:]]{2}\b' students.txt
```
Matches roll numbers like `CS20B15` (`CS` + `20` + `B` + `15`).

**Breakdown Table:**

| Segment | Pattern | Meaning |
|---|---|---|
| 1 | `\b` | Start at a word boundary |
| 2 | `[[:alpha:]]{2}` | Exactly 2 letters (e.g., department code) |
| 3 | `[[:digit:]]{2}` | Exactly 2 digits (e.g., year) |
| 4 | `[[:alpha:]]` | Exactly 1 letter (e.g., section) |
| 5 | `[[:digit:]]{2}` | Exactly 2 digits (e.g., roll sequence) |
| 6 | `\b` | End at a word boundary |

### 12.18 Matching URLs

**Concept:** URLs typically follow the pattern `scheme://domain/path`. A basic regex to identify URL-like text uses grouping and character classes.

**Example (representative pattern):**
```bash
egrep 'https?://[[:alnum:]./_-]+' file.txt
```

| Segment | Meaning |
|---|---|
| `https?` | Matches `http` or `https` (`s?` = optional `s`) |
| `://` | Literal separator |
| `[[:alnum:]./_-]+` | One or more letters, digits, dots, slashes, underscores, or hyphens (domain + path) |

**Example:**
```bash
egrep 'https?://[[:alnum:]./_-]+' links.txt
```
Matches lines containing URLs such as `https://example.com/page` or `http://site.org`.

---

## 13. The `cut` Command — Syntax and Examples

`cut` is a companion text-processing tool often paired with `grep` in the same pipeline — it extracts specific **columns/fields** from each line, either by character position or by delimiter.

### 13.1 Concept & General Syntax

```bash
cut -c <range> file        # cut by character position
cut -d "<delimiter>" -f <field-number> file   # cut by delimiter and field
```

| Flag | Meaning |
|---|---|
| `-c` | Select specific **character positions** |
| `-d` | Specify the **delimiter** character (default is tab) |
| `-f` | Select specific **field number(s)** based on the delimiter |

### 13.2 Cut by Character Range — `cut -c 1-4 file`

**Syntax:**
```bash
cut -c 1-4 file
```
Extracts characters **1 through 4** from every line.

**Example:**
```bash
cut -c 1-4 codes.txt
```
Input: `ABCDEFGH` → Output: `ABCD`

### 13.3 Cut from Start — `cut -c -4 file`

**Syntax:**
```bash
cut -c -4 file
```
Equivalent to `cut -c 1-4` — extracts characters from the **beginning of the line up to position 4**.

**Example:**
```bash
cut -c -4 codes.txt
```
Input: `ABCDEFGH` → Output: `ABCD`

### 13.4 Cut by Space Delimiter — `cat file | cut -d " " -f 1`

**Syntax:**
```bash
cat file | cut -d " " -f 1
```
Splits each line by the **space** character and prints only **field 1** (the first "word").

**Example:**
```bash
cat contacts.txt | cut -d " " -f 1
```
Input: `John Smith 25` → Output: `John`

### 13.5 Chained `cut` — `cat file | cut -d ";" -f 2 | cut -d "," -f 1`

**Syntax:**
```bash
cat file | cut -d ";" -f 2 | cut -d "," -f 1
```
> **Typo fix:** The original notes show `cut "," -f 1` — the delimiter flag `-d` was missing before `","`. The corrected, working syntax is `cut -d "," -f 1`.

**Concept:** This chains two `cut` operations:
1. First `cut -d ";" -f 2` — splits the line by `;` and keeps the 2nd field.
2. Second `cut -d "," -f 1` — takes that result, splits it by `,`, and keeps the 1st field.

**Example:**
```bash
echo "id;name,age;email" | cut -d ";" -f 2 | cut -d "," -f 1
```
Step 1 output: `name,age`
Step 2 output: `name`

### 13.6 Achieving the Same Result with `grep`

**Concept:** The same field-extraction task from 12.5 can often be reproduced using `grep -oP` (Perl-compatible regex) with a capturing group, instead of chaining `cut`.

**Example:**
```bash
echo "id;name,age;email" | grep -oP '(?<=;)[^,;]+(?=,)'
```
Extracts the same substring (`name`) directly using a **lookbehind** and **lookahead**, demonstrating that `grep` (with `-P`) can replicate what a `cut` pipeline does, but using regex.

> **Exam Tip:** `-o` makes `grep` print **only the matched portion** of a line (not the whole line) — extremely useful for field-extraction–style tasks.

### 13.7 Full Extraction Pipeline — `cut -d "/" -f 3 | cut -d " " -f 1 | head -n 19 | tail -n 1`

**Syntax:**
```bash
cat file | cut -d "/" -f 3 | cut -d " " -f 1 | head -n 19 | tail -n 1
```

**Concept — step by step:**

| Step | Command | Purpose |
|---|---|---|
| 1 | `cat file` | Stream file contents |
| 2 | `cut -d "/" -f 3` | Split each line by `/` and keep the 3rd field |
| 3 | `cut -d " " -f 1` | From that result, split by space and keep the 1st word |
| 4 | `head -n 19` | Keep only the first 19 resulting lines |
| 5 | `tail -n 1` | From those 19 lines, keep only the **last one** (i.e., isolate line 19) |

**Example:**
```bash
cat access.log | cut -d "/" -f 3 | cut -d " " -f 1 | head -n 19 | tail -n 1
```
This pipeline is a classic technique to **isolate one specific field from one specific line number** — here, it extracts the value from the 3rd `/`-separated field, then the 1st space-separated word of that, on exactly line 19.

---

## 14. Summary

### Core Takeaways

0. **`grep`** (Global Regular Expression Print) is the tool that *executes* regex searches from the command line; it supports three engines (BRE default, ERE via `-E`, fixed-string via `-F`) and a rich flag set (`-i`, `-v`, `-c`, `-n`, `-o`, `-w`, `-x`, `-r`, `-A/-B/-C`, `-q`, `-m`, `-P`, etc.) that control matching behavior, output format, and scripting integration.
1. **Regex** is a POSIX-standardized pattern-matching language with two flavors: **BRE** (default in `grep`) and **ERE** (used via `egrep` or `grep -E`).
2. **Common metacharacters** (`. * [ ] ^ $ \`) work identically in both BRE and ERE.
3. **BRE-only** syntax needs escaping for grouping/intervals: `\( \)`, `\{n,m\}`.
4. **ERE-only** syntax adds unescaped `( )`, `{ }`, plus new operators: `+`, `?`, `|`.
5. **POSIX character classes** (`[[:alpha:]]`, `[[:digit:]]`, etc.) provide portable, locale-safe ways to match categories of characters, and can be combined or negated.
6. **Backreferences** (`\1`–`\9`) allow a regex to require previously matched text to reoccur — powerful for detecting duplicates.
7. **Operator precedence** determines evaluation order; in both BRE and ERE, character collation and bracket expressions bind tightest, while anchors (`^ $`) and alternation (`|`, ERE only) bind loosest.
8. **`grep` practical patterns** demonstrated: anchors (`^`, `$`), wildcards (`.`, `.*`), quantifiers (`*`, `+`, `?`, `{n,m}`), grouping, alternation (`|`), and backreferences.
9. **Character classes in practice** are used to validate structured data — phone numbers, PIN codes, roll numbers, URLs — using fixed-length digit/letter patterns combined with word boundaries (`\b`).
10. **`cut`** complements `grep` by extracting specific characters or delimiter-separated fields from matched lines, and can be chained or combined with `head`/`tail` to isolate exact fields and line numbers. Many `cut`-based field extractions can alternatively be done with `grep -oP` using lookaheads/lookbehinds.

### Quick Reference Cheat-Sheet

| Task | Command Pattern |
|---|---|
| Basic search | `grep 'pattern' file` |
| Case: line starts with X | `grep '^X' file` |
| Case: line ends with X | `grep 'X$' file` |
| Literal dot | `grep '\.' file` |
| Any char, 0+ times | `grep 'a.*b' file` |
| BRE repetition | `grep 'x\{2,4\}' file` |
| ERE repetition | `grep -E 'x{2,4}' file` |
| ERE one-or-more | `grep -E 'x+' file` |
| ERE zero-or-one | `grep -E 'x?' file` |
| ERE OR | `grep -E '(a\|b)' file` |
| Invert match | `grep -v 'pattern' file` |
| Only matched text | `grep -o 'pattern' file` |
| Remove blank lines | `grep -v '^$' file` |
| Digit class | `grep '[[:digit:]]' file` |
| Backreference (duplicate check) | `grep '\(x\).*\1' file` |
| Cut by character | `cut -c 1-4 file` |
| Cut by delimiter/field | `cut -d "," -f 2 file` |

---

*End of Week 4 (Lecture 1 & 2) Notes — Pattern Matching: Regex & grep.*
