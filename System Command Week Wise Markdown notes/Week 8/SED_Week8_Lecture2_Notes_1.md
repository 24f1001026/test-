# SED — Stream Editor
Comprehensive Exam Preparation Notes | Week 8 – Lecture 2

---

## Table of Contents

- [1. Introduction](#1-introduction)
- [2. Execution Model](#2-execution-model)
- [3. Invoking SED — Usage Modes](#3-invoking-sed--usage-modes)
  - [3.1 Command Line Mode (`-e`)](#31-command-line-mode--e)
  - [3.2 Script Mode (`-f`)](#32-script-mode--f)
  - [3.3 Standalone Executable Script](#33-standalone-executable-script)
- [4. Important Command-Line Options](#4-important-command-line-options)
  - [4.1 The `-n` Option (Suppress Auto-Print)](#41-the--n-option-suppress-auto-print)
  - [4.2 The `-E` / `-r` Option (Extended Regex)](#42-the--e---r-option-extended-regex)
  - [4.3 The `-i` Option (In-place Editing)](#43-the--i-option-in-place-editing)
- [5. SED Statement Structure](#5-sed-statement-structure)
- [6. Grouping Commands](#6-grouping-commands)
  - [6.1 Combining Multiple Commands](#61-combining-multiple-commands)
- [7. Addressing in SED](#7-addressing-in-sed)
  - [7.1 Selecting by Line Numbers](#71-selecting-by-line-numbers)
  - [7.2 Printing a Particular Line](#72-printing-a-particular-line)
  - [7.3 Printing Every Nth Line (Step Addressing)](#73-printing-every-nth-line-step-addressing)
  - [7.4 Selecting by Regex (Text Matching)](#74-selecting-by-regex-text-matching)
  - [7.5 Range Addresses](#75-range-addresses)
  - [7.6 Negation with `!`](#76-negation-with-)
- [8. Basic Actions (Commands)](#8-basic-actions-commands)
  - [8.1 `p` — Print](#81-p--print)
  - [8.2 `d` — Delete](#82-d--delete)
  - [8.3 `s` — Substitute (Search and Replace)](#83-s--substitute-search-and-replace)
  - [8.4 `=` — Print Line Number](#84----print-line-number)
  - [8.5 `#` — Comment](#85--comment)
  - [8.6 `i` — Insert](#86-i--insert)
  - [8.7 `a` — Append](#87-a--append)
  - [8.8 `c` — Change](#88-c--change)
  - [8.9 Inserting Headers and Footers](#89-inserting-headers-and-footers)
- [9. Programming Commands (Advanced)](#9-programming-commands-advanced)
  - [9.1 `b label` — Unconditional Branch](#91-b-label--unconditional-branch)
  - [9.2 `:label` — Label Definition](#92-label--label-definition)
  - [9.3 `N` — Join / Read Next Line](#93-n--join--read-next-line)
  - [9.4 `q` — Quit](#94-q--quit)
  - [9.5 `t` / `T` — Conditional Branch](#95-t--t--conditional-branch)
  - [9.6 `w filename` — Write to File](#96-w-filename--write-to-file)
  - [9.7 `x`, `h`, `H`, `g`, `G` — Hold Space Family](#97-x-h-h-g-g--hold-space-family)
- [10. SED with Bash](#10-sed-with-bash)
- [11. Debugging SED Scripts](#11-debugging-sed-scripts)
- [12. Why SED?](#12-why-sed)
- [13. Quick Reference — All Demo Commands (Mapped by Topic)](#13-quick-reference--all-demo-commands-mapped-by-topic)
- [14. Summary](#14-summary)

---

## 1. Introduction

- `sed` is a **programming language** designed specifically for processing text streams line-by-line.
- **SED** is an abbreviation for **S**tream **ED**itor.
- It is a part of the **POSIX** standard — meaning it ships by default with virtually every Unix/Linux distribution and macOS.
- `sed` **precedes `awk`** historically; both tools belong to the classic Unix "text-processing toolkit," but `sed` is simpler and is optimized purely for **line-based transformations**, whereas `awk` is closer to a full scripting/reporting language.
- `sed` is **non-interactive** — unlike `vim` or `nano`, you cannot "see" and manually navigate the file; instead, you describe *what changes to make* using a script, and `sed` applies them automatically across the entire stream.

> **Exam Tip:** If asked to differentiate `sed` from a text editor like `vi`, remember: `sed` is a **stream editor** (batch/non-interactive, scriptable) while `vi`/`vim` is an **interactive editor**.

---

## 2. Execution Model

This is the **conceptual core** of `sed` and is frequently tested in exams.

- The **input stream** is treated as a set of **lines**.
- Each **line** is simply a sequence of characters terminated by a newline.
- `sed` maintains **two data buffers**:
  1. **Pattern Space** — the *active* working buffer that holds the line currently being processed.
  2. **Hold Space** — an *auxiliary* buffer used for temporary storage across cycles.
- For **each line** of input, `sed` performs an **execution cycle**:
  1. The line is **read** and **loaded into the pattern space**.
  2. **All statements** in the script are executed **in sequence**.
  3. Each statement checks its **address pattern** — if the current line matches, the corresponding **action** is executed.
  4. At the end of the cycle (unless suppressed with `-n`), the pattern space is **auto-printed** to standard output.
  5. The cycle repeats for the next line, until the input is exhausted.

### 🔎 Expanded Explanation: Pattern Space vs Hold Space

| Buffer | Purpose | Default Content | Lifetime | Commands That Use It |
|---|---|---|---|---|
| **Pattern Space** | Active working area where the current line is processed and edited | Current input line | Refreshed every cycle (unless modified by `N`) | `s`, `p`, `d`, `i`, `a`, `c`, `N` |
| **Hold Space** | Temporary/auxiliary storage not shown in output by default | Empty at start | Persists across cycles until explicitly changed | `h`, `H`, `g`, `G`, `x` |

### Visual Flow of One Execution Cycle

```
 ┌────────────────────┐
 │  Read next line     │
 └─────────┬───────────┘
           ▼
 ┌────────────────────┐
 │  Load into Pattern  │
 │  Space               │
 └─────────┬───────────┘
           ▼
 ┌────────────────────┐
 │ Execute script       │
 │ statements in order  │
 │ (address → action)   │
 └─────────┬───────────┘
           ▼
 ┌────────────────────┐
 │ Auto-print pattern   │
 │ space (unless -n)    │
 └─────────┬───────────┘
           ▼
      Next line / EOF
```

---

## 3. Invoking SED — Usage Modes

### 3.1 Command Line Mode (`-e`)

The `-e` flag tells `sed` that the following argument is a **script/expression** to execute.

**Concept:** Best for short, one-off transformations directly from the terminal.

**Syntax:**
```bash
sed -e 'script' file
```

**Demo Example :**
```bash
sed -e "" file
```
**Explanation:** An **empty script** (`""`) applies no transformation at all. Because `sed` **auto-prints** the pattern space after every cycle by default, this command simply **prints the file unchanged** — functionally similar to `cat file`. This is often used to demonstrate the *default auto-print behavior* before introducing `-n`.

**More Practical Example:**
```bash
sed -e 's/hello/world/g' input.txt
```
Replaces every occurrence of `hello` with `world` on every line of `input.txt`.

> **Note:** When only **one** script is given, `-e` is optional:
> ```bash
> sed 's/hello/world/g' input.txt
> ```
> is equivalent to the above.

### 3.2 Script Mode (`-f`)

The `-f` flag tells `sed` to read its script from an **external file** rather than the command line.

**Concept:** Best for longer, reusable, or complex scripts.

**Syntax:**
```bash
sed -f ./myscript.sed input.txt
```

**Example — `myscript.sed`:**
```sed
2,8s/hello/world/g
```
This applies the substitution only to **lines 2 through 8**.

**Demo Example :**
```bash
sed -f script.sed input.txt
```
**Explanation:** Instead of typing the transformation inline, all `sed` commands are placed inside `script.sed` (one command per line, or separated by `;`), which keeps complex, multi-step transformations **organized and reusable** — especially useful when a script contains many address/action pairs, labels, and branches.

### 3.3 Standalone Executable Script

A `sed` script file can also be made **directly executable** using a **shebang** line:

```bash
#!/usr/bin/sed -f
2,8s/hello/world/g
```

**Steps to use:**
```bash
chmod +x myscript.sed
./myscript.sed input.txt
```

**Explanation:** The shebang `#!/usr/bin/sed -f` tells the shell to invoke `sed -f` on this very file, treating everything below the shebang line as the script body. This effectively turns the `.sed` file into a self-contained mini-program.

---

## 4. Important Command-Line Options

### 4.1 The `-n` Option (Suppress Auto-Print)

**Concept:** By default, `sed` **automatically prints** the pattern space at the end of every cycle — *in addition to* any explicit `p` commands. The `-n` (no-autoprint) option **disables this default printing**, so **only** lines explicitly printed using `p` (or similar) appear in the output.

**Syntax:**
```bash
sed -n 'script' file
```

**Demo Example (`sed -n -e "" file`):**
```bash
sed -n -e "" file
```
**Explanation:** Here the script is still **empty**, but because `-n` suppresses auto-printing and there is **no `p` command**, this produces **no output at all**.

**Demo Example (Importance of `-n` option):**
```bash
sed -n '3p' file
```
**Explanation:** Without `-n`, this command would print **line 3 twice** — once from the explicit `p` command, and once again from the default auto-print at the end of the cycle. **With** `-n`, line 3 is printed **only once**, because auto-print is suppressed. This is the single most common `sed` idiom for **selectively printing lines**.

| Without `-n` | With `-n` |
|---|---|
| `sed '3p' file` → line 3 appears **twice**, all other lines appear **once** | `sed -n '3p' file` → **only** line 3 appears, **once** |

> **Exam Tip:** `-n` + `p` is the standard idiom to replicate `grep`/`head`/`sed`-only line-selection behavior. Forgetting `-n` is one of the most common `sed` mistakes.

### 4.2 The `-E` / `-r` Option (Extended Regex)

**Concept:** By default, `sed` uses **Basic Regular Expressions (BRE)**, where metacharacters like `+`, `?`, `|`, and `()` must be **escaped** (`\+`, `\?`, `\|`, `\(\)`) to have special meaning. The `-E` (or `-r` on GNU `sed`) option switches to **Extended Regular Expressions (ERE)**, where these metacharacters work **without escaping** — similar to `egrep`/`awk` regex syntax.

**Syntax:**
```bash
sed -E 'script' file
```

**Demo Example ("extended regex"):**
```bash
sed -E 's/[0-9]+/NUM/g' file
```
**Explanation:** In **BRE** (default), the same command would require escaping the `+` as `\+`:
```bash
sed 's/[0-9]\+/NUM/g' file
```
With `-E`, `+` (one or more), `?` (zero or one), `|` (alternation), and `()` (grouping) are used **directly** without backslashes, making complex patterns significantly more readable.

| Regex Type | Flag | `+` meaning | Example |
|---|---|---|---|
| **BRE** (Basic) | *(default)* | Literal `+` | `s/a\+/X/` → one-or-more `a`'s |
| **ERE** (Extended) | `-E` / `-r` | One-or-more | `s/a+/X/` → one-or-more `a`'s |

### 4.3 The `-i` Option (In-place Editing)

**Concept:** By default, `sed` sends output to **standard output** (the terminal), leaving the original file **untouched**. The `-i` option edits the file **in place**, overwriting it directly.

**Syntax:**
```bash
sed -i 's/foo/bar/g' file          # Edits file directly (GNU sed)
sed -i.bak 's/foo/bar/g' file      # Edits file, but keeps a backup as file.bak
```

> **Caution:** Always test a `sed` command **without** `-i` first (viewing output on screen) before applying it destructively with `-i`, especially in exams/labs where original files should be preserved.

---

## 5. SED Statement Structure

Every `sed` statement follows the general structure:

```
address pattern action
```

| Component | Description |
|---|---|
| **address** | Specifies *which* line(s) the action applies to (optional — if omitted, the action applies to **every** line) |
| **pattern** | The matching criteria (regex or line number) used within the action |
| **action** | The single-character command to execute (same convention as the `ed`/`ex` line editors) |

### Key Symbols Used in Statements

| Symbol | Meaning |
|---|---|
| `;` | Separates multiple commands/options on one line |
| `,` | Defines an address **range** (start,end) |
| `!` | **Negation** — inverts the address match; meaning depends on the action used |
| `:label` | Marks a location for branch commands (`b`, `t`, `T`) to jump to |

> **Note:** The single-character action convention (`p`, `d`, `s`, `i`, `a`, `c`, etc.) is directly inherited from the classic Unix line editors **`ed`** and **`ex`**, which predate `sed`.

---

## 6. Grouping Commands

Multiple commands can be **grouped** together and applied to the **same address** using curly braces `{ }`, with individual commands separated by semicolons `;`.

**Syntax:**
```bash
address { cmd; cmd; }
```

### 6.1 Combining Multiple Commands

**Demo Example ("combine commands"):**
```bash
sed -n '2,4{s/foo/bar/; p}' file
```
**Explanation:**
1. `2,4` — the address range restricts the action to lines 2 through 4.
2. `s/foo/bar/` — substitutes the first occurrence of `foo` with `bar` on those lines.
3. `p` — explicitly prints the (possibly modified) pattern space.
4. `-n` — ensures each line is printed **only once** (via the explicit `p`), not twice.

**Additional Example:**
```bash
sed '/START/,/END/{ s/foo/bar/g; s/baz/qux/g }' file
```
Within the block delimited by lines matching `START` and `END`, performs **two separate substitutions** on every line.

> **Exam Tip:** Grouping with `{}` is essential whenever **more than one action** needs to share the **same address/condition** — it avoids repeating the address for every command.

---

## 7. Addressing in SED

Addressing determines **which line(s)** a `sed` command acts upon. This is one of the most heavily tested areas.

### 7.1 Selecting by Line Numbers

| Address | Meaning |
|---|---|
| `5` | Refers to **line 5** only |
| `$` | Refers to the **last line** of input |
| `1~3` | Refers to **every 3rd line starting from line 1** (i.e., lines 1, 4, 7, 10 …) — a GNU `sed` extension |
| `%` | Context-dependent step-based addressing extension (GNU `sed`) |

### 7.2 Printing a Particular Line

**Demo Example (print a particular line):**
```bash
sed -n '3p' file
```
**Explanation:** Combines the **line-number address** (`3`) with the **print action** (`p`), and `-n` to avoid duplicate output. This prints **only line 3**.

**Related — Print the last line:**
```bash
sed -n '$p' file
```
Prints only the **last line** of the file, using the special `$` address.

### 7.3 Printing Every Nth Line (Step Addressing)

**Demo Example (print every nth line):**
```bash
sed -n '1~2p' file
```
**Explanation:** The GNU extension `first~step` addresses every `step`-th line starting at `first`. Here, `1~2` selects **lines 1, 3, 5, 7 …** (every odd line). Combined with `-n` and `p`, only these lines are printed.

**More Examples:**
```bash
sed -n '0~3p' file     # Every 3rd line starting from line 3 (3, 6, 9, ...)
sed -n '2~5p' file     # Lines 2, 7, 12, 17, ...
```

### 7.4 Selecting by Regex (Text Matching)

| Address | Meaning |
|---|---|
| `/regexp/` | Matches any line containing text that satisfies the regular expression |

**Demo Example (regex address):**
```bash
sed -n '/error/p' log.txt
```
**Explanation:** Prints only the lines that **contain the word `error`** — functionally similar to `grep 'error' log.txt`, but expressed using `sed`'s addressing mechanism.

### 7.5 Range Addresses

| Address | Meaning |
|---|---|
| `/regexp/, +4` | From the matched line, include the **next 4 lines** as well |
| `/regexp1/,/regexp2/` | From the line matching `regexp1` **up to** the line matching `regexp2` |
| `5,/regexp/` | From **line 5** up to the line matching `regexp` |
| `5,15` | From **line 5 to line 15** |
| `/regexp/, ~2` | From the matched line **up to the next line whose number is a multiple of 2** |

**Demo Example (`address range`):**
```bash
sed -n '5,10p' file
```
Prints lines **5 through 10**.

**Demo Example (`/regex/,+n`):**
```bash
sed -n '/START/,+3p' file
```
**Explanation:** Starts printing from the line matching `START`, and continues for the **next 3 lines** after it (i.e., 4 lines total are printed: the match plus 3 more).

**Demo Example (`range-end as regex`):**
```bash
sed -n '5,/END/p' file
```
**Explanation:** Starts printing from **line 5** and continues until it encounters a line matching the regex `END` (inclusive). Useful when you know the starting line number but only know the ending point by content, not line number.

**Demo Example (`regex to regex`):**
```bash
sed -n '/START/,/END/p' file
```
**Explanation:** Prints every line from the first line matching `START` through the first subsequent line matching `END` (inclusive). If `START` appears multiple times, `sed` will restart the range logic after each completed range in a single pass.

### 🔎 Expanded Notes on Range Addressing

- Range addresses are extremely important in exams for "select a block of lines" style questions.
- `,` always defines a **start and end boundary**.
- Ranges can freely mix **line numbers** and **regex patterns**, offering flexibility unmatched by simple line-based tools like `head`/`tail`.

**Combined Example:**
```bash
sed -n '/START/,/END/d' file.txt
```
Deletes all lines from the line containing `START` to the line containing `END` (inclusive) — but note `-n` here is unnecessary for `d` since `d` already prevents auto-print of *deleted* lines; it is shown for consistency with printing examples.

### 7.6 Negation with `!`

**Concept:** Appending `!` to an address **inverts** the match — the action applies to all lines **that do NOT match** the given address.

**Demo Example (15:38 — `'p' vs '!p' vs '$p'`):**
```bash
sed -n '3p'     file   # Prints ONLY line 3
sed -n '3!p'    file   # Prints EVERY line EXCEPT line 3
sed -n '$p'     file   # Prints ONLY the last line
```

| Command | Meaning |
|---|---|
| `sed -n '3p' file` | Print **only** line 3 |
| `sed -n '3!p' file` | Print **all lines except** line 3 (negation) |
| `sed -n '$p' file` | Print **only** the last line (`$` = last line address) |

> **Exam Tip:** `!` can be combined with **any** address type — line numbers, ranges, or regex — e.g., `/error/!p` prints all lines that **do NOT** contain "error".




### 7.7 Summary Table of Address Types


| Category | Syntax | Description |
|---|---|---|
| Exact line | `5` | Single specific line |
| Last line | `$` | Last line of the file/stream |
| Step (from start) | `1~3` | Every Nth line starting from a given line |
| Regex match | `/regexp/` | Any line matching a pattern |
| Numeric range | `5,15` | Between two fixed line numbers |
| Regex + offset | `/regexp/,+4` | Matched line plus N following lines |
| Regex-to-regex | `/regexp1/,/regexp2/` | Between two pattern matches |
| Line-to-regex | `5,/regexp/` | From a line number until a pattern match |
| Regex-to-step-end | `/regexp/,~2` | From a match until next line number multiple of N |
| Negation | `!` (added to any address) | Inverts the selection (acts on non-matching 
lines) |


> **Exam Tip:** Range addressing (`/regexp1/,/regexp2/`) is a **favorite exam 
question** — remember it is **inclusive** of both boundary lines.


---

## 8. Basic Actions (Commands)

These are the fundamental **single-character actions** applied to the pattern space once an address matches.

| Action | Description |
|---|---|
| `p` | Print the pattern space |
| `d` | Delete the pattern space (skip auto-print, move to next line) |
| `s` | Substitute using regex match: `s/pattern/replacement/g` |
| `=` | Print the current input line number, followed by a newline |
| `#` | Comment (ignored by `sed`) |
| `i` | Insert text **above** the current line |
| `a` | Append text **below** the current line |
| `c` | Change (replace) the current line entirely |

### 8.1 `p` — Print

**Syntax:** `[address]p`

Already covered extensively in [Section 7](#7-addressing-in-sed) — always pair with `-n` to avoid duplicate output unless duplication is intentional.

**Example:**
```bash
sed -n '/warning/p' file
```
Prints only lines containing "warning".

### 8.2 `d` — Delete

**Syntax:** `[address]d`

**Concept:** Deletes the pattern space for matching lines — meaning those lines are **excluded** from the output entirely (auto-print is skipped for them; no `-n` is needed).

**Demo Example (20:01 — "delete a particular line"):**
```bash
sed '3d' file
```
**Explanation:** Deletes **line 3** only; all other lines are printed normally via auto-print.

**Demo Example (21:17 — "delete a range of lines"):**
```bash
sed '5,10d' file
```
**Explanation:** Deletes **lines 5 through 10** (inclusive); all other lines pass through unchanged.

**Demo Example (21:33 — `/regex/d`):**
```bash
sed '/DEBUG/d' file
```
**Explanation:** Deletes **every line** that contains the text `DEBUG` — commonly used to strip out log noise, comments, or blank lines:
```bash
sed '/^$/d' file      # Deletes all blank lines
sed '/^#/d' file       # Deletes all comment lines starting with #
```

### 8.3 `s` — Substitute (Search and Replace)

**Syntax:**
```
[address]s/pattern/replacement/[flags]
```

**Common Flags:**

| Flag | Meaning |
|---|---|
| `g` | Global — replace **all** occurrences in the line, not just the first |
| `p` | Print the line if a substitution was made (often used with `-n`) |
| `i` / `I` | Case-insensitive matching |
| `N` (number) | Replace only the Nth occurrence in the line |
| `w file` | Write result to `file` if a substitution occurred |

**Demo Example (21:49 — "search and replace"):**
```bash
sed 's/foo/bar/' file        # Replaces FIRST occurrence of foo per line
sed 's/foo/bar/g' file       # Replaces ALL occurrences of foo per line
sed 's/foo/bar/2' file       # Replaces only the SECOND occurrence per line
sed 's/foo/bar/gi' file      # Case-insensitive, global replace
```

**Explanation:** This is the **most frequently used** `sed` action. The pattern between the first pair of `/` is a regular expression matched against the pattern space; the replacement between the second pair of `/` is substituted in. Flags after the third `/` fine-tune the behavior (how many matches, case sensitivity, printing, etc.).

**Additional Practical Examples:**
```bash
sed 's/[0-9]\+/#/g' file          # Replace all number sequences with '#' (BRE, escaped +)
sed 's/^/> /' file                # Prefix every line with "> "
sed 's/$/ END/' file              # Suffix every line with " END"
sed 's#/usr/local#/opt#g' file    # Using # as delimiter instead of / (useful with paths)
```

> **Exam Tip:** The delimiter `/` in `s/pattern/replacement/` can be **replaced by any character** (e.g., `#`, `|`, `,`) — very useful when the pattern itself contains `/` (like file paths), to avoid excessive escaping.

### 8.4 `=` — Print Line Number

**Syntax:** `[address]=`

**Demo Example (12:48 — `sed -e "=" file`):**
```bash
sed -e '=' file
```
**Explanation:** Prints the **current line number**, followed by a newline, **before** printing the actual line content (via normal auto-print). The output alternates between a number and the corresponding line of text:
```
1
This is line one
2
This is line two
```

**Combined with `-n` and `p` for a "line number: content" style:**
```bash
sed -n '=;p' file
```
Or, using `cat -n` for a cleaner alternative:
```bash
cat -n file
```

### 8.5 `#` — Comment

**Syntax:** `#comment text`

**Concept:** Any line in a `sed` script beginning with `#` is treated as a comment and ignored during execution — useful for documenting complex scripts stored with `-f`.

**Example (inside a script file):**
```sed
# This script removes all blank lines
/^$/d
```

> **Special case:** `#n` as the **very first line** of a script is equivalent to specifying `-n` on the command line (suppresses auto-print).

### 8.6 `i` — Insert

**Syntax:** `[address]i\text` (or `[address]i text` in GNU `sed`)

**Concept:** Inserts the given text **immediately before** (above) the matched line.

**Demo Example (33:49 / 34:16 — "insert or append at any line" / "@ a regex address"):**
```bash
sed '3i\This line is inserted above line 3' file
```
GNU `sed` also allows the simpler one-line form:
```bash
sed '3i This line is inserted above line 3' file
```

**Inserting at a regex address:**
```bash
sed '/BEGIN/i\--- Inserted before BEGIN ---' file
```
Inserts the given text immediately **before** any line matching `BEGIN`.

### 8.7 `a` — Append

**Syntax:** `[address]a\text` (or `[address]a text` in GNU `sed`)

**Concept:** Appends (inserts) the given text **immediately after** (below) the matched line.

**Demo Example (33:49 / 34:16):**
```bash
sed '3a\This line is appended after line 3' file
sed '/END/a\--- Appended after END ---' file
```
**Explanation:** The first command appends text after **line 3**; the second appends text after **every line matching the regex `END`** — demonstrating that `i`/`a` work with **both numeric and regex addresses**.

### 8.8 `c` — Change

**Syntax:** `[address]c\text` (or `[address]c text` in GNU `sed`)

**Concept:** **Replaces** the entire matched line (or range of lines) with the given text.

**Demo Example (36:18 — "change a line"):**
```bash
sed '3c\This line replaces line 3 entirely' file
```
**Explanation:** Line 3's original content is discarded and replaced with the new text.

**Changing a range:**
```bash
sed '5,7c\This single line replaces lines 5-7' file
```
**Explanation:** When `c` is applied to a **range**, the **entire range** is replaced by the given text **only once** (not once per line) — this is a subtle but important exam point.

> **Exam Tip — `i` vs `a` vs `c`:**
> | Action | Position | Original line kept? |
> |---|---|---|
> | `i` (Insert) | **Before** matched line | ✅ Yes |
> | `a` (Append) | **After** matched line | ✅ Yes |
> | `c` (Change) | **Replaces** matched line(s) | ❌ No |

### 8.9 Inserting Headers and Footers

**Demo Example (32:02 — "insert header and footer"):**
```bash
sed '1i\==== HEADER ====' file          # Insert a header before line 1
sed '$a\==== FOOTER ====' file          # Append a footer after the last line
```
**Explanation:** Combining `i` with address `1` (first line) adds a **header**, while combining `a` with address `$` (last line) adds a **footer**. Both can be combined in a single command:
```bash
sed '1i\==== HEADER ====
$a\==== FOOTER ====' file
```
This is a very common real-world use case — e.g., wrapping report output with title/footer lines, or wrapping HTML/XML fragments with boilerplate tags.

---

## 9. Programming Commands (Advanced)

These commands give `sed` its "programming language" capability — supporting branching, looping, multi-line handling, and buffer manipulation. This is the most conceptually advanced part of the syllabus.

| Command | Description |
|---|---|
| `b label` | Branch **unconditionally** to the specified label |
| `:label` | Specifies the **location** of a label for a branch command |
| `N` | Adds a new line to the pattern space by appending the **next line of input** |
| `q` | **Exit** `sed` without processing any more commands or input lines |
| `t label` | Branch to `label` **only if** a successful substitution was made since the last input line was read or branch taken |
| `T label` | Branch to `label` **only if** **no** successful substitution was made |
| `w filename` | Write the pattern space to the specified `filename` |
| `x` | **Exchange** the contents of the hold space and pattern space |

### 9.1 `b label` — Unconditional Branch

**Syntax:** `b [label]`

**Concept:** Jumps script execution to `:label`, skipping any commands in between. If no label is given, `b` jumps to the **end of the script** (i.e., skips remaining commands for the current cycle).

**Example — Looping substitution:**
```bash
sed ':a; s/aa/a/; ba' file
```
**Explanation:** This repeatedly replaces `aa` with `a` — but note this specific example **loops forever** unless paired with a **conditional** branch (`t`), because `b` is unconditional. This is why `t`/`T` (conditional branches) are typically preferred for loops that must terminate.

### 9.2 `:label` — Label Definition

**Syntax:** `:label`

**Concept:** Marks a named position in the script. Used as the **target** for `b`, `t`, and `T` commands — analogous to a `goto` label in traditional programming languages.

**Example:**
```bash
sed ':start
s/foo/bar/
tstart' file
```
Here `:start` marks the loop entry point.

### 9.3 `N` — Join / Read Next Line

**Syntax:** `N`

**Concept:** Appends the **next line of input** to the current pattern space, separated by an embedded newline (`\n`). This allows `sed` commands to operate across **multiple lines** at once, which is otherwise impossible since `sed` normally processes one line per cycle.

**Demo Example (43:47 — "join lines (demonstrates how to read one more line)"):**
```bash
sed 'N; s/\n/ /' file
```
**Explanation:**
1. `N` reads the **next line** and appends it to the pattern space (now containing 2 lines joined by `\n`).
2. `s/\n/ /` replaces that embedded newline with a **space**, effectively **joining every pair of lines into one line**.

**Step-by-step for a 4-line file:**
```
Input:                Output:
Line1                 Line1 Line2
Line2         →        Line3 Line4
Line3
Line4
```

**Related command — join ALL lines into one:**
```bash
sed ':a;N;$!ba;s/\n/ /g' file
```
**Explanation:** A classic `sed` idiom:
- `:a` — define label `a`.
- `N` — append the next line.
- `$!ba` — if **not** (`!`) on the last line (`$`), branch back to `a` (keep reading more lines).
- `s/\n/ /g` — once all lines are loaded, replace **all** newlines with spaces.

> **Exam Tip:** `N` is the gateway to **multi-line processing** in `sed` — almost every "operate on pairs of lines" or "join lines" question relies on `N`.

### 9.4 `q` — Quit

**Syntax:** `[address]q [exit-code]`

**Concept:** Immediately stops processing — no further input lines are read, and no further script commands run (the current pattern space is auto-printed first, unless `-n` was given).

**Example:**
```bash
sed '5q' file
```
**Explanation:** Prints the **first 5 lines** and then exits — functionally equivalent to `head -n 5 file`.

```bash
sed '/END/q' file
```
Prints lines up to and including the first line containing `END`, then stops.

### 9.5 `t` / `T` — Conditional Branch

**Syntax:**
```
t [label]     # branch if a substitution succeeded since last input line/branch
T [label]     # branch if NO substitution succeeded since last input line/branch
```

**Concept:** These are the **conditional counterparts** to `b`. They check an internal "substitution made" flag which is **reset** at the start of each new cycle (each new input line) or whenever `t`/`T` itself branches.

**Example — Repeat substitution until no more matches (`t`):**
```bash
sed ':a; s/aa/a/; ta' file
```
**Explanation:** Repeatedly collapses double `a` characters (`aa`) into a single `a`, looping via `ta` **only while** the previous substitution succeeded. Once `s/aa/a/` fails to match (no more `aa` left), the loop naturally exits — unlike the unconditional `b` example in [9.1](#91-b-label--unconditional-branch), this **terminates correctly**.

**Example — Skip further processing if substitution failed (`T`):**
```bash
sed 's/foo/bar/; Tend; s/bar/baz/; :end' file
```
**Explanation:** If the first substitution (`foo`→`bar`) **fails**, `T` branches straight to `:end`, **skipping** the second substitution (`bar`→`baz`). If it succeeds, execution falls through normally and both substitutions apply.

| Command | Branches when... |
|---|---|
| `t label` | The **last** `s` command **succeeded** |
| `T label` | The **last** `s` command **failed** |

### 9.6 `w filename` — Write to File

**Syntax:** `[address]w filename`

**Concept:** Writes the current pattern space to the specified external file, **in addition to** normal output — useful for extracting matching lines into a separate file while still processing/printing the main stream.

**Example:**
```bash
sed -n '/error/w errors.txt' log.txt
```
**Explanation:** Every line containing "error" is written to `errors.txt`; because `-n` is used and there's no `p`, nothing is printed to standard output — this behaves like a **filter that saves to a file**.

### 9.7 `x`, `h`, `H`, `g`, `G` — Hold Space Family

While the slide table only lists `x`, in practice `x` is almost always taught alongside its related hold-space commands, so they are grouped here for completeness.

| Command | Meaning |
|---|---|
| `h` | **Copy** pattern space → hold space (overwrite hold space) |
| `H` | **Append** pattern space → hold space (adds a newline + appends) |
| `g` | **Copy** hold space → pattern space (overwrite pattern space) |
| `G` | **Append** hold space → pattern space (adds a newline + appends) |
| `x` | **Exchange (swap)** pattern space ⇄ hold space |

**Classic Example — Reverse a file's line order (like `tac`):**
```bash
sed -n '1!G;h;$p' file
```
**Explanation (step-by-step):**
1. `1!G` — on every line **except line 1**, append the hold space (accumulated reversed lines so far) to the pattern space.
2. `h` — copy the (growing) pattern space into the hold space, preserving progress for the next cycle.
3. `$p` — on the **last** line, print the fully accumulated (and now reversed) pattern space.

**Simple Example — Double-space a file:**
```bash
sed 'G' file
```
**Explanation:** `G` appends the (empty, at start) hold space to every line, inserting a blank line after each — a quick way to double-space text.

---

## 10. SED with Bash

`sed` is commonly integrated into shell scripting workflows in three main ways:

1. **Including `sed` inside a shell script**
   ```bash
   #!/bin/bash
   sed 's/foo/bar/g' input.txt > output.txt
   ```

2. **Heredoc feature** — embedding a multi-line `sed` script inline within a shell script, without needing a separate `.sed` file:
   ```bash
   sed -f - input.txt <<'EOF'
   s/foo/bar/g
   /^$/d
   EOF
   ```
   **Explanation:** `-f -` tells `sed` to read its script from **standard input**, and the heredoc (`<<'EOF' ... EOF`) supplies that script inline.

3. **Using with other shell commands via pipes**
   ```bash
   cat file.txt | sed 's/foo/bar/g' | sort
   ```
   **Explanation:** `sed` reads from the pipe (standard input) when no filename is given, making it easy to chain with `cat`, `grep`, `sort`, `awk`, etc.

### 🔎 Expanded Notes

| Method | Use Case |
|---|---|
| Inside shell script | Automating repetitive text edits as part of larger automation tasks |
| Heredoc | Keeping multi-line `sed` scripts readable within a bash script without a separate `.sed` file |
| Pipe (`\|`) | Chaining `sed` with other Unix tools (`cat`, `grep`, `sort`, `awk`) for complex pipelines |

---

## 11. Debugging SED Scripts

**Concept:** Because `sed` scripts can become complex (especially with branching via `b`/`t`/`T` and multi-line handling via `N`), GNU `sed` provides built-in tools to trace exactly what is happening during execution.

**Demo Example (47:25 — "Debug"):**
```bash
sed --debug 's/foo/bar/' file
```
**Explanation:** The `--debug` flag (GNU `sed` ≥ 4.6) prints, for every cycle:
- The **input line read**.
- Each **command executed**, along with the **pattern space** and **hold space** contents at each step.
- Whether a substitution succeeded or failed.

This is extremely useful for understanding *why* a script isn't behaving as expected — particularly for scripts involving loops (`b`/`t`), multi-line joins (`N`), or hold-space manipulation (`h`/`H`/`g`/`G`/`x`).

**Other debugging techniques (complementary, not from `--debug` flag):**

| Technique | Purpose |
|---|---|
| `sed -n 'l'` | The `l` (list) command shows the pattern space with **non-printing characters visible** (e.g., `\n`, `\t`) — useful for spotting hidden characters |
| Testing without `-i` first | Always preview output on-screen before committing to in-place edits |
| Building scripts incrementally | Add one command at a time and verify output before adding the next, especially for branch-heavy scripts |
| `echo` + `sed` on sample text | Test a single regex/substitution against a small string before running it on a full file |

**Example — Using `l` to reveal hidden characters:**
```bash
sed -n 'l' file
```
Shows each line with special characters escaped (e.g., a literal tab appears as `\t`), which helps when troubleshooting unexpected `s///` mismatches caused by invisible whitespace.

> **Exam Tip:** `--debug` is a **GNU `sed` extension**, not part of POSIX `sed` — mention this distinction if asked about portability.

---

## 12. Why SED?

- `sed` is **available everywhere** — it is part of POSIX and ships by default on nearly all Unix/Linux systems.
- `sed` is meant for **text processing** and is **fast in execution**, since it processes input in a single, efficient streaming pass rather than loading the whole file into memory (in most typical usage).
- It is commonly used to **pre-process input** before passing it to further processing stages (e.g., before `awk`, `grep`, or custom scripts) — forming a key link in Unix pipelines.

---

## 13. Quick Reference — All Demo Commands (Mapped by Topic)

This table consolidates **every timestamped command** from the practical walkthrough, mapped to the concept/topic it belongs to, for rapid revision.

| Timestamp | Demo Command / Topic | Belongs To (Section) | One-Line Concept |
|---|---|---|---|
| 11:02 | `sed -e "" file` | §3.1 Command Line Mode | Empty script + default auto-print → prints file unchanged |
| 11:50 | `sed -n -e "" file` | §4.1 The `-n` Option | Empty script + `-n` → no output at all |
| 12:48 | `sed -e "=" file` | §8.4 `=` Print Line Number | Prints line number before each line's content |
| 13:28 | Print a particular line | §7.2 Printing a Particular Line | `sed -n '3p' file` |
| 14:15 | Importance of `-n` option | §4.1 The `-n` Option | Prevents duplicate output when using `p` |
| 15:38 | `'p'` vs `'!p'` vs `'$p'` | §7.6 Negation with `!` | Print line / print all-except-line / print last line |
| 16:17 | Address range | §7.5 Range Addresses | `sed -n '5,10p' file` |
| 16:37 | Combine commands | §6.1 Combining Multiple Commands | `sed -n '2,4{s/foo/bar/; p}' file` |
| 17:39 | Print every Nth line | §7.3 Step Addressing | `sed -n '1~2p' file` |
| 18:52 | Regex address | §7.4 Regex Addressing | `sed -n '/error/p' file` |
| 19:36 | `/regex/,+n` | §7.5 Range Addresses | Match + next `n` lines |
| 20:01 | Delete a particular line | §8.2 `d` Delete | `sed '3d' file` |
| 21:17 | Delete a range of lines | §8.2 `d` Delete | `sed '5,10d' file` |
| 21:33 | `/regex/d` | §8.2 `d` Delete | `sed '/DEBUG/d' file` |
| 21:49 | Search and replace | §8.3 `s` Substitute | `sed 's/foo/bar/g' file` |
| 24:32 | Extended regex | §4.2 The `-E`/`-r` Option | `sed -E 's/[0-9]+/NUM/g' file` |
| 26:58 | Range-end as regex | §7.5 Range Addresses | `sed -n '5,/END/p' file` |
| 30:09 | Regex to regex | §7.5 Range Addresses | `sed -n '/START/,/END/p' file` |
| 32:02 | Insert header and footer | §8.9 Headers and Footers | `1i\...` and `$a\...` |
| 33:49 | Insert or append at any line | §8.6 / §8.7 `i` / `a` | Numeric-address insert/append |
| 34:16 | Insert or append @ regex address | §8.6 / §8.7 `i` / `a` | Regex-address insert/append |
| 36:18 | Change a line | §8.8 `c` Change | `sed '3c\New text' file` |
| 38:06 | Sed script file | §3.2 Script Mode | `sed -f script.sed file` |
| 43:47 | Join lines (`N`) | §9.3 `N` Join/Read Next Line | `sed 'N; s/\n/ /' file` |
| 47:25 | Debug | §11 Debugging SED Scripts | `sed --debug 's/foo/bar/' file` |

---

## 14. Summary

- `sed` (**S**tream **ED**itor) is a POSIX-standard, non-interactive text-processing programming language that predates `awk` and is available on virtually every Unix/Linux system.
- Its **execution model** loads each input line into the **pattern space**, runs the script's address/action pairs against it each cycle, and auto-prints the result unless `-n` is used; a separate **hold space** buffer enables advanced cross-line state (via `h`, `H`, `g`, `G`, `x`).
- `sed` can be invoked in three main ways: inline via **`-e`**, from a file via **`-f`**, or as a **standalone executable script** using a shebang.
- Key **command-line options**: `-n` (suppress auto-print — essential when using explicit `p`), `-E`/`-r` (extended regex, no escaping needed for `+`, `?`, `|`, `()`), and `-i` (in-place file editing, ideally with a `.bak` backup).
- **Addressing** is central to `sed` and can be done by:
  - Line number (`5`, `$`, `1~3` for step addressing),
  - Regex (`/pattern/`),
  - Ranges (`5,15`, `/a/,/b/`, `5,/regex/`, `/regex/,+n`, `/regex/,~n`),
  - and can be **negated** with `!` to invert the match.
- **Basic actions** — `p` (print), `d` (delete), `s` (substitute, the most-used command), `=` (line number), `#` (comment), `i`/`a`/`c` (insert/append/change text around or in place of a line) — cover the vast majority of everyday `sed` one-liners, including common idioms like removing blank lines, adding headers/footers, and prefixing/suffixing lines.
- **Programming commands** — `b`/`:label` (unconditional branch/label), `t`/`T` (conditional branch on substitution success/failure), `N` (multi-line joining), `q` (early exit), `w` (write to file), and the hold-space family (`x`, `h`, `H`, `g`, `G`) — turn `sed` into a genuine mini scripting language capable of loops, conditionals, and cross-line logic (e.g., reversing a file, joining lines, deduplicating).
- `sed` integrates seamlessly with **bash**, via inline scripts, heredocs, or Unix pipes, making it a foundational building block of shell-based text-processing pipelines.
- **Debugging** complex scripts (loops, multi-line handling) is aided by GNU `sed`'s `--debug` flag and the `l` (list) command for revealing hidden characters.
- **Bottom line:** `sed` is fast, universally available, POSIX-compliant, and ideal for both simple one-line text edits **and** more advanced, programmatic multi-line stream transformations — making it a core tool for pre-processing text before further analysis (e.g., by `awk` or custom scripts).

---






## 15. Quick Revision Sheet


| Category | Key Symbols/Commands to Remember |
|---|---|
| Invocation | `-e` (inline), `-f` (script file), `-n` (suppress auto-print), `-i` 
(in-place edit) |
| Address types | `5`, `$`, `1~3`, `/regexp/`, `5,15`, `/re1/,/re2/`, `5,/re/`, `/re/
+4`, `/re/,~2` |
| Basic actions | `p`, `d`, `s///`, `=`, `#`, `i`, `a`, `c` |
| Flow control | `b`, `:label`, `N`, `q`, `t`, `T`, `w`, `x` |
| Buffers | Pattern space (active) vs. Hold space (auxiliary) |
| Special chars | `;` separator, `,` range, `!` negation, `{ }` grouping |

*End of Notes — Week 8, Lecture 2: SED (Expanded Edition)*
