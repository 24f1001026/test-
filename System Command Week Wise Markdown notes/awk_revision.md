# 📘 Complete Revision Guide: `awk`

> A beginner-friendly, detailed guide covering everything about `awk` — with examples, explanations, practice problems, and solutions.

---

## Table of Contents

1. [What is `awk`?](#1-what-is-awk)
2. [Basic Syntax](#2-basic-syntax)
3. [How awk Works (Concept)](#3-how-awk-works-concept)
4. [Sample File Used in Examples](#4-sample-file-used-in-examples)
5. [Fields and Records (The Core Idea)](#5-fields-and-records-the-core-idea)
6. [Built-in Variables](#6-built-in-variables)
7. [Patterns (Choosing WHICH lines to act on)](#7-patterns-choosing-which-lines-to-act-on)
8. [Actions — What awk Does with Matching Lines](#8-actions--what-awk-does-with-matching-lines)
9. [BEGIN and END Blocks](#9-begin-and-end-blocks)
10. [Print and Printf](#10-print-and-printf)
11. [Operators in awk](#11-operators-in-awk)
12. [Variables — User Defined](#12-variables--user-defined)
13. [Control Flow: if / else, loops](#13-control-flow-if--else-loops)
14. [Built-in Functions](#14-built-in-functions)
15. [Arrays in awk](#15-arrays-in-awk)
16. [Field Separator Options (`-F`, `FS`, `OFS`)](#16-field-separator-options--f-fs-ofs)
17. [Using awk Scripts (`-f`)](#17-using-awk-scripts--f)
18. [Practice Problems with Solutions](#18-practice-problems-with-solutions)
19. [Common Mistakes Beginners Make](#19-common-mistakes-beginners-make)
20. [Quick Cheat Sheet](#20-quick-cheat-sheet)
21. [Summary](#21-summary)

---

## 1. What is `awk`?

**awk** is a **pattern scanning and text-processing programming language** built into Linux/Unix systems. Its name comes from the initials of its three creators: **A**ho, **W**einberger, and **K**ernighan.

Unlike `grep` (which only searches) or `sed` (which mainly edits text), **awk is a mini programming language** — it can search, filter, calculate, format, and transform data, especially when data is organized in **columns/fields** (like CSV files, log files, or command output).

### Why do we use awk?
- To process and analyze **column-based data** (like `.csv` files or `ls -l` output).
- To perform **calculations** (sum, average, count) directly from the command line.
- To **print specific columns** from a file.
- To **filter rows based on conditions** (like `WHERE` in SQL).
- To **reformat** text output.

> 🧠 Think of awk like **Excel for the terminal** — it understands rows and columns, and you can write small "formulas" (patterns + actions) to work with that data.

---

## 2. Basic Syntax

```bash
awk 'PATTERN { ACTION }' FILENAME
```

- **PATTERN** → condition that decides which lines to process (optional — if omitted, action applies to all lines)
- **ACTION** → what to do with matching lines (inside `{ }`)
- **FILENAME** → the file to process

### Simplest Example

```bash
awk '{print}' file.txt
```
👉 No pattern given (so applies to all lines), and the action `print` prints every line — similar to `cat file.txt`.

---

## 3. How awk Works (Concept)

1. awk reads the file **line by line**. Each line is called a **record**.
2. Each record is automatically **split into fields** (columns) based on a separator (default: whitespace).
3. For every record, awk checks: *"Does this record match my PATTERN?"*
4. If it matches (or no pattern is given) → awk executes the **ACTION**.
5. This repeats for every line until the file ends.
6. Special **BEGIN** and **END** blocks run once — before processing starts and after it finishes.

---

## 4. Sample File Used in Examples

Let's create one file to use throughout this guide.

**File name:** `students.txt`

```
Ravi 25 Delhi Passed
Sonia 22 Mumbai Failed
Aman 28 Delhi Passed
Neha 21 Pune Passed
ravi 30 Chennai Failed
Karan 19 Delhi Failed
Priya 24 Mumbai Passed
```

Create it yourself:
```bash
cat > students.txt << EOF
Ravi 25 Delhi Passed
Sonia 22 Mumbai Failed
Aman 28 Delhi Passed
Neha 21 Pune Passed
ravi 30 Chennai Failed
Karan 19 Delhi Failed
Priya 24 Mumbai Passed
EOF
```

> Note: This file uses **spaces** as separators (Name, Age, City, Status) — perfect for demonstrating awk's default field-splitting behavior.

---

## 5. Fields and Records (The Core Idea)

This is the **most important concept** in awk.

- Each **line** of a file = one **record**.
- Each **word/column** in that line (split by spaces by default) = a **field**.
- Fields are numbered: `$1`, `$2`, `$3`, ... and `$0` means **the whole line**.

### Example
For the line:
```
Ravi 25 Delhi Passed
```
| Field | Value |
|-------|-------|
| `$0` | Ravi 25 Delhi Passed (entire line) |
| `$1` | Ravi |
| `$2` | 25 |
| `$3` | Delhi |
| `$4` | Passed |

### Example Command
```bash
awk '{print $1, $3}' students.txt
```
**Output:**
```
Ravi Delhi
Sonia Mumbai
Aman Delhi
Neha Pune
ravi Chennai
Karan Delhi
Priya Mumbai
```
👉 Prints only the Name (`$1`) and City (`$3`) columns.

---

## 6. Built-in Variables

awk provides several **ready-made variables** that give useful information automatically.

| Variable | Meaning |
|----------|---------|
| `$0` | The entire current line |
| `$1, $2, ...` | Individual fields (columns) |
| `NF` | Number of Fields in the current line |
| `NR` | Number of Records (current line number) processed so far |
| `FNR` | Line number relative to current file (resets per file when using multiple files) |
| `FS` | Field Separator (default = space/tab) |
| `OFS` | Output Field Separator (default = space) |
| `RS` | Record Separator (default = newline) |
| `ORS` | Output Record Separator (default = newline) |
| `FILENAME` | Name of the current input file |

### Examples

```bash
awk '{print NR, $0}' students.txt
```
👉 Prints line number before each line (like `cat -n`).

```bash
awk '{print NF}' students.txt
```
👉 Prints the number of fields (columns) in each line — here it will print `4` for every line.

```bash
awk '{print $NF}' students.txt
```
👉 Prints the **last field** of each line (here: Passed/Failed) — `$NF` means "field number NF", i.e., the last one.

---

## 7. Patterns (Choosing WHICH lines to act on)

Patterns decide **which lines** the action should run on. If no pattern is given, the action applies to **every line**.

| Pattern Type | Example | Meaning |
|---------------|---------|---------|
| No pattern | `{print}` | applies to all lines |
| Regex match | `/Delhi/{print}` | lines containing "Delhi" |
| Comparison | `$2 > 25 {print}` | lines where field 2 (Age) > 25 |
| Combined (AND) | `$3=="Delhi" && $4=="Passed"` | both conditions true |
| Combined (OR) | `$3=="Delhi" \|\| $3=="Pune"` | either condition true |
| NOT | `!/Failed/` | lines NOT containing "Failed" |
| Line number range | `NR==2,NR==4` | lines 2 through 4 |

### Examples

```bash
awk '/Delhi/' students.txt
```
👉 Prints full lines containing "Delhi" (default action is `print` when action is omitted).

```bash
awk '$2 > 24' students.txt
```
👉 Prints lines where Age (`$2`) is greater than 24.

```bash
awk '$3=="Delhi" && $4=="Passed"' students.txt
```
👉 Prints lines where City is Delhi **AND** Status is Passed.

```bash
awk 'NR==2,NR==4' students.txt
```
👉 Prints lines 2 to 4 (range using record number).

---

## 8. Actions — What awk Does with Matching Lines

Actions are written inside `{ }` and tell awk **what to do** with matching records.

```bash
awk '$4=="Failed" {print $1, "failed the exam"}' students.txt
```
**Output:**
```
Sonia failed the exam
ravi failed the exam
Karan failed the exam
```
👉 Custom formatted output combining field values with plain text.

You can have **multiple statements** inside `{ }`, separated by `;`:
```bash
awk '{count++; print $1}' students.txt
```

---

## 9. BEGIN and END Blocks

- **BEGIN** → runs **once**, **before** any line is read. Used for initialization, printing headers.
- **END** → runs **once**, **after** all lines are processed. Used for totals, summaries.

### Example
```bash
awk 'BEGIN {print "Name List:"} {print $1} END {print "--- End of List ---"}' students.txt
```
**Output:**
```
Name List:
Ravi
Sonia
Aman
Neha
ravi
Karan
Priya
--- End of List ---
```

### Practical Example — Counting Lines
```bash
awk 'END {print "Total students:", NR}' students.txt
```
👉 `NR` at the END block holds the **total number of lines** processed.

---

## 10. Print and Printf

### 10.1 `print` — simple output
```bash
awk '{print $1, $4}' students.txt
```
👉 Comma in `print` inserts the **OFS** (default: single space) between values.

### 10.2 `printf` — formatted output (like C language)
Gives you full control over spacing, decimal points, alignment.

```bash
awk '{printf "%-10s %-5s %s\n", $1, $2, $3}' students.txt
```
**Output (nicely aligned columns):**
```
Ravi       25    Delhi
Sonia      22    Mumbai
Aman       28    Delhi
...
```

**Common format specifiers:**
| Specifier | Meaning |
|-----------|---------|
| `%s` | string |
| `%d` | integer |
| `%f` | floating-point number |
| `%-10s` | left-align string in 10-character width |
| `%5d` | right-align integer in 5-character width |
| `\n` | newline (printf does NOT add newline automatically, unlike print) |

> ⚠️ Important: `printf` does **not** auto-add a newline — you must add `\n` yourself, or all output will run together on one line.

---

## 11. Operators in awk

| Category | Operators |
|----------|-----------|
| Arithmetic | `+  -  *  /  %  ^` (power) |
| Comparison | `==  !=  <  >  <=  >=` |
| Logical | `&&` (AND), `\|\|` (OR), `!` (NOT) |
| String concatenation | just place values next to each other, e.g. `$1 $2` |
| Regex match | `~` (matches), `!~` (does not match) |
| Assignment | `=  +=  -=  *=  /=` |
| Increment/Decrement | `++`, `--` |

### Examples

```bash
awk '$2 % 2 == 0 {print $1, "has even age"}' students.txt
```
👉 Prints names where Age is an even number.

```bash
awk '$1 ~ /^R/ {print}' students.txt
```
👉 Prints lines where field 1 (Name) **matches** the regex `^R` (starts with R).

```bash
awk '$1 !~ /^R/ {print}' students.txt
```
👉 Prints lines where Name does **NOT** start with R.

```bash
awk '{print $1 " is from " $3}' students.txt
```
👉 String concatenation — joins values with plain text (no comma used, so no automatic space; spaces are added manually with `" "`).

---

## 12. Variables — User Defined

You can create your own variables inside awk, just like in any programming language (no need to declare type).

```bash
awk '{total = $2 + 10; print $1, total}' students.txt
```
👉 Creates a variable `total`, adds 10 to Age, and prints it.

You can also pass variables from the shell into awk using `-v`:
```bash
awk -v bonus=5 '{print $1, $2+bonus}' students.txt
```
👉 `bonus` becomes usable inside the awk program.

---

## 13. Control Flow: if / else, loops

awk supports real programming constructs.

### 13.1 if / else
```bash
awk '{if ($4 == "Passed") print $1, "PASS"; else print $1, "FAIL"}' students.txt
```

### 13.2 for loop
```bash
awk '{for(i=1; i<=NF; i++) print $i}' students.txt
```
👉 Loops through **every field** in each line and prints it separately.

### 13.3 while loop
```bash
awk '{i=1; while (i<=NF) {print $i; i++}}' students.txt
```
👉 Same result as above, using `while` instead of `for`.

---

## 14. Built-in Functions

awk has many useful **ready-made functions**.

| Function | Meaning | Example |
|----------|---------|---------|
| `length($1)` | length of a string | `awk '{print length($1)}' file` |
| `toupper(str)` | convert to UPPERCASE | `awk '{print toupper($1)}' file` |
| `tolower(str)` | convert to lowercase | `awk '{print tolower($1)}' file` |
| `substr(str,start,len)` | extract part of a string | `awk '{print substr($1,1,3)}' file` |
| `index(str,substr)` | position of substring | `awk '{print index($1,"a")}' file` |
| `split(str,arr,sep)` | split string into an array | see Section 15 |
| `sprintf(fmt,...)` | format a string (like printf but returns it) | `awk '{s=sprintf("%05d",$2); print s}' file` |
| `sin, cos, sqrt, int, rand` | math functions | `awk '{print sqrt($2)}' file` |

### Examples
```bash
awk '{print toupper($1)}' students.txt
```
👉 Converts every name to uppercase.

```bash
awk '{print length($1)}' students.txt
```
👉 Prints how many characters are in each name.

```bash
awk '{print substr($3,1,3)}' students.txt
```
👉 Prints the first 3 characters of the City field.

---

## 15. Arrays in awk

awk supports **associative arrays** (like dictionaries/hashmaps — indexed by any value, not just numbers).

### Example — Counting occurrences per city
```bash
awk '{count[$3]++} END {for (city in count) print city, count[city]}' students.txt
```
**Output:**
```
Delhi 3
Mumbai 2
Pune 1
Chennai 1
```
👉 `count[$3]++` increases the counter for each city as it's seen; the `END` block then prints the final totals.

### Using `split()` to create an array from a string
```bash
awk 'BEGIN {
  n = split("Delhi,Mumbai,Pune", arr, ",")
  for(i=1; i<=n; i++) print arr[i]
}'
```
**Output:**
```
Delhi
Mumbai
Pune
```

---

## 16. Field Separator Options (`-F`, `FS`, `OFS`)

By default, awk splits fields using **whitespace** (spaces/tabs). But real files often use commas, colons, pipes, etc.

### 16.1 `-F` → Set input Field Separator from command line
```bash
awk -F',' '{print $1}' data.csv
```
👉 Treats `,` as the field separator instead of space.

### 16.2 `FS` → Set field separator inside the program (in BEGIN block)
```bash
awk 'BEGIN {FS=","} {print $1, $3}' data.csv
```

### 16.3 `OFS` → Output Field Separator (used when you print with commas)
```bash
awk 'BEGIN {FS=","; OFS="-"} {print $1, $2}' data.csv
```
👉 Reads fields separated by `,` but prints them joined with `-`.

### Example with our comma-based earlier file (from grep/sed guide)
Imagine `students.csv`:
```
Ravi,25,Delhi,Passed
Sonia,22,Mumbai,Failed
```
```bash
awk -F',' '{print $1, "is from", $3}' students.csv
```
**Output:**
```
Ravi is from Delhi
Sonia is from Mumbai
```

---

## 17. Using awk Scripts (`-f`)

For longer/complex programs, you can save awk code in a separate file instead of typing it all on one line.

**File: `myscript.awk`**
```awk
BEGIN { print "Report:" }
{ print $1, $4 }
END { print "Total records:", NR }
```

Run it:
```bash
awk -f myscript.awk students.txt
```
👉 Cleaner and reusable for bigger tasks — same idea as sed's `-f`.

---

## 18. Practice Problems with Solutions

> 💪 Try solving these yourself first using `students.txt` before checking the solution!

```
Ravi 25 Delhi Passed
Sonia 22 Mumbai Failed
Aman 28 Delhi Passed
Neha 21 Pune Passed
ravi 30 Chennai Failed
Karan 19 Delhi Failed
Priya 24 Mumbai Passed
```

---

**Problem 1:** Print only the names (first column) of all students.

**Solution:**
```bash
awk '{print $1}' students.txt
```

---

**Problem 2:** Print the name and city of students who Passed.

**Solution:**
```bash
awk '$4=="Passed" {print $1, $3}' students.txt
```

---

**Problem 3:** Print total number of students in the file.

**Solution:**
```bash
awk 'END {print NR}' students.txt
```

---

**Problem 4:** Print students whose age is greater than 24.

**Solution:**
```bash
awk '$2 > 24 {print $1, $2}' students.txt
```

---

**Problem 5:** Print names in UPPERCASE for students from Delhi.

**Solution:**
```bash
awk '$3=="Delhi" {print toupper($1)}' students.txt
```

---

**Problem 6:** Count how many students are from each city.

**Solution:**
```bash
awk '{count[$3]++} END {for (c in count) print c, count[c]}' students.txt
```

---

**Problem 7:** Calculate and print the average age of all students.

**Solution:**
```bash
awk '{sum += $2} END {print "Average age:", sum/NR}' students.txt
```

---

**Problem 8:** Print line number along with each line.

**Solution:**
```bash
awk '{print NR, $0}' students.txt
```

---

**Problem 9:** Print only the last field (Status) of each line.

**Solution:**
```bash
awk '{print $NF}' students.txt
```

---

**Problem 10:** Print students whose name starts with "R" (case-sensitive).

**Solution:**
```bash
awk '$1 ~ /^R/ {print $1}' students.txt
```

---

**Problem 11:** Print all fields in reverse order (last field first) for each line.

**Solution:**
```bash
awk '{for(i=NF; i>=1; i--) printf "%s ", $i; print ""}' students.txt
```

---

**Problem 12:** Print name and age nicely formatted/aligned in columns.

**Solution:**
```bash
awk '{printf "%-10s %-5s\n", $1, $2}' students.txt
```

---

**Problem 13:** Count how many students Passed and how many Failed.

**Solution:**
```bash
awk '{status[$4]++} END {for (s in status) print s, status[s]}' students.txt
```

---

**Problem 14:** Print students who are from Delhi **AND** have Passed.

**Solution:**
```bash
awk '$3=="Delhi" && $4=="Passed" {print $1}' students.txt
```

---

**Problem 15:** Given a CSV file `students.csv` (comma-separated), print Name and Status columns only.

**Solution:**
```bash
awk -F',' '{print $1, $4}' students.csv
```

---

## 19. Common Mistakes Beginners Make

1. ❌ Forgetting that awk fields start from `$1`, not `$0` — remember `$0` is the **whole line**, not the first field.

2. ❌ Using `print` with commas but expecting no space — `print $1, $2` inserts `OFS` (default = space) between them automatically.

3. ❌ Forgetting `\n` in `printf` — unlike `print`, `printf` does **not** auto-add a newline, so output can get jumbled together.

4. ❌ Using the wrong field separator for the file type — a comma-separated file needs `-F','`; using default awk (space-based) on a CSV file will treat the **whole line** as `$1`.

5. ❌ Confusing `NR` and `NF`:
   - `NR` = Number of Records (line number)
   - `NF` = Number of Fields (columns in that line)

6. ❌ Forgetting quotes around the awk program — always wrap in **single quotes** `'...'` so the shell doesn't interpret `$1`, `{}`, etc.

7. ❌ Trying to use `=` for comparison instead of `==` — `=` is assignment, `==` is comparison. Using `=` inside a pattern accidentally **assigns** a value instead of comparing.

---

## 20. Quick Cheat Sheet

### Basic Syntax
```bash
awk 'PATTERN {ACTION}' file.txt
```

### Built-in Variables
| Variable | Meaning |
|----------|---------|
| `$0` | entire line |
| `$1, $2...` | individual fields |
| `NF` | number of fields |
| `NR` | current line/record number |
| `FS` | input field separator |
| `OFS` | output field separator |
| `RS` | input record separator |
| `ORS` | output record separator |
| `FILENAME` | current file name |

### Patterns
| Pattern | Meaning |
|---------|---------|
| `/regex/` | lines matching regex |
| `$2 > 25` | condition on a field |
| `cond1 && cond2` | AND |
| `cond1 \|\| cond2` | OR |
| `!/regex/` | NOT matching |
| `NR==2,NR==5` | line range |

### Blocks
| Block | Runs |
|-------|------|
| `BEGIN {}` | once, before processing |
| (main) `{}` | for every matching line |
| `END {}` | once, after processing |

### Common Functions
| Function | Purpose |
|----------|---------|
| `length(str)` | string length |
| `toupper()/tolower()` | case conversion |
| `substr(str,start,len)` | extract substring |
| `split(str,arr,sep)` | split into array |
| `sprintf(fmt,...)` | formatted string |

### Options
| Option | Meaning |
|--------|---------|
| `-F` | set input field separator |
| `-v var=value` | pass a variable from shell |
| `-f script.awk` | run commands from a script file |

---

## 21. Summary

- **awk** is a full text-processing programming language — much more powerful than `grep`/`sed` for structured, column-based data.
- It automatically splits each line (**record**) into **fields** (`$1, $2, ... $NF`), based on a separator (default: whitespace).
- Basic syntax: `awk 'PATTERN {ACTION}' file` — pattern decides **which** lines to act on, action decides **what** to do.
- **BEGIN** runs once before processing (good for headers/setup); **END** runs once after (good for totals/summaries).
- `print` gives quick output; `printf` gives full formatting control (but needs manual `\n`).
- awk supports real programming features: variables, `if/else`, `for`/`while` loops, arithmetic, string functions, and **associative arrays** — making it ideal for counting, summarizing, and calculating directly from text files.
- Use `-F` (or `FS` inside `BEGIN`) to handle files with separators other than whitespace, like CSV (`,`) files.
- For big/reusable logic, store your awk program in a `.awk` file and run it with `-f`.
- awk is the perfect tool when you need to **analyze, calculate, or reformat column data** — think of it as a lightweight Excel/SQL for the command line.

---

### 🎯 Final Tip for a Beginner
Start simple: practice printing specific columns (`$1`, `$2`) and using basic conditions (`$2 > 25`). Once comfortable, move to `BEGIN`/`END` blocks for totals and averages, and finally explore arrays for counting/grouping data — that's when awk starts feeling like a real superpower for handling text data on Linux!

---
*Made for revision purposes — practice each example on your own terminal for best learning.*
