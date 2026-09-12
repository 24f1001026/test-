# 📘 Complete Revision Guide: `grep` and `egrep`

> A beginner-friendly, detailed guide covering everything about `grep` and `egrep` — with examples, explanations, practice problems, and solutions.

---

## Table of Contents

1. [What is `grep`?](#1-what-is-grep)
2. [Basic Syntax](#2-basic-syntax)
3. [How grep Works (Concept)](#3-how-grep-works-concept)
4. [Sample File Used in Examples](#4-sample-file-used-in-examples)
5. [grep Options — One by One (Detailed)](#5-grep-options--one-by-one-detailed)
6. [Regular Expressions Basics (Needed for grep)](#6-regular-expressions-basics-needed-for-grep)
7. [What is `egrep`?](#7-what-is-egrep)
8. [Difference Between `grep` and `egrep`](#8-difference-between-grep-and-egrep)
9. [egrep / grep -E Examples](#9-egrep--grep--e-examples)
10. [Combining Multiple Options](#10-combining-multiple-options)
11. [Practice Problems with Solutions](#11-practice-problems-with-solutions)
12. [Common Mistakes Beginners Make](#12-common-mistakes-beginners-make)
13. [Quick Cheat Sheet](#13-quick-cheat-sheet)
14. [Summary](#14-summary)

---

## 1. What is `grep`?

**grep** stands for **G**lobal **R**egular **E**xpression **P**rint.

It is a **command-line tool** used in Linux/Unix systems to **search for a specific pattern (text) inside a file or multiple files**, and it **prints the lines that match** that pattern.

Think of `grep` like the **"Find" (Ctrl+F)** feature in a text editor, but for the terminal — and much more powerful because it supports **regular expressions**.

### Why do we use grep?
- To search for a word/phrase inside one or thousands of files instantly.
- To filter output of another command (using pipes `|`).
- To search log files for errors.
- To find specific lines in code files.

---

## 2. Basic Syntax

```bash
grep [OPTIONS] "PATTERN" FILENAME
```

- **OPTIONS** → flags like `-i`, `-v`, `-c` etc. (optional)
- **PATTERN** → the text/word/regex you want to search
- **FILENAME** → the file(s) you want to search in

### Simplest Example

```bash
grep "hello" file.txt
```
👉 This searches for the word `hello` in `file.txt` and prints every line that contains it.

---

## 3. How grep Works (Concept)

1. `grep` reads the file **line by line**.
2. For each line, it checks: *"Does this line contain the pattern I'm searching for?"*
3. If **yes** → it prints that entire line.
4. If **no** → it skips that line.
5. This continues until the whole file is scanned.

So grep never gives you "part of a line" — by default it gives you the **whole matching line**.

---

## 4. Sample File Used in Examples

Let's create one file that we will use throughout this guide, so every example makes sense practically.

**File name:** `students.txt`

```
Ravi,25,Delhi,Passed
Sonia,22,Mumbai,Failed
Aman,28,Delhi,Passed
Neha,21,Pune,Passed
ravi,30,Chennai,Failed
Karan,19,Delhi,Failed
Priya,24,Mumbai,Passed
```

You can create this file yourself to practice:
```bash
cat > students.txt << EOF
Ravi,25,Delhi,Passed
Sonia,22,Mumbai,Failed
Aman,28,Delhi,Passed
Neha,21,Pune,Passed
ravi,30,Chennai,Failed
Karan,19,Delhi,Failed
Priya,24,Mumbai,Passed
EOF
```

---

## 5. grep Options — One by One (Detailed)

### 5.1 `-i` → Ignore Case (case-insensitive search)

By default grep is **case-sensitive** (`Ravi` and `ravi` are different).
`-i` makes it ignore uppercase/lowercase difference.

```bash
grep -i "ravi" students.txt
```
**Output:**
```
Ravi,25,Delhi,Passed
ravi,30,Chennai,Failed
```
👉 Without `-i`, only the exact-case `ravi` line would appear.

---

### 5.2 `-v` → Invert Match (show lines that DO NOT match)

```bash
grep -v "Passed" students.txt
```
**Output:**
```
Sonia,22,Mumbai,Failed
ravi,30,Chennai,Failed
Karan,19,Delhi,Failed
```
👉 Shows all lines **except** those containing "Passed".

---

### 5.3 `-c` → Count of Matching Lines

Instead of printing lines, it prints **how many lines matched**.

```bash
grep -c "Delhi" students.txt
```
**Output:**
```
3
```

---

### 5.4 `-n` → Show Line Numbers

```bash
grep -n "Mumbai" students.txt
```
**Output:**
```
2:Sonia,22,Mumbai,Failed
7:Priya,24,Mumbai,Passed
```
👉 Very useful in large files/code to know **where** the match is.

---

### 5.5 `-l` → Show Only File Names (not the lines)

Useful when searching **multiple files** and you only want to know **which files contain the pattern**.

```bash
grep -l "Passed" *.txt
```
**Output:**
```
students.txt
```

---

### 5.6 `-L` → Show File Names that DON'T Match

```bash
grep -L "Passed" *.txt
```
👉 Opposite of `-l`. Lists files where the pattern is **absent**.

---

### 5.7 `-w` → Match Whole Word Only

Without `-w`, grep matches even if pattern is part of a bigger word.

```bash
grep -w "Delhi" students.txt
```
👉 This matches only if `Delhi` appears as a **complete word**, not as part of "NewDelhi" or "Delhite".

---

### 5.8 `-x` → Match Whole Line Only

The **entire line** must exactly equal the pattern.

```bash
grep -x "Neha,21,Pune,Passed" students.txt
```
**Output:**
```
Neha,21,Pune,Passed
```
👉 If even one character differs, no output.

---

### 5.9 `-o` → Print Only the Matched Part (not full line)

```bash
grep -o "Passed" students.txt
```
**Output:**
```
Passed
Passed
Passed
Passed
```
👉 Instead of printing the whole line, it prints only the matched word — once per match.

---

### 5.10 `-r` or `-R` → Recursive Search (search inside folders)

Searches **all files inside a directory and subdirectories**.

```bash
grep -r "error" /var/log/
```
👉 `-R` also follows symbolic links, `-r` does not.

---

### 5.11 `-A`, `-B`, `-C` → Show Context Lines

These show lines **After**, **Before**, or **around (Context)** a match. Extremely useful in log file debugging.

```bash
grep -A 2 "Failed" students.txt   # 2 lines AFTER match
grep -B 1 "Failed" students.txt   # 1 line BEFORE match
grep -C 1 "Failed" students.txt   # 1 line before AND after
```

---

### 5.12 `-e` → Specify Multiple Patterns

```bash
grep -e "Delhi" -e "Mumbai" students.txt
```
👉 Matches lines containing **either** "Delhi" **or** "Mumbai".

---

### 5.13 `-f` → Read Patterns from a File

If you have many search patterns saved in a file:

```bash
grep -f patterns.txt students.txt
```
Where `patterns.txt` contains one pattern per line.

---

### 5.14 `-q` → Quiet Mode (no output, only exit status)

Used mostly inside shell scripts to **check** if a pattern exists.

```bash
grep -q "Passed" students.txt
echo $?
```
👉 `$?` gives `0` if found, `1` if not found. No text is printed.

---

### 5.15 `--color` → Highlight the Matched Text

```bash
grep --color "Delhi" students.txt
```
👉 Highlights the matched word in the terminal (usually red).

---

### 5.16 `-E` → Extended Regex (same as using `egrep`)

Covered in detail in Section 7 & 8.

---

## 6. Regular Expressions Basics (Needed for grep)

grep becomes truly powerful with **regex** (patterns), not just plain words.

| Symbol | Meaning | Example | Matches |
|--------|---------|---------|---------|
| `^` | Start of line | `^Ravi` | Line starting with "Ravi" |
| `$` | End of line | `Passed$` | Line ending with "Passed" |
| `.` | Any single character | `R.vi` | Ravi, Rvvi, R9vi etc. |
| `*` | Zero or more of previous char | `ra*vi` | rvi, ravi, raavi |
| `[abc]` | Any one of a, b, or c | `[RS]avi` | Ravi or Savi |
| `[^abc]` | Any character EXCEPT a,b,c | `[^R]avi` | not starting with Ravi |
| `[0-9]` | Any digit | `[0-9][0-9]` | any 2-digit number |
| `\` | Escape special character | `\.` | Literal dot `.` |

### Examples using regex in grep

```bash
grep "^Ravi" students.txt        # lines starting with Ravi
grep "Passed$" students.txt      # lines ending with Passed
grep "^[RS]" students.txt        # lines starting with R or S
grep "[0-9][0-9]" students.txt   # lines containing a 2-digit number
```

> ⚠️ Note: Plain `grep` uses **Basic Regular Expressions (BRE)**. Some symbols like `+`, `?`, `|`, `()` **need a backslash** (`\+`, `\?`, `\|`, `\(\)`) to work in plain grep. This is exactly the problem `egrep` solves — explained next.

---

## 7. What is `egrep`?

**egrep** = **E**xtended **grep**.

It is the same as running:
```bash
grep -E "pattern" file.txt
```

`egrep` understands **Extended Regular Expressions (ERE)**, which support more powerful symbols **directly, without needing a backslash**:

| Symbol | Meaning |
|--------|---------|
| `+` | one or more of previous character |
| `?` | zero or one of previous character |
| `\|` | OR (alternation) |
| `()` | grouping |
| `{n,m}` | repeat n to m times |

---

## 8. Difference Between `grep` and `egrep`

| Feature | grep | egrep |
|---------|------|-------|
| Full form | Global Regular Expression Print | Extended grep |
| Regex type used | Basic Regular Expression (BRE) | Extended Regular Expression (ERE) |
| Need backslash for `+ ? | ( )` | Yes | No |
| Command equivalent | `grep pattern` | `grep -E pattern` |
| Speed | Slightly faster for simple search | Slightly more overhead (negligible) |
| Recommended today | Yes, for simple/basic search | Yes, for complex pattern matching |

### Example showing the exact difference

**Using plain grep (need backslash for OR):**
```bash
grep "Delhi\|Mumbai" students.txt
```

**Using egrep (no backslash needed):**
```bash
egrep "Delhi|Mumbai" students.txt
```

Both give the **same result** — but `egrep`'s syntax is cleaner and easier to read.

> 💡 In modern Linux, `egrep` is technically "deprecated" in favor of `grep -E`, but it still works everywhere and is very common to see.

---

## 9. egrep / grep -E Examples

### 9.1 OR condition (`|`)
```bash
egrep "Delhi|Chennai" students.txt
```
👉 Lines containing Delhi OR Chennai.

### 9.2 One or more (`+`)
```bash
egrep "a+" students.txt
```
👉 Lines containing one or more `a`.

### 9.3 Zero or one (`?`)
```bash
egrep "colou?r" file.txt
```
👉 Matches both "color" and "colour".

### 9.4 Grouping with `()`
```bash
egrep "(Ravi|Sonia),2[0-9]" students.txt
```
👉 Matches "Ravi" or "Sonia" followed by a comma and a number from 20-29.

### 9.5 Repetition `{n,m}`
```bash
egrep "[0-9]{2}" students.txt
```
👉 Matches exactly 2 consecutive digits.

---

## 10. Combining Multiple Options

You can combine flags together for powerful searches.

```bash
grep -inv "delhi" students.txt
```
Breakdown:
- `-i` → ignore case
- `-n` → show line number
- `-v` → invert match (show lines NOT containing "delhi")

```bash
grep -c -i "passed" students.txt
```
👉 Count of lines containing "passed" (case-insensitive).

---

## 11. Practice Problems with Solutions

> 💪 Try solving these yourself first using `students.txt` before checking the solution!

```
Ravi,25,Delhi,Passed
Sonia,22,Mumbai,Failed
Aman,28,Delhi,Passed
Neha,21,Pune,Passed
ravi,30,Chennai,Failed
Karan,19,Delhi,Failed
Priya,24,Mumbai,Passed
```

---

**Problem 1:** Find all lines that contain "Delhi", regardless of case.

**Solution:**
```bash
grep -i "delhi" students.txt
```

---

**Problem 2:** Find all lines that do NOT contain "Passed".

**Solution:**
```bash
grep -v "Passed" students.txt
```

---

**Problem 3:** Count how many students are from Mumbai.

**Solution:**
```bash
grep -c "Mumbai" students.txt
```

---

**Problem 4:** Show line numbers of all "Failed" entries.

**Solution:**
```bash
grep -n "Failed" students.txt
```

---

**Problem 5:** Find lines where the name starts with "R".

**Solution:**
```bash
grep "^R" students.txt
```

---

**Problem 6:** Find lines that end with "Passed".

**Solution:**
```bash
grep "Passed$" students.txt
```

---

**Problem 7:** Find students who are from Delhi **OR** Mumbai (use egrep).

**Solution:**
```bash
egrep "Delhi|Mumbai" students.txt
```

---

**Problem 8:** Find lines containing a 2-digit age starting with 2 (i.e., 20-29).

**Solution:**
```bash
grep -E "2[0-9]" students.txt
```

---

**Problem 9:** Print only the matched word "Passed" from each line (not full lines).

**Solution:**
```bash
grep -o "Passed" students.txt
```

---

**Problem 10:** Check (in a script, without printing anything) whether "Karan" exists in the file.

**Solution:**
```bash
grep -q "Karan" students.txt
echo $?
```
(`0` means found, `1` means not found)

---

**Problem 11:** Find all lines that exactly match: `Neha,21,Pune,Passed`

**Solution:**
```bash
grep -x "Neha,21,Pune,Passed" students.txt
```

---

**Problem 12:** Search recursively for the word "Passed" in all `.txt` files inside the current folder and subfolders.

**Solution:**
```bash
grep -r "Passed" --include="*.txt" .
```

---

**Problem 13:** Find lines that contain either "Delhi" or "Pune" using plain grep (not egrep).

**Solution:**
```bash
grep "Delhi\|Pune" students.txt
```
👉 Notice the backslash before `|` — required in plain grep, not needed in egrep.

---

**Problem 14:** Find the word "ravi" as a **whole word only** (case-sensitive).

**Solution:**
```bash
grep -w "ravi" students.txt
```

---

**Problem 15:** From `students.txt`, list only the **file names** (imagine multiple files) that contain the word "Failed".

**Solution:**
```bash
grep -l "Failed" *.txt
```

---

## 12. Common Mistakes Beginners Make

1. ❌ Forgetting quotes around the pattern:
   ```bash
   grep Delhi students.txt      # works but risky
   grep "Delhi" students.txt    # ✅ safer, always use quotes
   ```

2. ❌ Using `+`, `?`, `|`, `()` in plain grep without backslash — gives wrong/no result.
   ✅ Either escape them (`\+`) or use `egrep` / `grep -E`.

3. ❌ Forgetting `-i` when case doesn't matter and expecting all matches.

4. ❌ Confusing `-v` (invert match) with `-c` (count) — remember:
   - `-v` = opposite lines
   - `-c` = just a number (count)

5. ❌ Thinking grep searches "anywhere" in file structure — by default it only searches the given file, not subfolders (need `-r` for that).

---

## 13. Quick Cheat Sheet

| Option | Meaning |
|--------|---------|
| `-i` | ignore case |
| `-v` | invert match (non-matching lines) |
| `-c` | count of matching lines |
| `-n` | show line numbers |
| `-l` | show file names with match |
| `-L` | show file names without match |
| `-w` | match whole word |
| `-x` | match whole line |
| `-o` | print only matched part |
| `-r` / `-R` | recursive search in folders |
| `-A n` | show n lines after match |
| `-B n` | show n lines before match |
| `-C n` | show n lines before & after match |
| `-e` | specify multiple patterns |
| `-f file` | read patterns from a file |
| `-q` | quiet mode (used in scripts) |
| `--color` | highlight matched text |
| `-E` | extended regex (same as egrep) |

### Regex Quick Reference
| Symbol | Meaning |
|--------|---------|
| `^` | start of line |
| `$` | end of line |
| `.` | any one character |
| `*` | zero or more of previous |
| `+` (egrep only / `\+` in grep) | one or more of previous |
| `?` (egrep only / `\?` in grep) | zero or one of previous |
| `\|` (egrep: `|`) | OR |
| `[abc]` | any one of a,b,c |
| `[^abc]` | none of a,b,c |
| `[0-9]` | any digit |
| `{n,m}` (egrep only) | repeat n to m times |

---

## 14. Summary

- **grep** searches for a pattern in text and prints matching lines. It's one of the most-used Linux commands for searching, filtering, and debugging.
- Basic syntax: `grep [options] "pattern" filename`.
- Important everyday options: `-i` (ignore case), `-v` (invert), `-n` (line numbers), `-c` (count), `-r` (recursive), `-w` (whole word), `-o` (only matched text).
- Plain `grep` uses **Basic Regular Expressions (BRE)** — special symbols like `+ ? | ( )` need a backslash to work.
- **egrep** (or `grep -E`) uses **Extended Regular Expressions (ERE)** — the same special symbols work **without** a backslash, making complex patterns easier to write and read.
- Both commands are extremely useful together with pipes (`|`) to filter output of other commands, e.g. `ps aux | grep "python"`.
- Practice regularly with real files/logs — grep skill improves fastest with hands-on use, not just reading theory.

---

### 🎯 Final Tip for a Beginner
Start by using plain `grep` with simple words. Once comfortable, slowly start using regex symbols (`^`, `$`, `.`, `*`). Once regex feels comfortable, move to `egrep`/`grep -E` for OR conditions, grouping, and repetition — that's when you'll feel truly powerful with searching text on Linux!

---
*Made for revision purposes — practice each example on your own terminal for best learning.*
