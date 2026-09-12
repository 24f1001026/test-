# AWK Programming — Week 9 (Lecture 1 & 2)


## 1. Introduction to AWK

### 1.1 What is AWK?

AWK is a **general-purpose programming language** built specifically for **scanning and processing text files**. Unlike a general-purpose language such as C or Python — which requires many lines of code just to open a file, read it line by line, and split each line into pieces — AWK does all of that **automatically**, so the programmer can focus purely on *what to do* with the data rather than *how to read it*.

AWK is best understood as sitting between simple filters like `grep`/`sed` and a full programming language:

| Tool | Capability |
|---|---|
| `grep` | Finds lines matching a pattern — cannot transform or compute |
| `sed` | Performs simple text substitution / stream editing |
| **AWK** | Full pattern-matching **and** programming language: variables, arrays, functions, arithmetic, loops, formatted output |
| Python / Perl | Full-scale general-purpose languages — more powerful but far more verbose for simple field-based text processing |

### 1.2 Origin and Naming

AWK is a **programming language**, and the name `awk` is an abbreviation formed from the surnames of its three original developers at Bell Labs (1977):

| Letter | Developer |
|---|---|
| **A** | Alfred **A**ho |
| **W** | Peter **W**einberger |
| **K** | Brian **K**ernighan |

*(Brian Kernighan is also well known as co-author of "The C Programming Language" and co-creator of the "Hello, World!" convention.)*

### 1.3 Standardization

AWK is not just a random Unix utility — it has been formally standardized:

- It is a part of **POSIX**, specifically **IEEE 1003.1-2008**.

- This means any POSIX-compliant AWK implementation is guaranteed to support a common, predictable core set of features, which makes AWK scripts portable across different Unix/Linux systems.

### 1.4 Variants of AWK

| Variant | Description |
|---|---|
| `nawk` | "New AWK" — an early rewrite of the original AWK, closer to the POSIX standard |
| `gawk` | **GNU AWK** — the most widely used, most feature-rich variant, found on almost all Linux distributions by default |
| `mawk` | A **fast**, minimalist implementation optimized for speed and small memory footprint |
| Others | e.g., `busybox awk` (a stripped-down version for embedded systems), `goawk`, `original awk` (`the one true awk`, maintained by Kernighan himself) |

**Important distinction:** `gawk` contains features that **extend** POSIX AWK — meaning `gawk` can do things that plain POSIX AWK cannot (such as `asort()`, `asorti()`, `strtonum()`, bit-wise functions, and enhanced multi-file handling). A script that uses `gawk`-only features may **not** run correctly under `mawk` or `nawk`.

**Checking which variant the `awk` command actually points to:**
On most Linux distributions, the plain command `awk` is not a standalone program — it is a **symbolic link** that points to whichever variant is actually installed (usually `gawk`). The `realpath` command can be used to resolve this link and confirm exactly which binary will run.

```bash
which awk
realpath $(which awk)
```
Example output:
```
/usr/bin/awk
/usr/bin/gawk
```
*Explanation:* `which awk` shows the first `awk` found on the shell's `PATH`; `realpath` then follows the symbolic link to reveal the **actual underlying binary** — here, confirming that `awk` is really `gawk` under the hood. This is a useful sanity check before relying on any `gawk`-only feature.

### 1.5 Why Learn AWK? (Real-World Motivation)

- Log file analysis (extracting specific fields from web server logs, system logs).

- Quick data extraction from CSV/TSV files without opening a full spreadsheet program.

- Generating simple reports and summaries directly from the command line.

- Preprocessing data before feeding it into another program.

- It is a **default command available on almost every Unix/Linux machine**, so it requires no installation.

### 1.6 Exam-Focused Notes

- AWK is *not* just a command — it is a full scripting/programming language.

- Remember the acronym expansion: **A**ho, **W**einberger, **K**ernighan — a very common one-mark / fill-in-the-blank question.

- `gawk` is the most feature-rich and most commonly available variant on Linux systems; when a textbook says simply "awk", it usually means `gawk` in practice on Linux.

- POSIX standard number to remember: **IEEE 1003.1-2008**.

---

## 2. Execution Model

### 2.1 The Core Idea — Read, Process, Write

AWK's entire design revolves around a simple but powerful three-stage execution model, often summarized as **Read → Process → Write**:

| Stage | What Happens |
|---|---|
| **Read** | AWK reads one record at a time from the input (by default, one line at a time) |
| **Process** | AWK checks the record against every pattern in the script and runs the matching action(s), which may compute, transform, or store values |
| **Write** | Any output produced by `print`/`printf` during processing is written to standard output (or to a file/pipe, if redirected) |

This Read → Process → Write cycle repeats **automatically** for every single record until the input is exhausted, and is the reason AWK scripts are so much shorter than equivalent C or Python code for text-processing tasks — the programmer never has to write the reading loop themselves.

### 2.2 Step-by-Step Breakdown

1. **The input stream is a set of records.**
   By default, the **record separator** (`RS`) is the newline character `"\n"`, so **each line of input becomes one record**.

2. **Each record is a sequence of fields.**
   By default, the **field separator** (`FS`) is a single space (technically, any run of whitespace — spaces and/or tabs), so **each word becomes one field**.

3. **Splitting of records into fields is done automatically.**
   The programmer never has to manually call a "split" function on every line — AWK does this internally the moment a record is read, and immediately populates `$1`, `$2`, ..., `$NF`, and `NF`.

4. **Each code block executes on one record at a time**, evaluated against the **pattern** attached to that block. If the pattern is true (or absent), the block's action runs for that record.

### 2.3 Conceptual Flow Diagram

```
Input Stream
    │
    ▼
Split into RECORDS   (default separator: RS = "\n"  → one record per line)
    │
    ▼
Split each record into FIELDS   (default separator: FS = " "  → one field per word)
    │
    ▼
For each record:
    → Evaluate pattern of Block 1 → if TRUE → run Block 1's action
    → Evaluate pattern of Block 2 → if TRUE → run Block 2's action
    → ... (repeat for every block in the script, in order)
```

### 2.4 Changing the Record and Field Separators

Both `RS` and `FS` can be customized, and this is one of the most frequently tested practical skills in AWK.

**Example — default behavior (line = record, whitespace = field):**
```bash
awk '{ print $1, $2 }' data.txt
```

**Example — custom field separator using `-F`:**
```bash
awk -F":" '{ print $1 }' /etc/passwd
```
*Explanation:* Colon (`:`) is now treated as the field separator, so each colon-delimited value in `/etc/passwd` becomes a separate field.

**Example — custom field separator set inside `BEGIN`:**
```bash
awk 'BEGIN{ FS="," } { print $2 }' data.csv
```

**Example — custom record separator (treat blank lines as record boundaries — paragraph mode):**
```bash
awk 'BEGIN{ RS="" ; FS="\n" } { print "Paragraph:", $1 }' notes.txt
```
*Explanation:* Setting `RS=""` switches AWK into **paragraph mode**, where a *blank line* separates records instead of every single newline. This is a well-known `gawk` trick for processing multi-line paragraphs as single records.

**Example — multi-character record separator (gawk extension):**
```bash
awk 'BEGIN{ RS=";" } { print NR, $0 }' data.txt
```
*Explanation:* Now, instead of newlines, the semicolon (`;`) character splits the input into records.

**Example — field separator as a regular expression:**
```bash
echo "one,  two,three" | awk -F'[,]+[ ]*' '{ print $1, $2, $3 }'
```
*Explanation:* `FS` can be a **regular expression**, not just a single character — here, it matches a comma followed by any number of spaces, so both `", "` and `","` are treated as separators.

**Example — setting multiple different separator characters at once using a character class:**
```bash
echo "10.20;30:40-50" | awk 'BEGIN{ FS="[ .;:-]" } { print $1, $2, $3, $4, $5 }'
```
Output:
```
10 20 30 40 50
```
*Explanation:* `FS="[ .;:-]"` is a **regex character class** meaning "match any single one of these characters: space, dot, semicolon, colon, or hyphen." This lets a single record be split correctly even when several *different* delimiter characters are mixed together in the same line — extremely useful for messy, inconsistently formatted log files or data dumps.

### 2.5 Example Showing the Full Model End-to-End

Input file `data.txt`:
```
Alice 25 Engineer
Bob 30 Doctor
Charlie 22 Artist
```

Command:
```bash
awk '{ print $1, $2 }' data.txt
```

Output:
```
Alice 25
Bob 30
Charlie 22
```

*Explanation:* Each line is a record (default `RS = "\n"`), and each word is a field (default `FS = " "`). `$1` refers to the first field, `$2` to the second field. `$0` (not printed here) would refer to the **entire original record**, e.g., `"Alice 25 Engineer"`.

### 2.6 Common Pitfalls (Exam Traps)

- Forgetting that a `FS` change made inside `BEGIN` applies from the **first record onward**, but a `FS` change made in the middle of the script (outside `BEGIN`) only takes effect **starting from the next record read**, not the current one.

- Confusing `$0` (whole record) with `$1` (first field only).

- Assuming AWK's default field separator is *exactly* one space — it is actually **any amount of leading/trailing whitespace**, and multiple consecutive spaces/tabs are treated as a single separator, when `FS` is left at its default value.

---

## 3. Usage of AWK

AWK can be used in **two main ways**: directly on the command line for quick, one-off tasks, or as a standalone interpreted script for longer, reusable programs.

### 3.1 Running AWK from the Command Line

**Basic syntax:**
```bash
awk [options] 'program' [file(s)]
```
or, reading from a pipe:
```bash
command | awk [options] 'program'
```

**Common command-line options:**

| Option | Meaning |
|---|---|
| `-F fs` | Sets the field separator (`FS`) before processing begins |
| `-v var=value` | Assigns a value to a variable **before** the `BEGIN` block runs |
| `-f scriptfile` | Reads the AWK program from a file instead of the command line (can be repeated) |

**Example (from the source material):**
```bash
cat /etc/passwd | awk -F":" '{print $1}'
```
*Explanation:*
- `cat /etc/passwd` streams the contents of the password file.
- `-F":"` tells AWK to use `:` (colon) as the **field separator** instead of the default whitespace.
- `{print $1}` prints only the **first field** of every record — in this case, the username.

**Example — direct file argument (no piping needed):**
```bash
awk -F":" '{print $1}' /etc/passwd
```
*Explanation:* AWK can read files directly by name — piping through `cat` is not required, though it is a very common habit.

**Example — passing a variable with `-v`:**
```bash
awk -v threshold=25 '$2 > threshold { print $1 }' data.txt
```
*Explanation:* The `-v` option lets you inject external values into the AWK script from the shell, useful for parameterizing scripts without editing the source.

**Example — processing multiple files at once:**
```bash
awk '{ print FILENAME, $1 }' file1.txt file2.txt
```
*Explanation:* AWK can accept multiple filenames; it processes them one after another, and `FILENAME` tells you which file the current record came from.

**Example — a one-liner with no pattern, printing everything (equivalent to `cat`):**
```bash
awk '{ print }' data.txt
```

**Example — a one-liner using only a pattern, no explicit action (default action is `{print $0}`):**
```bash
awk '/Engineer/' data.txt
```
*Explanation:* If a block has a pattern but **no action**, AWK's default action is to print the whole current record (`$0`). This is identical in effect to `grep "Engineer" data.txt`.

### 3.2 Running AWK as a Script

AWK scripts are usually saved with a `.awk` extension and can include a **shebang line** so they can be executed directly, like any shell script.

**Syntax:**
```bash
./myscript.awk /etc/passwd
```

**Example script — `myscript.awk`:**
```awk
#!/usr/bin/gawk -f

BEGIN {
    FS = ":"
}
{
    print $1
}
```

*Explanation:*
- `#!/usr/bin/gawk -f` — the shebang line, telling the shell to run the file using `gawk` in script (`-f`) mode.
- `BEGIN { FS = ":" }` — runs **once**, before any input is read, and sets the field separator to `:`.
- `{ print $1 }` — runs for **every record** and prints the first field.

**How to make it executable and run it:**
```bash
chmod +x myscript.awk
./myscript.awk /etc/passwd
```

**Example — running a script without the shebang, using `-f` explicitly:**
```bash
awk -f myscript.awk /etc/passwd
```
*Explanation:* Even without `chmod +x` or a shebang line, any AWK program saved in a file can be executed with the `-f` flag.

**Example — combining `-f` with a filename argument and a `-v` variable:**
```bash
awk -f myscript.awk -v mode="verbose" /etc/passwd
```

**Note (typo correction):** The typographic quotes (`"` or `”`) shown in some textbook slides/presentations must be replaced with standard straight quotes (`"`) when typing real AWK code — otherwise the shell or the AWK interpreter will throw a syntax error. This is a very common copy-paste mistake when transcribing notes from slides into a real terminal.

### 3.3 Comparison: Command-Line AWK vs. Script AWK

| Aspect | Command-Line AWK | Script File AWK |
|---|---|---|
| Best for | Quick, one-off, single-line tasks | Longer, reusable, multi-block programs |
| Readability | Harder to read for complex logic (all on one line) | Much more readable — proper indentation, comments |
| Reusability | Must be retyped or recalled from shell history each time | Saved permanently as a file; can be version-controlled |
| Shareable functions | Not practical | Can use `-f` with multiple files (library + main script) |

### 3.4 Comments in AWK Scripts

**Concept:** Just like most scripting languages, AWK allows the programmer to leave explanatory notes in the source code that are ignored during execution. This is essential for making longer scripts (like the library + main script setup in Section 11) understandable and maintainable.

**Syntax:**
```awk
# This is a comment — everything from the # to the end of the line is ignored
```

*Explanation:* AWK uses the hash symbol (`#`) for comments. There is no special syntax for multi-line comments — every line that needs a comment must start with its own `#`.

**Example — a full-line comment:**
```bash
awk '
# This script prints the first field of every record
{ print $1 }
' data.txt
```

**Example — an inline (end-of-line) comment:**
```bash
awk '{ print $1 }  # prints only the first field' data.txt
```

**Example — a well-commented script file:**
```awk
#!/usr/bin/gawk -f
# Script: report.awk
# Purpose: Summarize employee ages by department
# Author: Exam Prep Notes

BEGIN {
    FS = ","          # input is comma-separated
    print "Department Summary"
}

{
    total[$3] += $2   # accumulate age by department (field 3)
    count[$3]++        # track how many employees per department
}

END {
    # Print the average age for each department
    for (dept in total)
        printf "%-15s %.2f\n", dept, total[dept]/count[dept]
}
```

---

## 4. Built-in Variables

### 4.1 Concept

AWK automatically maintains a set of special variables that describe the current state of parsing (current record, current field separator, counts, etc.). These do **not** need to be declared — they are ready to use, and most are updated automatically by AWK itself as it reads through the input.

Because this is a large topic, it is broken into **two grouped tables**, and then **every single variable gets its own worked example** below so nothing is left unexplained.

### Table 4A — Record, Field, and File-Related Variables

| Variable | Meaning |
|---|---|
| `ARGC` | Number of arguments supplied on the command line (excluding those consumed by `-f` and `-v` options) |
| `ARGV` | Array of command-line arguments supplied; indexed from `0` to `ARGC-1` |
| `ENVIRON` | Associative array containing the process's environment variables |
| `FILENAME` | Name of the current file being processed |
| `FNR` | Number of the current record, **relative to the current file** (resets to 1 for each new file) |
| `FS` | Field separator; can be a single character or a regular expression |
| `NF` | Number of fields in the current record |
| `NR` | Number of the current record, **across all files** processed so far (does not reset) |
| `OFMT` | Output format used for printing numbers |

### Table 4B — Output, Separator, and Function-Support Variables

| Variable | Meaning |
|---|---|
| `OFS` | Output field separator (used by `print` when joining fields with commas) |
| `ORS` | Output record separator (printed at the end of each `print` statement) |
| `RS` | Record separator (defines what counts as one "record" of input) |
| `RLENGTH` | Length of the string matched by the `match()` function |
| `RSTART` | Starting position in the string matched by the `match()` function |
| `SUBSEP` | Separator character used for constructing multi-dimensional array subscripts |
| `$0` | The entire current input record |
| `$n` | The `n`-th field of the current record (e.g., `$1`, `$2`, ...) |

### 4.2 Every Built-in Variable Explained Individually, With Examples

#### `ARGC` and `ARGV`

`ARGV` is an array holding every command-line argument (including the string `"awk"` itself at index 0), and `ARGC` holds the count of those arguments.

```bash
awk 'BEGIN{ for (i=0; i<ARGC; i++) print i, ARGV[i] }' file1.txt file2.txt
```
Output:
```
0 awk
1 file1.txt
2 file2.txt
```

**Additional example — `ARGC` when there are no file arguments at all (input comes from a pipe instead):**
```bash
echo "hello world" | awk 'END{ print ARGC }'
```
Output:
```
1
```
*Explanation:* Since no filenames were passed on the command line (the data arrived via a pipe instead), `ARGV` only contains index `0` (the program name itself), so `ARGC` is `1`.

**Additional example — iterating `ARGV` with `for...in`:**
```bash
awk 'BEGIN{ for (i in ARGV) print ARGV[i] }' file1.txt file2.txt
```
*Explanation:* This is functionally similar to the numeric `for` loop shown above, but uses `for (i in ARGV)` to iterate over the array's indices directly rather than counting up to `ARGC` manually. The **iteration order is not guaranteed** with `for...in` in the general case, so for strictly ordered output (index 0, 1, 2, ...) the counting `for` loop is the safer choice.

#### `ENVIRON`

An associative array exposing the shell's environment variables to the AWK program.

```bash
awk 'BEGIN{ print ENVIRON["HOME"] }'
```
Output (example):
```
/home/student
```

#### `FILENAME`

```bash
awk '{ print FILENAME, $0 }' file1.txt file2.txt
```
*Explanation:* Prints the source filename alongside every record — useful when processing several files at once and you need to know where each line came from.

#### `FNR` vs `NR` (classic exam comparison)

`file1.txt`:
```
a
b
```
`file2.txt`:
```
c
d
```

Command:
```bash
awk '{print "NR="NR, "FNR="FNR, "FILE="FILENAME}' file1.txt file2.txt
```

Output:
```
NR=1 FNR=1 FILE=file1.txt
NR=2 FNR=2 FILE=file1.txt
NR=3 FNR=1 FILE=file2.txt
NR=4 FNR=2 FILE=file2.txt
```

**Key distinction to remember (very common exam trap):**

- `NR` = record number across **all** input files combined (keeps counting upward).

- `FNR` = record number **within the current file only** (resets to 1 whenever a new file starts).

**Practical use of `FNR == NR` (a famous AWK idiom):** this comparison is `true` only while AWK is still reading the *first* file, and becomes `false` once it starts the *second* file — commonly used to load one file into an array before comparing against a second file.

```bash
awk 'FNR==NR { seen[$1]=1; next } !( $1 in seen ) { print "New:", $1 }' old.txt new.txt
```
*Explanation:* While reading `old.txt` (`FNR==NR` is true), every first field is stored in array `seen`. Once AWK moves to `new.txt`, `FNR==NR` becomes false, so the second rule runs instead, printing any first field from `new.txt` that was **not** present in `old.txt`.

#### `FS` (already covered in depth in Section 2.4, repeated briefly here for completeness)

```bash
awk 'BEGIN{ FS="," } { print $1 }' data.csv
```

#### `NF`

```bash
echo "one two three" | awk '{ print "Number of fields:", NF }'
```
Output:
```
Number of fields: 3
```

**Bonus use — printing the last field regardless of how many fields exist:**
```bash
echo "one two three four" | awk '{ print $NF }'
```
Output:
```
four
```
*Explanation:* `$NF` always refers to the **last** field because `NF` holds the total field count, and `$` combined with a variable dynamically resolves to that field number.

**Bonus use — deleting the last field:**
```bash
echo "one two three four" | awk '{ NF--; print }'
```
Output:
```
one two three
```

#### `OFMT`

Controls how AWK formats floating-point numbers when they are **printed** with `print` (not `printf`).

```bash
awk 'BEGIN{ OFMT="%.2f"; print 3.14159 }'
```
Output:
```
3.14
```

**Additional example — applying `OFMT` to a computed field value (a common practical use):**
```bash
echo "10" | awk 'BEGIN{ OFMT="%.3f" } { print $1/3 }'
```
Output:
```
3.333
```
*Explanation:* `$1/3` produces a non-terminating decimal; `OFMT="%.3f"` controls how many digits `print` shows when the result is an "impure" (non-integer) number.

#### `OFS`

```bash
echo "one two three" | awk 'BEGIN{OFS="-"} {print $1,$2,$3}'
```
Output:
```
one-two-three
```
*Explanation:* `OFS` only takes effect when `print` joins **multiple comma-separated arguments** — it has no effect on `$0` unless a field is reassigned (see below).

**Additional example — joining just two fields with a comma (CSV-style output):**
```bash
echo "Alice 25" | awk 'BEGIN{OFS=","} {print $1, $2}'
```
Output:
```
Alice,25
```
*Explanation:* This is a very common real-world use of `OFS` — reading whitespace-separated input and re-emitting it as comma-separated (CSV) output.

**Important subtlety (common exam trap):** simply changing `OFS` does **not** automatically rewrite `$0` — `$0` is only rebuilt (using the new `OFS`) the moment you **assign to any field** (like `$1="x"`) or to `NF`.
```bash
echo "a b c" | awk 'BEGIN{OFS="-"} {$1=$1; print}'
```
Output:
```
a-b-c
```
*Explanation:* `$1=$1` forces AWK to rebuild `$0` using the current `OFS`, even though the value of `$1` doesn't actually change.

#### `ORS`

```bash
printf "a\nb\nc\n" | awk 'BEGIN{ORS=" | "} {print}'
```
Output:
```
a | b | c |
```
*Explanation:* `ORS` (default `"\n"`) is appended after every `print` statement — changing it to `" | "` turns line-by-line output into a single space-and-pipe-separated line.

**Additional example — inserting a blank line after every record (double-spacing output):**
```bash
printf "line1\nline2\nline3\n" | awk 'BEGIN{ORS="\n\n"} {print $0}'
```
Output:
```
line1

line2

line3

```
*Explanation:* Setting `ORS="\n\n"` makes AWK insert an **extra blank line** after every record it prints — a quick way to visually double-space a file's output.

#### `RS`

Already demonstrated in Section 2.4 (paragraph mode and custom separators). Additional examples:

```bash
printf "one;two;three" | awk 'BEGIN{RS=";"} {print NR, $0}'
```
Output:
```
1 one
2 two
3 three
```

**Additional example — using a comma as the record separator (splitting on commas instead of newlines):**
```bash
printf "10,20,30,40" | awk 'BEGIN{RS=","} {print $0}'
```
Output:
```
10
20
30
40
```
*Explanation:* With `RS=","`, AWK treats every comma-delimited chunk of the input as a **separate record**, printing one per line — effectively turning a single comma-separated line into a column of values.

#### `RSTART` and `RLENGTH`

Both are set automatically by the `match()` function (see Section 8.2).

```bash
awk 'BEGIN{
    match("hello world", /wor/)
    print "Start:", RSTART, "Length:", RLENGTH
}'
```
Output:
```
Start: 7 Length: 3
```

**Additional example — using `match()`'s own return value together with `RSTART`:**
```bash
awk 'BEGIN{ print match("hello world", /world/) }'
```
Output:
```
7
```
*Explanation and correction:* The `match(s, re)` function **returns the starting position** of the match (the same value it stores in `RSTART`) — **not** the length of the matched text. It is a common mix-up to assume `match()`'s return value represents the match *length*; the length is only available separately, via `RLENGTH`, as shown below:
```bash
awk 'BEGIN{ match("hello world", /world/); print RLENGTH }'
```
Output:
```
5
```
*Explanation:* `RLENGTH` correctly reports that the matched text `"world"` is `5` characters long, while the earlier example's return value (`7`) was the **starting position** of that match within the string.

#### `SUBSEP`

Used internally to build the composite index for multi-dimensional arrays (see Section 9.5).

```bash
awk 'BEGIN{ print length(SUBSEP) }'
```
Output:
```
1
```
*Explanation:* `SUBSEP` is a single, typically non-printable character (`\034`), used purely as an internal separator — programmers rarely print it, but must know it exists and what it's for.

**Additional example — the exact mechanism `SUBSEP` enables (a multi-dimensional array lookup):**
```bash
awk 'BEGIN{ a["hello","world"]=1; print a["hello","world"] }'
```
Output:
```
1
```
*Explanation:* Behind the scenes, the index `"hello","world"` is automatically joined into a single string `"hello" SUBSEP "world"` before being used as the actual array key. This is precisely what allows Section 9.5's simulated multi-dimensional arrays (like `matrix[i,j]`) to work.

#### `$0`

```bash
echo "  Hello   World  " | awk '{ print "[" $0 "]" }'
```
Output:
```
[  Hello   World  ]
```
*Explanation:* `$0` always holds the **entire, unmodified current record**, including any leading/trailing whitespace exactly as read (unless fields are reassigned).

#### `$n`

```bash
echo "one two three" | awk '{ print $2 }'
```
Output:
```
two
```

**Bonus — computed field references:**
```bash
echo "one two three" | awk '{ n=2; print $n }'
```
Output:
```
two
```
*Explanation:* The `$` operator can be followed by any expression, not just a literal number — `$n`, `$(NF-1)`, `$(i+1)` are all valid.

### 4.3 Summary Table — Default Values

| Variable | Default Value |
|---|---|
| `FS` | `" "` (space — actually, any whitespace run) |
| `OFS` | `" "` (single space) |
| `RS` | `"\n"` (newline) |
| `ORS` | `"\n"` (newline) |
| `SUBSEP` | `"\034"` (non-printable) |
| `NR`, `FNR` | `0` before any record is read |

---

## 5. Structure of an AWK Script

### 5.1 Concept

Every AWK program is fundamentally a series of **pattern–action pairs**. This is the single most important structural idea in the entire language, and almost every AWK question in an exam boils down to correctly identifying or writing pattern–action pairs.

**General syntax:**
```awk
pattern { procedure }
```

Either the `pattern` or the `{ procedure }` may be omitted (but not both):

| Form | Meaning |
|---|---|
| `pattern { action }` | Run `action` only on records where `pattern` is true |
| `pattern` (no action) | Default action is `{ print $0 }` — print the matching record |
| `{ action }` (no pattern) | Pattern is always considered true — run `action` on **every** record |

### 5.2 Building Blocks That Can Appear Inside a Script

An AWK script can be composed of the following building blocks, in any combination:

- `BEGIN` block

- `END` block

- General expressions

- Regular expressions (regex)

- Relational expressions

- Pattern-matching expressions

- Variable assignments

- Array assignments

- Input / Output commands

- Built-in functions

- User-defined functions

- Control loops

### 5.3 Examples of Each Kind of Pattern

**General expression pattern (true if the expression is non-zero / non-empty):**
```bash
awk 'NF' data.txt
```
*Explanation:* Since `NF` (number of fields) is used directly as the pattern, this prints every record that has **at least one field** — effectively, it skips blank lines.

**Regex pattern:**
```bash
awk '/Engineer/' data.txt
```
*Explanation:* Matches (and by default prints) any record where the text `Engineer` appears anywhere in `$0`.

**Relational expression pattern:**
```bash
awk '$2 > 25' data.txt
```

**Pattern-matching (`~` / `!~`) expression:**
```bash
awk '$1 ~ /^A/' data.txt
```

**Combined pattern using logical operators:**
```bash
awk '$2 > 20 && $3 == "Engineer"' data.txt
```

**Range pattern:**
```bash
awk '/START/,/END/' logfile.txt
```

### 5.4 Multiple Pattern–Action Pairs in One Script

A single AWK script is not limited to one rule — it can contain **any number** of pattern–action pairs, and every single one of them is checked, in order, against **every** record.

```bash
awk '
$2 > 25 { print $1, "is over 25" }
$3 == "Doctor" { print $1, "is a doctor" }
{ total++ }
END { print "Total records:", total }
' data.txt
```
*Explanation:* For every record, AWK checks all three rules in order. The first two only fire conditionally; the third (no pattern) fires on every record to keep a running count, and the `END` block reports the final total once all input is exhausted.

---

## 6. Execution Blocks — BEGIN, END, Pattern Blocks

### 6.1 Concept

AWK programs are organized into blocks that run at specific, well-defined times during execution. Understanding exactly **when** each block type runs is essential, both for exams and for writing correct scripts.

**Syntax of the four core block types:**
```awk
BEGIN { commands; }        # runs once, before any file is read

END { commands; }          # runs once, after all files are read

{ commands; }               # runs on every record (no pattern = always true)

pattern { commands; }      # runs only on records where 'pattern' evaluates true
```

### 6.2 Rules and Properties

| Block Type | When It Runs | Notes |
|---|---|---|
| `BEGIN { }` | **Once**, before any input is read | Used for initialization (e.g., setting `FS`, printing headers/titles) |
| `END { }` | **Once**, after all input has been read | Used for summaries, totals, final reports |
| `{ }` (no pattern) | For **every** record | Runs unconditionally, once per record |
| `pattern { }` | Once per record **where the pattern evaluates to true** | The script can contain multiple such blocks |

**Additional rules (important for exams):**

- These blocks **can appear anywhere** in the script — AWK does not require `BEGIN` to be physically first, though it is best practice to write it first for readability.

- They **can appear multiple times** — e.g., you may have more than one `BEGIN` block; they all run, **in the order they are written**, before input processing starts. The same is true for multiple `END` blocks.

- Patterns can be **combined** using logical operators: `&&` (AND), `||` (OR), `!` (NOT).

- A **range of records** can be specified using a **comma** between two patterns (range pattern) — the action runs for every record from the first matching line through the line that matches the second pattern (inclusive).

### 6.3 Worked Examples

**Example — multiple `BEGIN` blocks:**
```bash
awk 'BEGIN{print "Step 1"} BEGIN{print "Step 2"} {print}' data.txt
```
Output (first two lines, before any data is printed):
```
Step 1
Step 2
```

**Example — BEGIN and END together:**
```bash
awk 'BEGIN{print "Start of Report"} {print} END{print "End of Report. Total lines:", NR}' data.txt
```

**Example — Pattern block (condition on a field):**
```bash
awk '$2 > 25 { print $1, "is older than 25" }' data.txt
```
*Explanation:* This prints the first field of every record only where the second field's value is greater than 25.

**Example — Combining patterns with `&&` and `||`:**
```bash
awk '$2 > 20 && $3 == "Engineer" { print $1 }' data.txt
```

**Example — using `!` (NOT):**
```bash
awk '!($3 == "Doctor") { print $1 }' data.txt
```
*Explanation:* Prints the first field of every record **except** those where the third field equals `"Doctor"`.

**Example — Range pattern (comma-separated):**
```bash
awk '/START/,/END/ { print }' logfile.txt
```
*Explanation:* Prints all lines starting from the line that matches `/START/` up to (and including) the line that matches `/END/`.

**Example — Range pattern using line numbers via `NR`:**
```bash
awk 'NR==2,NR==5' data.txt
```
*Explanation:* Prints only lines 2 through 5 (inclusive) — a very handy substitute for `sed -n '2,5p'`.

**Example — `exit` inside a pattern block to stop early but still trigger `END`:**
```bash
awk '{ if (NR==3) exit } { print } END{ print "Stopped after", NR, "records" }' data.txt
```
*Explanation:* `exit` immediately stops reading further input, but AWK still executes the `END` block afterward (unless `exit` is called from *within* `END` itself, in which case it terminates immediately).

---

## 7. Operators in AWK

AWK supports several categories of operators. These are grouped below exactly as they should be studied, with each category expanded into its own explanation table and its own set of examples.

### 7.1 Assignment Operators

| Operator | Meaning |
|---|---|
| `=` | Simple assignment |
| `+=` | Add and assign |
| `-=` | Subtract and assign |
| `*=` | Multiply and assign |
| `/=` | Divide and assign |
| `%=` | Modulus and assign |
| `^=` | Exponentiate and assign |
| `**=` | Exponentiate and assign (alternate form) |

**Examples for every assignment operator:**
```bash
awk 'BEGIN{ x=5;  x+=3;  print x }'    # Output: 8
awk 'BEGIN{ x=5;  x-=3;  print x }'    # Output: 2
awk 'BEGIN{ x=5;  x*=2;  print x }'    # Output: 10
awk 'BEGIN{ x=10; x/=2;  print x }'    # Output: 5
awk 'BEGIN{ x=10; x%=3;  print x }'    # Output: 1
awk 'BEGIN{ x=2;  x^=3;  print x }'    # Output: 8
awk 'BEGIN{ x=2;  x**=3; print x }'    # Output: 8
```

### 7.2 Logical Operators

| Operator | Meaning |
|---|---|
| Double pipe (two vertical-bar characters together) | Logical OR |
| `&&` | Logical AND |

**Examples:**
```bash
awk 'BEGIN{ if (5>3 && 2<4) print "Both true" }'
```
Output:
```
Both true
```

```bash
awk 'BEGIN{ if (5<3 || 2<4) print "At least one true" }'
```
Output:
```
At least one true
```

**Example combining logical operators with field data:**
```bash
awk '$2>18 && $2<60 { print $1, "is working age" }' data.txt
```

### 7.3 Algebraic (Arithmetic) Operators

| Operator | Meaning |
|---|---|
| `+` | Addition |
| `-` | Subtraction |
| `*` | Multiplication |
| `/` | Division |
| `%` | Modulus (remainder) |
| `^` | Exponentiation |
| `**` | Exponentiation (alternate form) |

**Examples for every arithmetic operator:**
```bash
awk 'BEGIN{ print 5+3 }'     # Output: 8
awk 'BEGIN{ print 5-3 }'     # Output: 2
awk 'BEGIN{ print 5*3 }'     # Output: 15
awk 'BEGIN{ print 5/2 }'     # Output: 2.5
awk 'BEGIN{ print 10 % 3 }'  # Output: 1
awk 'BEGIN{ print 2^10 }'    # Output: 1024
awk 'BEGIN{ print 2**10 }'   # Output: 1024
```

**Example — computing an average across fields:**
```bash
echo "10 20 30" | awk '{ print ($1+$2+$3)/3 }'
```
Output:
```
20
```

### 7.4 Relational Operators

| Operator | Meaning |
|---|---|
| `>` | Greater than |
| `<` | Less than |
| `>=` | Greater than or equal to |
| `<=` | Less than or equal to |
| `!=` | Not equal to |
| `==` | Equal to |

**Examples for every relational operator:**
```bash
awk 'BEGIN{ print (5>3) }'    # Output: 1 (true)
awk 'BEGIN{ print (5<3) }'    # Output: 0 (false)
awk 'BEGIN{ print (5>=5) }'   # Output: 1
awk 'BEGIN{ print (5<=4) }'   # Output: 0
awk 'BEGIN{ print (5!=3) }'   # Output: 1
awk 'BEGIN{ print (5==5) }'   # Output: 1
```
*Explanation:* AWK does not have a distinct Boolean type — comparisons evaluate to `1` (true) or `0` (false), which can themselves be printed or used in arithmetic.

**Example with field data:**
```bash
awk '$1 == "Alice" { print "Found Alice" }' data.txt
```

**Important note on string vs numeric comparison:** AWK automatically decides whether to compare two values as numbers or as strings based on context. If both operands **look like** numbers (either are numeric constants or came from input and look numeric), AWK compares them **numerically**; otherwise, it compares them as **strings** (lexicographically).
```bash
awk 'BEGIN{ print ("10" == 10) }'     # Output: 1  (numeric comparison, since "10" looks numeric)
awk 'BEGIN{ print ("abc" == "abc") }' # Output: 1  (string comparison)
awk 'BEGIN{ print ("10" < "9") }'     # Output: 1  (STRING comparison: "1" < "9" lexicographically!)
```
*Explanation:* The last example is a classic exam trap — since both `"10"` and `"9"` are **string constants** typed directly with quotes (not derived from arithmetic), AWK compares them as strings, and `"1"` sorts before `"9"` character-by-character, so `"10" < "9"` is **true** — even though numerically 10 is greater than 9.

### 7.5 Special / Miscellaneous Operators

These are equally important but conceptually distinct from the arithmetic/relational groups above, so they are separated into their own table.

| Operator | Meaning |
|---|---|
| `expr ? a : b` | Conditional (ternary) expression |
| `a in array` | Tests array membership (true if index `a` exists in `array`) |
| `a ~ /regex/` | Regular expression match (true if `a` matches the regex) |
| `a !~ /regex/` | Negation of regular expression match |
| `++` | Increment (works as both prefix `++a` and postfix `a++`) |
| `--` | Decrement (works as both prefix `--a` and postfix `a--`) |
| `$` | Field reference operator (e.g., `$1`, `$NF`) |
| *(blank / space)* | String **concatenation** operator (juxtaposition — no explicit symbol) |

**Example — Ternary operator:**
```bash
awk '{ print ($2 > 25 ? "Senior" : "Junior") }' data.txt
```

**Example — Nested ternary:**
```bash
awk 'BEGIN{ x=75; grade = (x>=90 ? "A" : x>=75 ? "B" : x>=50 ? "C" : "F"); print grade }'
```
Output:
```
B
```

**Example — Regex match operator `~`:**
```bash
awk '$1 ~ /^A/ { print $1, "starts with A" }' data.txt
```

**Example — Negated regex match `!~`:**
```bash
awk '$1 !~ /^A/ { print $1, "does NOT start with A" }' data.txt
```

**Example — Array membership `in`:**
```bash
awk 'BEGIN{ arr["x"]=1; if ("x" in arr) print "x exists" }'
```
Output:
```
x exists
```

**Example — Prefix vs postfix increment (a subtle but important difference):**
```bash
awk 'BEGIN{ i=5; print i++; print i }'    # Output: 5  then  6  (value used BEFORE increment)
awk 'BEGIN{ i=5; print ++i; print i }'    # Output: 6  then  6  (value used AFTER increment)
```

**Example — Field reference `$` with computed index:**
```bash
echo "a b c d" | awk '{ print $(NF-1) }'
```
Output:
```
c
```
*Explanation:* `$(NF-1)` refers to the **second-to-last** field.

**Example — Concatenation (blank operator):**
```bash
awk 'BEGIN{ a="Hello"; b="World"; print a " " b }'
```
Output:
```
Hello World
```
*Explanation:* There is no `+` for joining strings in AWK — simply placing two values next to each other (separated by a space, or nothing at all) concatenates them.

**Example — Concatenation without any separating space:**
```bash
awk 'BEGIN{ a="File"; b=".txt"; print a b }'
```
Output:
```
File.txt
```

### 7.6 Operator Precedence (High to Low — Simplified for Exam Purposes)

| Priority | Operator Category | Examples |
|---|---|---|
| Highest | Field reference | `$` |
| | Increment/decrement | `++`, `--` |
| | Exponentiation | `^`, `**` |
| | Unary plus/minus, NOT | `+x`, `-x`, `!x` |
| | Multiplication/Division/Modulus | `*`, `/`, `%` |
| | Addition/Subtraction | `+`, `-` |
| | String concatenation | *(blank)* |
| | Relational | `<`, `<=`, `>`, `>=`, `!=`, `==` |
| | Regex match | `~`, `!~` |
| | Array membership | `in` |
| | Logical AND | `&&` |
| | Logical OR | Double pipe |
| | Ternary | `?:` |
| Lowest | Assignment | `=`, `+=`, `-=`, ... |

---

## 8. Functions and Commands

AWK ships with many built-in functions, grouped by category below. Each group is expanded into its own explanation table **and** every single function gets at least one worked example, since the original course material lists them only by name.

### 8.1 Arithmetic Functions

| Function | Purpose |
|---|---|
| `atan2(y,x)` | Arctangent of `y/x`, returns radians |
| `cos(x)` | Cosine of `x` (radians) |
| `exp(x)` | Exponential, e^x |
| `int(x)` | Truncates `x` to an integer (toward zero) |
| `log(x)` | Natural logarithm of `x` |
| `rand()` | Random number between 0 (inclusive) and 1 (exclusive) |
| `sin(x)` | Sine of `x` (radians) |
| `sqrt(x)` | Square root of `x` |
| `srand()` | Seeds the random number generator (optionally takes a seed argument) |

**Individual examples:**
```bash
awk 'BEGIN{ print atan2(1,1) }'     # Output: 0.785398 (pi/4 radians)
awk 'BEGIN{ print cos(0) }'         # Output: 1
awk 'BEGIN{ print exp(1) }'         # Output: 2.71828 (Euler's number, e)
awk 'BEGIN{ print int(4.9) }'       # Output: 4
awk 'BEGIN{ print int(-4.9) }'      # Output: -4  (truncates toward zero, NOT floor)
awk 'BEGIN{ print log(1) }'         # Output: 0
awk 'BEGIN{ print sin(0) }'         # Output: 0
awk 'BEGIN{ print sqrt(16) }'       # Output: 4
awk 'BEGIN{ srand(42); print rand() }'   # Output: a repeatable pseudo-random number, seeded with 42
```

**Practical example — generating a random integer between 1 and 100:**
```bash
awk 'BEGIN{ srand(); print int(rand()*100)+1 }'
```
*Explanation:* `rand()` gives a fraction between 0 and 1; multiplying by 100 scales it up; `int()` truncates to a whole number; adding 1 shifts the range from `0–99` to `1–100`.

### 8.2 String Functions

| Function | Purpose |
|---|---|
| `asort(arr)` | Sorts array values (gawk extension) |
| `asorti(arr)` | Sorts array indices (gawk extension) |
| `gsub(re,rep,tgt)` | **Global** substitution of `re` with `rep` in `tgt` (all occurrences) |
| `index(s,t)` | Position of substring `t` inside `s` (1-based; `0` if not found) |
| `length(s)` | Length of string `s` (or `$0` if omitted) |
| `match(s,re)` | Tests if `s` contains a match for `re`; sets `RSTART`/`RLENGTH` |
| `split(s,arr,fs)` | Splits string `s` into array `arr` using separator `fs` |
| `sprintf(fmt,...)` | Returns a formatted string (like `printf` but returns instead of printing) |
| `sub(re,rep,tgt)` | Substitutes only the **first** match of `re` with `rep` in `tgt` |
| `strtonum(s)` | Converts string `s` to a number, understanding hex/octal prefixes (gawk extension) |
| `substr(s,m,n)` | Extracts substring of `s` starting at position `m`, length `n` (1-based indexing) |
| `tolower(s)` | Converts string to lowercase |
| `toupper(s)` | Converts string to uppercase |

**Individual examples for every string function:**

```bash
awk 'BEGIN{ n=split("a:b:c", parts, ":"); asort(parts); for(i=1;i<=n;i++) print parts[i] }'
```
*Explanation of `asort`:* sorts the **values** of array `parts` and re-indexes them numerically starting at 1.

```bash
awk 'BEGIN{ arr["z"]=1; arr["a"]=2; n=asorti(arr, sortedIdx); for(i=1;i<=n;i++) print sortedIdx[i] }'
```
Output:
```
a
z
```
*Explanation of `asorti`:* sorts the **indices** of the array instead of the values.

```bash
echo "foo bar foo baz" | awk '{ gsub(/foo/, "XXX"); print }'
```
Output:
```
XXX bar XXX baz
```
*Explanation of `gsub`:* replaces **every** occurrence of `foo` with `XXX` throughout `$0` (the default target if none is specified).

```bash
awk 'BEGIN{ print index("hello","ll") }'
```
Output:
```
3
```
*Explanation of `index`:* returns the starting position (1-based) of `"ll"` within `"hello"`.

```bash
awk 'BEGIN{ print length("hello") }'
```
Output:
```
5
```
*Explanation of `length`:* also works with no arguments, defaulting to `length($0)`, and can be applied to arrays in gawk (`length(arr)` returns element count).

```bash
awk 'BEGIN{ if (match("hello world", /wor/)) print "Matched at", RSTART, "length", RLENGTH }'
```
Output:
```
Matched at 7 length 3
```
*Explanation of `match`:* returns the position of the match (or `0` if no match) and additionally sets the global variables `RSTART` and `RLENGTH`.

```bash
awk 'BEGIN{ n=split("a:b:c", parts, ":"); for(i=1;i<=n;i++) print i, parts[i] }'
```
Output:
```
1 a
2 b
3 c
```
*Explanation of `split`:* breaks a string into an array using a given separator and returns the number of resulting elements.

```bash
awk 'BEGIN{ x=sprintf("%05d", 7); print x }'
```
Output:
```
00007
```
*Explanation of `sprintf`:* behaves exactly like `printf`, but instead of printing directly, it **returns** the formatted text as a string, which can be stored in a variable.

```bash
echo "foo bar foo baz" | awk '{ sub(/foo/, "XXX"); print }'
```
Output:
```
XXX bar foo baz
```
*Explanation of `sub`:* replaces **only the first** occurrence of `foo`, unlike `gsub` which replaces all of them.

```bash
awk 'BEGIN{ print strtonum("0x1A") }'
```
Output:
```
26
```
*Explanation of `strtonum`:* converts a string that looks like a number (including hex `0x` and octal `0` prefixes) into an actual numeric value.

```bash
awk 'BEGIN{ print substr("hello",2,3) }'
```
Output:
```
ell
```
*Explanation of `substr`:* extracts a substring starting at position `2`, of length `3`. If the length argument is omitted, it extracts to the end of the string.

```bash
awk 'BEGIN{ print substr("hello",2) }'
```
Output:
```
ello
```

```bash
awk 'BEGIN{ print tolower("ABC") }'   # Output: abc
awk 'BEGIN{ print toupper("abc") }'   # Output: ABC
```

**Worked example combining several string functions together (common exam-style question):**
```bash
echo "Hello World" | awk '{ print toupper($1), length($2), substr($2,1,3) }'
```
Output:
```
HELLO 5 Wor
```

### 8.3 Control Flow Statements

| Statement | Purpose |
|---|---|
| `break` | Exits the innermost loop immediately |
| `continue` | Skips to the next iteration of the innermost loop |
| `do ... while` | Executes a block **at least once**, then repeats while condition is true |
| `for` | Standard counting or array-traversal loop |
| `if / else` | Conditional branching |
| `return` | Returns a value from a user-defined function |
| `exit` | Terminates the program (optionally jumps straight to the `END` block) |

*(Full syntax and multiple worked examples for every one of these statements are provided in Section 10 — Loops and Conditionals — to avoid duplication.)*

**Example of `break`:**
```bash
awk 'BEGIN{ for(i=1;i<=10;i++){ if(i==5) break; print i } }'
```
Output:
```
1
2
3
4
```

**Example of `continue`:**
```bash
awk 'BEGIN{ for(i=1;i<=5;i++){ if(i==3) continue; print i } }'
```
Output:
```
1
2
4
5
```

**Example of `exit` with a status code:**
```bash
awk 'BEGIN{ print "Checking..."; exit 1 }'
echo "Exit status: $?"
```
Output:
```
Checking...
Exit status: 1
```

### 8.4 Input / Output Commands

| Command | Purpose |
|---|---|
| `close(file)` | Closes an open file or pipe (needed before re-reading a file within the same script) |
| `fflush()` | Flushes buffered output immediately |
| `getline` | Reads the next line of input into `$0` (or a variable) |
| `next` | Skips remaining code and moves to the **next record** immediately |
| `nextfile` | Skips remaining input in the current file and moves to the next file |
| `print` | Outputs data, using `OFS` and `ORS` |
| `printf` | Outputs formatted data (C-style formatting) |

> **Correction / typo fix:** The original source slide lists a command called *"nextline"*. The correct, standard AWK/POSIX command is **`nextfile`**, which skips the rest of the current input file and moves on to the next file. There is no built-in command literally named `nextline`; this has been corrected here for exam accuracy.

**Example of `getline` (reading an extra line manually):**
```bash
awk '{ print "Current:", $0; if ((getline line) > 0) print "Next:", line }' data.txt
```
*Explanation:* Inside the loop for the current record, `getline line` manually pulls in the **following** record early, storing it in the variable `line` without disturbing `$0`/`NR` in the usual way for the current cycle.

**Example of `getline` from a command (running a shell command and reading its output):**
```bash
awk 'BEGIN{ "date" | getline d; print "Today is", d }'
```

**Example of `next`:**
```bash
awk '{ if ($2 < 18) next; print $1, "is an adult" }' data.txt
```
*Explanation:* If the second field is less than 18, `next` immediately skips the rest of the block **and moves to the following record**, so the `print` statement never runs for that record.

**Example of `nextfile`:**
```bash
awk 'FNR==1 { print "First line of", FILENAME; nextfile }' file1.txt file2.txt
```
*Explanation:* Prints only the **first line** of each file, then jumps straight to the next file, skipping the rest of the current one entirely.

**Example of `close()`:**
```bash
awk 'BEGIN{
    print "line1" > "out.txt"
    print "line2" > "out.txt"
    close("out.txt")
    while ((getline l < "out.txt") > 0) print "Read back:", l
}'
```

**Example of `fflush()`:**
```bash
awk '{ print; fflush() }' data.txt
```
*Explanation:* Forces AWK to immediately write output rather than buffering it — useful in real-time monitoring scripts (e.g., watching a live-growing log file).

### 8.5 Programming Extensions

| Function/Statement | Purpose |
|---|---|
| `delete` | Removes an element (or the whole array) from an associative array |
| `function` | Declares a user-defined function |
| `system(cmd)` | Executes an external shell command from within AWK |

**Bit-wise functions (gawk extensions):**

| Function | Purpose |
|---|---|
| `and(a,b)` | Bit-wise AND |
| `compl(a)` | Bit-wise complement (NOT) |
| `lshift(a,n)` | Left shift `a` by `n` bits |
| `or(a,b)` | Bit-wise OR |
| `rshift(a,n)` | Right shift `a` by `n` bits |
| `xor(a,b)` | Bit-wise XOR |

**Example — `system()`:**
```bash
awk 'BEGIN{ system("date") }'
```

**Example — `system()` with a formatted command:**
```bash
awk 'BEGIN{ system("echo Hello from the shell") }'
```

**Example — `delete` (single element):**
```bash
awk 'BEGIN{ arr["a"]=1; arr["b"]=2; delete arr["a"]; for (k in arr) print k, arr[k] }'
```
Output:
```
b 2
```

**Example — `delete` (entire array, gawk extension):**
```bash
awk 'BEGIN{ arr["a"]=1; arr["b"]=2; delete arr; for (k in arr) print k; print "done" }'
```
Output:
```
done
```

**Examples of every bit-wise function:**
```bash
awk 'BEGIN{ print and(6,3) }'      # Output: 2   (0110 AND 0011 = 0010)
awk 'BEGIN{ print or(6,3) }'       # Output: 7   (0110 OR  0011 = 0111)
awk 'BEGIN{ print xor(6,3) }'      # Output: 5   (0110 XOR 0011 = 0101)
awk 'BEGIN{ print lshift(1,3) }'   # Output: 8   (1 shifted left by 3 = 1000)
awk 'BEGIN{ print rshift(8,3) }'   # Output: 1   (1000 shifted right by 3 = 1)
awk 'BEGIN{ print compl(0) }'      # Output: platform-dependent bit-complement of 0
```

---

## 9. Arrays in AWK

### 9.1 Concept

AWK provides **associative arrays** — arrays indexed by arbitrary strings (or numbers), not just sequential integers as in C-like languages.

### 9.2 Key Properties

| Property | Explanation |
|---|---|
| Associative | Every array in AWK is a key–value (associative) structure |
| Sparse storage | Only the indices you actually assign are stored — there are no "empty slots" to worry about |
| Non-integer index | The index does **not** need to be an integer — it can be any string |
| Assignment syntax | `arr[index] = value` |
| Iteration syntax | `for (var in arr) { ... }` |
| Deletion syntax | `delete arr[index]` |

### 9.3 Syntax Summary

```awk
arr[index] = value        # assign
for (var in arr) { ... }  # iterate over all indices
delete arr[index]         # remove one element
delete arr                # remove entire array (gawk extension)
```

### 9.4 Worked Examples

**Example — building and reading an array:**
```bash
awk 'BEGIN{
    fruit["apple"] = 3
    fruit["banana"] = 5
    fruit["cherry"] = 10
    for (name in fruit)
        print name, fruit[name]
}'
```
Possible output (order of associative arrays is not guaranteed):
```
apple 3
banana 5
cherry 10
```

**Example — checking membership before accessing an index (avoids accidentally creating an empty entry):**
```bash
awk 'BEGIN{
    arr["x"]=1
    if ("y" in arr) print "y exists"
    else print "y does not exist"
}'
```
Output:
```
y does not exist
```
*Explanation:* Testing with `if (arr["y"])` directly would actually **create** an empty entry for `"y"` as a side effect; using `"y" in arr` avoids this and is the correct, safe way to test membership.

**Example — counting word frequency using an array (very common exam question):**
```bash
awk '{ for (i=1; i<=NF; i++) count[$i]++ }
     END { for (word in count) print word, count[word] }' textfile.txt
```
*Explanation:* For every word (`$i`) in every record, the array `count` is incremented. At the end, all words and their frequencies are printed.

**Example — summing values grouped by a key (a mini "GROUP BY", extremely common in exams):**

Input `sales.txt`:
```
North 100
South 200
North 50
East 75
South 25
```

Command:
```bash
awk '{ total[$1] += $2 } END { for (region in total) print region, total[region] }' sales.txt
```
Output:
```
North 150
South 225
East 75
```

### 9.5 Multi-Dimensional (Simulated) Arrays

AWK does not have true multi-dimensional arrays, but it **simulates** them using `SUBSEP` to combine multiple indices into a single composite string index.

**Syntax:**
```awk
arr[i, j] = value       # internally becomes arr[i SUBSEP j] = value
```

**Example:**
```bash
awk 'BEGIN{
    matrix[1,1] = "a"
    matrix[1,2] = "b"
    matrix[2,1] = "c"
    matrix[2,2] = "d"
    for (i=1;i<=2;i++) {
        for (j=1;j<=2;j++)
            printf "%s ", matrix[i,j]
        print ""
    }
}'
```
Output:
```
a b
c d
```

**Example — checking membership in a simulated 2D array:**
```bash
awk 'BEGIN{
    matrix[1,1] = "x"
    if ((1,1) in matrix) print "exists"
}'
```
Output:
```
exists
```

**Example — iterating and splitting composite keys manually with `SUBSEP`:**
```bash
awk 'BEGIN{
    matrix[1,1] = "a"; matrix[2,3] = "b"
    for (key in matrix) {
        split(key, parts, SUBSEP)
        print "row:", parts[1], "col:", parts[2], "value:", matrix[key]
    }
}'
```

---

## 10. Loops and Conditionals

### 10.1 Concept

AWK supports the same fundamental control-flow constructs found in C-like languages, making it very easy to pick up for anyone who already knows C, Java, or Python's control-flow syntax.

### 10.2 Syntax of All Loop/Conditional Types

```awk
# for-in loop — iterate over array indices
for (a in array)
{
    print a
}

# for loop — counting loop
for (i=1; i<n; i++)
{
    print i
}

# if statement
if (a > b)
{
    print a
}

# while loop
while (a < n)
{
    print a
}

# do-while loop — executes body at least once
do
{
    print a
} while (a < n)
```

### 10.3 Explanation Table

| Construct | Runs Body | Typical Use |
|---|---|---|
| `for (a in array)` | Once per index in the array | Iterating over associative arrays |
| `for (i=1;i<n;i++)` | While the condition is true, with an update step each time | Counting loops, fixed number of iterations |
| `if / else` | Only if the condition is true | Conditional branching |
| `while` | While the condition is true (checked **before** each iteration) | Loops where the condition might be false at the start |
| `do ... while` | **At least once**, then while the condition remains true | Loops where the body must run at least one time |

### 10.4 Worked Examples for Every Construct

**Example — `if / else`:**
```bash
awk '{ if ($2 > 25) print $1, "Senior"; else print $1, "Junior" }' data.txt
```

**Example — `if / else if / else` chain:**
```bash
awk '{
    if ($2 >= 60) print $1, "Senior Citizen"
    else if ($2 >= 18) print $1, "Adult"
    else print $1, "Minor"
}' data.txt
```

**Example — `while` loop:**
```bash
awk 'BEGIN{ i=1; while (i<=5) { print i; i++ } }'
```
Output:
```
1
2
3
4
5
```

**Example — `do...while` loop:**
```bash
awk 'BEGIN{ i=1; do { print i; i++ } while (i<=5) }'
```
Output:
```
1
2
3
4
5
```

**Example — `do...while` running once even when the condition starts false (key distinguishing feature vs. `while`):**
```bash
awk 'BEGIN{ i=10; do { print "Runs at least once, i=" i } while (i<5) }'
```
Output:
```
Runs at least once, i=10
```

**Example — `for` loop (classic counting):**
```bash
awk 'BEGIN{ for (i=1;i<=5;i++) print i }'
```

**Example — `for` loop over fields:**
```bash
awk '{ for (i=1; i<=NF; i++) print "Field", i, "=", $i }' data.txt
```

**Example — `for (a in array)` loop:**
```bash
awk 'BEGIN{ arr["a"]=1; arr["b"]=2; for (k in arr) print k, arr[k] }'
```

**Example — nested loops (printing a multiplication table):**
```bash
awk 'BEGIN{
    for (i=1;i<=3;i++) {
        for (j=1;j<=3;j++)
            printf "%d ", i*j
        print ""
    }
}'
```
Output:
```
1 2 3
2 4 6
3 6 9
```

**Example — combining `break` inside nested loops:**
```bash
awk 'BEGIN{
    for (i=1;i<=3;i++) {
        for (j=1;j<=3;j++) {
            if (j==2) break
            print i, j
        }
    }
}'
```
Output:
```
1 1
2 1
3 1
```
*Explanation:* `break` only exits the **innermost** loop (the `j` loop), not the outer `i` loop.

### 10.5 The `switch` Statement (gawk Extension)

**Concept:** In addition to `if/else` chains, `gawk` supports a **C-style `switch` statement**, which can make multi-branch conditionals easier to read when checking one variable against several possible fixed values. This is **not** part of the original POSIX AWK standard — it is available only in `gawk` (run with `gawk --posix` disabled, which is the default).

**Syntax:**
```awk
switch (expression) {
    case value1:
        # statements
        break
    case value2:
        # statements
        break
    default:
        # statements
}
```

**Example:**
```bash
gawk 'BEGIN{
    day = "Tue"
    switch (day) {
        case "Mon":
            print "Start of the week"
            break
        case "Tue":
        case "Wed":
        case "Thu":
            print "Midweek"
            break
        case "Fri":
            print "Almost the weekend"
            break
        default:
            print "Weekend"
    }
}'
```
Output:
```
Midweek
```
*Explanation:* Since `day` is `"Tue"`, execution jumps to the `case "Tue":` label. Because there is no `break` immediately after `case "Tue":`, execution **falls through** into `case "Wed":` and `case "Thu":` as well (they share the same body) until it reaches a `break`. This "fall-through" behavior — grouping several `case` labels together to share one action — is a deliberate and commonly used C-style idiom.

**Note for exam purposes:** because `switch` is a `gawk`-only extension, if a question or environment specifies **strict POSIX AWK**, the safe and portable alternative is always an `if / else if / else` chain (see Section 10.4), which behaves identically on every AWK variant.

---

## 11. User-Defined Functions

### 11.1 Concept

Besides built-in functions, AWK allows programmers to define their **own** functions, which can be reused across scripts and even shared via separate "library" files.

### 11.2 Syntax

```awk
function function_name(parameters)
{
    # function body
    return value   # optional
}
```

**Important rules about parameters (frequently tested):**

- AWK functions can be called with **fewer** arguments than declared — the missing parameters are automatically treated as **local variables**, initialized to the empty/uninitialized value. This is actually how AWK simulates local variables, since AWK has no separate `local` keyword.

- **Scalars are passed by value** (a copy is made — changes inside the function do NOT affect the caller's variable).

- **Arrays are passed by reference** (changes made inside the function DO affect the caller's array).

### 11.3 Combining Multiple Script Files (a Library + a Main Script)

```bash
cat infile | awk -f mylib -f myscript.awk
```
*Explanation:* AWK allows multiple `-f` options, letting you keep reusable functions in one file (`mylib`) and the main logic in another (`myscript.awk`). AWK concatenates them logically before execution.

**Example — `mylib` (library file with function definitions):**
```awk
function myfunc1()
{
    printf "%s\n", $1
}

function myfunc2(a)
{
    return a * rand()
}
```

**Example — `myscript.awk` (main script that uses the library functions):**
```awk
BEGIN
{
    a = 1
}
{
    myfunc1()
    b = myfunc2(a)
    print b
}
```

*Explanation:*
- `myfunc1()` takes no parameters and prints the first field of the current record.
- `myfunc2(a)` takes one parameter `a`, multiplies it by a random number, and **returns** the result using `return`.
- In the main script, `BEGIN` initializes `a = 1`. Then, for every record, `myfunc1()` is called for its side effect (printing), and `myfunc2(a)` is called and its return value stored in `b`.

### 11.4 Additional Worked Examples

**Example — a simple standalone user-defined function:**
```bash
awk 'function square(x) { return x*x } BEGIN{ print square(5) }'
```
Output:
```
25
```

**Example — a function with multiple parameters:**
```bash
awk 'function add(a,b) { return a+b } BEGIN{ print add(3,4) }'
```
Output:
```
7
```

**Example — demonstrating pass-by-value for scalars (caller's variable is unaffected):**
```bash
awk '
function tryChange(x) { x = 999 }
BEGIN{ n = 5; tryChange(n); print n }
'
```
Output:
```
5
```
*Explanation:* Even though `tryChange` sets its local parameter `x` to `999`, the caller's variable `n` remains `5`, proving scalars are passed **by value**.

**Example — demonstrating pass-by-reference for arrays (caller's array IS affected):**
```bash
awk '
function fillArray(arr) { arr["key"] = "modified" }
BEGIN{ fillArray(myArr); print myArr["key"] }
'
```
Output:
```
modified
```
*Explanation:* Unlike scalars, arrays are passed **by reference** — any change made to the array parameter inside the function is visible to the caller.

**Example — using "extra" unfilled parameters as local variables (a classic AWK idiom):**
```bash
awk '
function factorial(n,   i, result) {
    result = 1
    for (i=1;i<=n;i++) result *= i
    return result
}
BEGIN{ print factorial(5) }
'
```
Output:
```
120
```
*Explanation:* The extra parameters `i` and `result` (separated from `n` by extra whitespace, purely as a **style convention**) are never passed a value by the caller, so AWK treats them as **local variables** private to this function call — this is the standard way to declare "local" variables in AWK, since there is no dedicated keyword for it.

**Example — recursive user-defined function:**
```bash
awk '
function fib(n) {
    if (n <= 1) return n
    return fib(n-1) + fib(n-2)
}
BEGIN{ for (i=0;i<10;i++) printf "%d ", fib(i); print "" }
'
```
Output:
```
0 1 1 2 3 5 8 13 21 34
```
*Explanation:* AWK functions can call themselves recursively, just like in most other programming languages.

---

## 12. Pretty Printing with printf

### 12.1 Concept

While `print` gives simple, quick output, `printf` gives **fine-grained, C-style formatted output** — essential for aligning columns, controlling decimal places, and producing clean, report-style output.

**Syntax:**
```awk
printf "format", a, b, c
```

The **format string** contains one or more format specifiers of the form:
```
%[modifier]control-letter
```

### 12.2 Format Specifiers (Control Letters)

| Control Letter | Meaning |
|---|---|
| `c` | ASCII character |
| `d` | Integer (decimal) |
| `i` | Integer (decimal, same as `d`) |
| `e` | Scientific notation |
| `f` | Floating-point notation |
| `g` | Shorter of scientific notation or floating notation |
| `o` | Octal value |
| `s` | String of text |
| `x` | Hexadecimal value (lowercase letters) |
| `X` | Hexadecimal value (uppercase letters) |

**Individual example for every control letter:**
```bash
awk 'BEGIN{ printf "%c\n", 65 }'          # Output: A   (ASCII code 65 = 'A')
awk 'BEGIN{ printf "%d\n", 42.9 }'        # Output: 42  (truncated, not rounded)
awk 'BEGIN{ printf "%i\n", 42.9 }'        # Output: 42
awk 'BEGIN{ printf "%e\n", 123456 }'      # Output: 1.234560e+05
awk 'BEGIN{ printf "%f\n", 3.14159 }'     # Output: 3.141590 (6 decimal places by default)
awk 'BEGIN{ printf "%g\n", 0.0000123 }'   # Output: 1.23e-05
awk 'BEGIN{ printf "%o\n", 8 }'           # Output: 10
awk 'BEGIN{ printf "%s\n", "hello" }'     # Output: hello
awk 'BEGIN{ printf "%x\n", 255 }'         # Output: ff
awk 'BEGIN{ printf "%X\n", 255 }'         # Output: FF
```

### 12.3 Width and Precision Modifiers

| Modifier | Meaning |
|---|---|
| `width` | Minimum field width (pads with spaces, or zeros if a `0` prefix is used) |
| `prec` (precision) | For floats: number of digits after the decimal point; for strings: maximum number of characters printed |

**Examples of width and precision:**
```bash
awk 'BEGIN{ printf "[%10d]\n", 42 }'       # Output: [        42]  (right-aligned, width 10)
awk 'BEGIN{ printf "[%-10d]\n", 42 }'      # Output: [42        ]  (left-aligned, width 10)
awk 'BEGIN{ printf "[%010d]\n", 42 }'      # Output: [0000000042]  (zero-padded, width 10)
awk 'BEGIN{ printf "[%.2f]\n", 3.14159 }'  # Output: [3.14]  (2 digits after decimal)
awk 'BEGIN{ printf "[%8.2f]\n", 3.14159 }' # Output: [    3.14]  (width 8, precision 2)
awk 'BEGIN{ printf "[%.3s]\n", "hello" }'  # Output: [hel]  (string truncated to 3 chars)
```

### 12.4 Combined Practical Examples

**Example — basic `printf`:**
```bash
awk 'BEGIN{ printf "%s is %d years old\n", "Alice", 25 }'
```
Output:
```
Alice is 25 years old
```

**Example — controlling width and precision together:**
```bash
awk 'BEGIN{ printf "%-10s%5.2f\n", "Price:", 3.14159 }'
```
Output:
```
Price:      3.14
```
*Explanation:* `%-10s` left-aligns the string in a 10-character-wide field; `%5.2f` right-aligns the float in a 5-character field with 2 digits after the decimal point.

**Example — hexadecimal and octal output together:**
```bash
awk 'BEGIN{ printf "Hex: %x, Octal: %o\n", 255, 8 }'
```
Output:
```
Hex: ff, Octal: 10
```

**Example — formatted table (a very common exam-style question):**
```bash
awk '{ printf "%-10s %5d\n", $1, $2 }' data.txt
```
*Explanation:* Prints the first field left-aligned in a 10-character column, and the second field right-aligned in a 5-character column — producing a neatly aligned table.

**Example — formatted multi-column report with a header:**
```bash
awk 'BEGIN{ printf "%-10s %-10s %6s\n", "Name", "Role", "Age" }
     { printf "%-10s %-10s %6d\n", $1, $3, $2 }' data.txt
```
Output (example):
```
Name       Role         Age
Alice      Engineer      25
Bob        Doctor        30
Charlie    Artist        22
```

---

## 13. Combining Bash and AWK

### 13.1 Concept

AWK is rarely used in complete isolation — it is most powerful when combined with shell scripting.

### 13.2 Common Integration Techniques

| Technique | Description |
|---|---|
| Including AWK inside a shell script | An AWK command or script is embedded directly inside a `.sh` file |
| Heredoc feature | Multi-line AWK programs are embedded in bash using `<<` heredoc syntax, avoiding the need for a separate `.awk` file |
| Piping with other commands | AWK is combined with other shell utilities (`grep`, `sort`, `cut`, etc.) on the command line using the pipe operator |

### 13.3 Worked Examples

**Example — AWK inside a bash script using heredoc:**
```bash
#!/bin/bash
awk << 'EOF'
BEGIN { print "Report generated from heredoc" }
EOF
```

**Example — heredoc processing a real file, with a shell variable passed in via `-v`:**
```bash
#!/bin/bash
THRESHOLD=25
awk -v t="$THRESHOLD" << 'EOF' data.txt
$2 > t { print $1, "is above threshold" }
EOF
```

**Example — chaining AWK with other Unix tools:**
```bash
cat access.log | grep "ERROR" | awk '{ print $1, $NF }' | sort
```
*Explanation:* `grep` filters only lines containing "ERROR", `awk` extracts the first field and the last field (`$NF` = last field of the record), and `sort` orders the final output.

**Example — using AWK's output as input to another command (`uniq -c` style counting):**
```bash
awk '{ print $3 }' data.txt | sort | uniq -c
```
*Explanation:* Extracts the third field from every record, sorts the results, and `uniq -c` counts how many times each unique value appears — a very common pattern for quick frequency reports.

**Example — a full mini bash script combining several ideas from this document:**
```bash
#!/bin/bash
# report.sh - generate a simple summary report

INPUT="data.txt"

awk -v file="$INPUT" '
BEGIN {
    FS = " "
    print "===== Report for", file, "====="
}
{
    total_age += $2
    count++
}
END {
    printf "Average age: %.2f\n", total_age/count
    printf "Total records: %d\n", count
}
' "$INPUT"
```

### 13.4 Useful Companion Command-Line Tools

AWK is frequently used **alongside** other standard Unix command-line tools rather than as a total replacement for them. This section covers several such companion tools, showing how their output can either be piped into AWK, or generated from within an AWK script using `system()` or `getline`.

#### Reading a Shell Command's Output Directly Into an AWK Variable (`cmd | getline var`)

**Syntax:**
```awk
"command" | getline var
```
*Explanation:* This runs `command` as a subprocess, and reads **one line** of its output into `var`, without disturbing the current record's `$0`/`NF`. This is the standard way to pull external, dynamically generated data (like the current date, or a network lookup) into an AWK script.

**Example:**
```bash
awk 'BEGIN{ "whoami" | getline user; print "Running as:", user }'
```

#### Computing a Past or Future Date with `date --date`

The `date` command supports flexible relative-date arithmetic via its `--date` option, which is commonly combined with AWK using `system()` or `cmd | getline`.

**Example — getting the date 5 days ago, formatted as `DD/MM/YYYY`:**
```bash
date --date="5 days ago" +%d/%m/%Y
```
Example output:
```
23/08/2026
```

**Example — pulling that computed date into an AWK script:**
```bash
awk 'BEGIN{
    "date --date=\"5 days ago\" +%d/%m/%Y" | getline fiveDaysAgo
    print "Five days ago was:", fiveDaysAgo
}'
```
*Explanation:* This is a practical example of combining `getline` with a shell command that performs date arithmetic — useful for report scripts that need to reference a relative date (e.g., "show all log entries from the last 5 days").

#### Sorting AWK's Output with `sort`

AWK itself has no built-in "sort by column" statement for records, so its output is very commonly piped into the external `sort` command.

| Option | Meaning |
|---|---|
| `-n` | Sort **numerically** rather than lexicographically (so `9` sorts before `10`) |
| `-r` | Sort in **reverse** (descending) order |

**Example — numeric sort:**
```bash
awk '{ print $2 }' data.txt | sort -n
```

**Example — numeric, reverse (largest first) sort:**
```bash
awk '{ print $2 }' data.txt | sort -nr
```
*Explanation:* Combining `-n` and `-r` sorts numerically and then reverses the order, giving the **largest values first** — a very common pattern for "top N" style reports (e.g., highest sales, oldest employees).

#### Looking Up Domain/IP Information with `dig`

`dig` is a standard DNS lookup utility. While not part of AWK itself, its output is often **filtered and reformatted using AWK** as part of network-diagnostic scripts.

**Example — getting the IP address of a domain:**
```bash
dig example.com
```
*Explanation:* Returns a full, detailed DNS response, including the domain's associated IP address(es) among other information.

**Example — reverse lookup: getting the domain name from an IP address, using `-x`:**
```bash
dig -x 8.8.8.8
```
*Explanation:* The `-x` flag performs a **reverse DNS lookup**, converting an IP address back into its associated hostname.

**Example — a clean, one-line answer only, using `+noall +answer`:**
```bash
dig +noall +answer -x 8.8.8.8
```
*Explanation:* By default, `dig` prints a large amount of diagnostic output (query header, question section, timing statistics, etc.). Adding `+noall` suppresses **all** default output sections, and `+answer` then re-enables **only** the answer section — giving a clean, single-line result that is much easier to pipe directly into AWK for further field extraction, e.g.:
```bash
dig +noall +answer -x 8.8.8.8 | awk '{ print $NF }'
```
*Explanation:* This extracts just the resolved hostname (the last field of the one-line `dig` answer) — a practical example of AWK acting as the final formatting stage in a small DNS-lookup pipeline.

---

## 14. Why AWK Matters

### 14.1 Closing Remark from the Source Material

> "AWK is available everywhere! AWK is a programming language, quick to code and fast in execution. Combine it on the command line with other scripts."

### 14.2 Key Exam Takeaways From This Statement

| Point | Explanation |
|---|---|
| Universally available | AWK (or a variant like `gawk`/`mawk`) is installed by default on virtually every Unix/Linux system |
| Quick to code | Its concise pattern–action syntax means small text-processing tasks can be written in a single line |
| Fast in execution | AWK is implemented efficiently and is well-suited for processing large text files quickly |
| Composable | It integrates smoothly with shell pipelines and other command-line tools |

### 14.3 When to Choose AWK vs. Other Tools

| Task | Best Tool |
|---|---|
| Simple line filtering by pattern | `grep` |
| Simple find-and-replace | `sed` |
| Field-based extraction, arithmetic, reports | **AWK** |
| Complex data structures, external libraries, web requests | Python / Perl |
| Sorting / counting unique values | `sort`, `uniq` (often combined with AWK) |

---

## 15. Working with Large Files and Real-World Log Processing

### 15.1 Concept

One of AWK's most important practical strengths — beyond its concise syntax — is its ability to process **very large text files efficiently**. This section covers why that matters and demonstrates it with a realistic example.

### 15.2 Processing a File With Millions of Lines

AWK reads input **one record at a time**, using a small, constant amount of memory (unless the script itself deliberately accumulates large arrays). This means AWK can comfortably process files containing **millions of lines** — multi-gigabyte log files, for example — without ever needing to load the entire file into memory at once.

**Example — counting the number of lines in a very large file:**
```bash
awk 'END{ print NR }' huge_file.txt
```
*Explanation:* Even if `huge_file.txt` is several gigabytes in size, this command completes quickly and uses very little memory, because AWK never holds more than the current record in memory at any given time (the `END` block simply reports the final value of `NR` after the last record has been read).

**Example — summing a numeric column across millions of records:**
```bash
awk '{ sum += $3 } END{ print "Total:", sum }' huge_sales.txt
```

### 15.3 Why Spreadsheet Applications Struggle With Very Large Files

Spreadsheet applications (such as Excel or Google Sheets) are designed around loading an **entire dataset into memory at once**, and typically enforce hard row limits (for example, around one million rows in many spreadsheet programs). Beyond that limit — or even well before it, due to memory and performance constraints — spreadsheet applications become slow, unresponsive, or simply refuse to open the file at all.

| Aspect | Spreadsheet Application | AWK |
|---|---|---|
| Memory usage | Loads the entire file into memory | Processes one record at a time (streaming) |
| Row limits | Often has a hard maximum row count | No inherent row limit |
| Performance on huge files | Degrades significantly, or fails to open | Remains fast and lightweight |
| Automation / scripting | Manual, GUI-driven | Fully scriptable from the command line |

*Explanation:* Because AWK **streams** through a file rather than loading it wholesale, it can process files that would be completely impractical — or outright impossible — to open in a spreadsheet program, making it the preferred tool for large-scale log analysis, data extraction, and preprocessing before the data is ever brought into a smaller tool like a spreadsheet.

### 15.4 Practical Example — Processing a Web Server Log File

Web server log files (e.g., Apache or Nginx access logs) are among the most common real-world use cases for AWK, since each log line is naturally a **record** with clearly delimited **fields** (IP address, timestamp, request, status code, etc.).

Example log line format (`access.log`):
```
192.168.1.10 - - [28/Aug/2026:10:15:32] "GET /index.html HTTP/1.1" 200 1024
203.0.113.5 - - [28/Aug/2026:10:15:45] "GET /about.html HTTP/1.1" 404 512
192.168.1.10 - - [28/Aug/2026:10:16:02] "POST /login HTTP/1.1" 200 256
```

**Example — extracting just the IP address (the first field) of every request:**
```bash
awk '{ print $1 }' access.log
```

**Example — counting how many requests came from each unique IP address:**
```bash
awk '{ count[$1]++ } END{ for (ip in count) print ip, count[ip] }' access.log
```
Output (example):
```
192.168.1.10 2
203.0.113.5 1
```

**Example — printing only requests that resulted in an error (HTTP status code 404):**
```bash
awk '$9 == 404 { print $1, $7 }' access.log
```
*Explanation:* `$9` refers to the HTTP status code field, and `$7` refers to the requested resource path, in this particular log format — printing the client IP and the path that produced a `404 Not Found` error.

**Example — combining AWK with `sort` and `uniq` for a "top requesters" report:**
```bash
awk '{ print $1 }' access.log | sort | uniq -c | sort -nr | head -5
```
*Explanation:* Extracts every IP address, sorts them so identical values are adjacent, counts occurrences with `uniq -c`, sorts numerically in descending order, and shows only the top 5 — a complete, realistic pipeline for identifying the busiest clients hitting a server.

---

## 16. Full Worked Example — Payroll Management System

### 16.1 Concept and Purpose

To bring together everything covered in this document — fields, records, built-in variables, arrays, user-defined functions, control flow, and `printf` formatting — this section walks through a **single, complete, realistic AWK program**: a simple payroll management script that reads employee data from a text file and produces a formatted salary report.

### 16.2 Input Data

File `employees.txt` (fields: **Name, Department, Hours Worked, Hourly Rate**, space-separated):
```
Alice Engineering 160 25
Bob Sales 150 20
Charlie Engineering 170 30
Diana HR 140 18
Eve Sales 165 22
```

### 16.3 The Complete Script

File `payroll.awk`:
```awk
#!/usr/bin/gawk -f
#
# payroll.awk
# A simple Payroll Management System built entirely in AWK.
# Reads: Name, Department, Hours Worked, Hourly Rate
# Produces: a formatted payslip line per employee, overtime flags,
#           and a final department-wise summary report.

# ---- User-defined function: computes gross pay, applying ----
# ---- a 1.5x overtime multiplier for any hours beyond 150  ----
function computePay(hours, rate,   normalHours, otHours, pay) {
    if (hours > 150) {
        normalHours = 150
        otHours = hours - 150
    } else {
        normalHours = hours
        otHours = 0
    }
    pay = (normalHours * rate) + (otHours * rate * 1.5)
    return pay
}

BEGIN {
    FS = " "
    OFS = "\t"
    print "===================================================="
    print "             PAYROLL MANAGEMENT REPORT"
    print "===================================================="
    printf "%-10s %-12s %6s %8s %10s\n", "Name", "Dept", "Hours", "Rate", "Gross Pay"
    print "----------------------------------------------------"
}

# Skip any accidental blank lines in the input file
NF == 0 { next }

{
    name   = $1
    dept   = $2
    hours  = $3
    rate   = $4

    gross = computePay(hours, rate)

    printf "%-10s %-12s %6d %8.2f %10.2f\n", name, dept, hours, rate, gross

    # Accumulate department-wise totals for the summary at the end
    deptTotal[dept] += gross
    deptCount[dept]++
    grandTotal += gross

    if (hours > 150)
        overtimeCount++
}

END {
    print "----------------------------------------------------"
    printf "Total employees processed: %d\n", NR
    printf "Employees with overtime:   %d\n", overtimeCount
    print ""
    print "---------------- Department Summary ----------------"
    for (d in deptTotal)
        printf "%-15s Employees: %-3d  Total Pay: %10.2f\n", d, deptCount[d], deptTotal[d]
    print "------------------------------------------------------"
    printf "GRAND TOTAL PAYROLL: %.2f\n", grandTotal
    print "===================================================="
}
```

### 16.4 Running the Script

```bash
chmod +x payroll.awk
./payroll.awk employees.txt
```

or equivalently:
```bash
awk -f payroll.awk employees.txt
```

### 16.5 Expected Output

```
====================================================
             PAYROLL MANAGEMENT REPORT
====================================================
Name       Dept          Hours     Rate  Gross Pay
----------------------------------------------------
Alice      Engineering      160    25.00    4125.00
Bob        Sales            150    20.00    3000.00
Charlie    Engineering      170    30.00    5400.00
Diana      HR               140    18.00    2520.00
Eve        Sales            165    22.00    3795.00
----------------------------------------------------
Total employees processed: 5
Employees with overtime:   3

---------------- Department Summary ----------------
Engineering     Employees: 2    Total Pay:    9525.00
Sales           Employees: 2    Total Pay:    6795.00
HR              Employees: 1    Total Pay:    2520.00
------------------------------------------------------
GRAND TOTAL PAYROLL: 18840.00
====================================================
```

### 16.6 Line-by-Line Explanation of Every Concept Used

| Concept Used | Where It Appears |
|---|---|
| Comments (`#`) | Throughout, documenting the script's purpose |
| Shebang line | `#!/usr/bin/gawk -f`, making the script directly executable |
| User-defined function with local variables | `computePay(hours, rate,   normalHours, otHours, pay)` — the extra parameters after the blank gap are local variables |
| `BEGIN` block | Sets `FS`, `OFS`, and prints the report header once |
| Pattern with `next` | `NF == 0 { next }` skips blank lines entirely |
| Field references (`$1`–`$4`) | Extracting name, department, hours, and rate from each record |
| Associative arrays | `deptTotal[dept]`, `deptCount[dept]` — grouping and summing by department |
| Arithmetic and assignment operators | `+=`, `*`, comparisons (`>`) inside `computePay` |
| `printf` formatting | Every report line uses width/precision specifiers (`%-10s`, `%6d`, `%10.2f`, etc.) |
| `if / else` | Overtime calculation logic inside `computePay` |
| `for (var in array)` loop | Iterating over `deptTotal` in the `END` block to print the summary |
| `END` block | Final grand-total calculation and formatted summary report |
| Built-in variable `NR` | Used in the `END` block to report the total number of employees processed |

*Explanation:* This single script demonstrates that AWK is fully capable of acting as a genuine small-scale **data-processing application** — not just a one-line text filter — by combining input parsing, business logic (overtime calculation), data aggregation (department totals via arrays), and professional, formatted reporting (via `printf`), all within one self-contained, well-commented file.

---

## 17. Summary

- **AWK** is a POSIX-standard programming language (IEEE 1003.1-2008) created by **A**ho, **W**einberger, and **K**ernighan, designed to process text organized into **records** and **fields**.

- Its **execution model** automatically splits input into records (default: lines, via `RS`) and fields (default: whitespace-separated words, via `FS`), and runs matching pattern–action blocks on each record — both `RS` and `FS` can be customized, including with regular expressions.

- AWK can be invoked **directly on the command line** (`awk -F"..." '{...}'`, with `-v` for variable injection) or as a **standalone script** with a shebang line (`#!/usr/bin/gawk -f`).

- A rich set of **built-in variables** (`NR`, `FNR`, `NF`, `FS`, `OFS`, `RS`, `ORS`, `$0`, `$n`, `ARGC`/`ARGV`, `ENVIRON`, `FILENAME`, `RSTART`/`RLENGTH`, `SUBSEP`, `OFMT`) describe the current parsing state without needing manual bookkeeping — each was demonstrated individually above with its own working example.

- Scripts are structured as **pattern { action }** pairs, with special blocks `BEGIN` (runs once before input) and `END` (runs once after input) for setup and summary tasks; multiple `BEGIN`/`END` blocks are allowed and run in order.

- AWK provides a full set of **assignment, logical, arithmetic, relational,** and **special operators** (ternary `?:`, array membership `in`, regex match `~`/`!~`, increment/decrement, field reference `$`, and string concatenation by juxtaposition), with well-defined precedence and important string-vs-numeric comparison rules.

- A large **standard library of functions** is available: arithmetic (`sqrt`, `int`, `rand`, `sin`, `cos`, `log`, `exp`, `atan2`, `srand`), string (`substr`, `split`, `gsub`, `sub`, `length`, `index`, `match`, `sprintf`, `tolower`, `toupper`, `strtonum`, `asort`, `asorti`), control flow (`if`, `for`, `while`, `do-while`, `break`, `continue`, `return`, `exit`), I/O (`print`, `printf`, `getline`, `next`, `nextfile`, `close`, `fflush`), and extensions (`delete`, `function`, `system`, and the bit-wise functions `and`, `or`, `xor`, `lshift`, `rshift`, `compl`).

- **Arrays** in AWK are **associative** and **sparse**, allowing any string as an index, with `for (var in array)` for iteration, `delete` for removal, and `SUBSEP`-based composite keys to simulate multi-dimensional arrays.

- All standard **loop constructs** (`for`, `for-in`, `while`, `do-while`) and **conditionals** (`if/else`, `if/else if/else`) are supported, mirroring C-like syntax, including `break` and `continue` for fine-grained loop control.

- **User-defined functions** let programmers build reusable logic — scalars are passed **by value**, arrays are passed **by reference**, extra unfilled parameters serve as **local variables**, and functions can be **recursive**. Functions can even be split across multiple files and combined using multiple `-f` flags.

- **`printf`** enables precise, C-style formatted output using control letters (`d`, `i`, `s`, `f`, `e`, `g`, `o`, `x`, `X`, `c`) along with width and precision modifiers — essential for producing clean, tabular reports.

- AWK integrates naturally with **bash**, either embedded directly in shell scripts (including via heredocs, with shell variables passed through `-v`) or chained with other Unix tools using pipes (`grep`, `sort`, `uniq`, etc.).

- AWK scripts can be documented with **comments** (`#`), support a `gawk`-only **`switch` statement** as an alternative to long `if/else` chains, and can be paired with companion command-line tools like `date --date`, `sort -n`/`-r`, and `dig` for real-world tasks such as DNS lookups and relative-date computation.

- AWK is well suited to **very large files** (millions of lines) because it streams records one at a time with constant memory usage — a capability that spreadsheet applications, which load entire datasets into memory and enforce row limits, cannot match — making AWK the standard tool for tasks like web server log analysis.

- The **Payroll Management System** example (Section 16) demonstrates all of these concepts working together in a single, realistic, well-commented program: field extraction, a user-defined function with local variables and overtime logic, associative arrays for department-wise grouping, `printf`-formatted reporting, and a summary `END` block.

- Overall, AWK's greatest strengths are that it is **universally available**, **quick to write**, and **fast to execute**, making it a core tool for text processing on any Unix-like system — and the right tool to reach for whenever a task involves extracting, transforming, or summarizing field-based text data.

---

*End of Notes — Week 9, AWK Programming, Lecture 1 & 2 (Part 1 & Part 2), Expanded Edition*
