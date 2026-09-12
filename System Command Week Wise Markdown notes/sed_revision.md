# 📘 Complete Revision Guide: `sed` (Stream Editor)

> A beginner-friendly, detailed guide covering everything about `sed` — with examples, explanations, practice problems, and solutions.

---

## Table of Contents

1. [What is `sed`?](#1-what-is-sed)
2. [Basic Syntax](#2-basic-syntax)
3. [How sed Works (Concept)](#3-how-sed-works-concept)
4. [Sample File Used in Examples](#4-sample-file-used-in-examples)
5. [Addressing in sed (Choosing WHICH lines to act on)](#5-addressing-in-sed-choosing-which-lines-to-act-on)
6. [sed Commands — One by One (Detailed)](#6-sed-commands--one-by-one-detailed)
7. [Substitution (`s///`) in Deep Detail](#7-substitution-s-in-deep-detail)
8. [sed Options (Flags)](#8-sed-options-flags)
9. [Regex in sed (BRE vs ERE)](#9-regex-in-sed-bre-vs-ere)
10. [In-place Editing (`-i`)](#10-in-place-editing--i)
11. [Using Multiple sed Commands Together](#11-using-multiple-sed-commands-together)
12. [sed Scripts (`-f`)](#12-sed-scripts--f)
13. [Practice Problems with Solutions](#13-practice-problems-with-solutions)
14. [Common Mistakes Beginners Make](#14-common-mistakes-beginners-make)
15. [Quick Cheat Sheet](#15-quick-cheat-sheet)
16. [Summary](#16-summary)

---

## 1. What is `sed`?

**sed** stands for **S**tream **ED**itor.

It is a **command-line tool** in Linux/Unix used to **edit text automatically**, without opening the file in an editor like `nano` or `vim`.

Think of `sed` as a tool that reads a file **line by line**, applies the instructions you give it (find, replace, delete, insert, print), and outputs the result — **all in one command**, without manual typing.

### Why do we use sed?
- To **find and replace** text in one or many files instantly (like "Find & Replace" but from the terminal).
- To **delete** specific lines from a file.
- To **insert or append** text at specific positions.
- To automate repetitive text-editing tasks in scripts.
- To clean/transform data (logs, CSVs, config files) without manual editing.

> 🧠 Key idea: `sed` **does NOT change the original file by default**. It prints the modified result to the screen. You need a special flag (`-i`) to actually save changes into the file — this is explained later, so no accidental damage happens while you're learning!

---

## 2. Basic Syntax

```bash
sed [OPTIONS] 'COMMAND' FILENAME
```

- **OPTIONS** → flags like `-n`, `-i`, `-e` etc. (optional)
- **COMMAND** → the editing instruction, e.g. `s/old/new/`
- **FILENAME** → the file you want to work on

### Simplest Example

```bash
sed 's/hello/hi/' file.txt
```
👉 This replaces the **first occurrence** of "hello" with "hi" **on each line**, and prints the result on screen (file is untouched).

---

## 3. How sed Works (Concept)

1. `sed` reads the file **line by line** into something called the **pattern space**.
2. For each line, it **applies your command** (substitute, delete, print, etc.).
3. After processing, it **prints the result** (unless told not to).
4. This continues until every line has been processed.
5. By default, the **original file is not modified** — output goes to the terminal (standard output).

---

## 4. Sample File Used in Examples

Let's create one file to use throughout this guide.

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

Create it yourself:
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

## 5. Addressing in sed (Choosing WHICH lines to act on)

Before running a command, sed lets you specify **which line(s)** the command should apply to. This is called an **address**.

```bash
sed 'ADDRESS COMMAND' file.txt
```

| Address Type | Example | Meaning |
|--------------|---------|---------|
| Specific line number | `sed '3d' file.txt` | act on line 3 only |
| Last line | `sed '$d' file.txt` | act on the last line |
| Range of lines | `sed '2,4d' file.txt` | act on lines 2 to 4 |
| Line to end | `sed '3,$d' file.txt` | act on line 3 till end |
| Pattern match | `sed '/Delhi/d' file.txt` | act on lines containing "Delhi" |
| Pattern range | `sed '/Ravi/,/Neha/d' file.txt` | from first match of "Ravi" to first match of "Neha" |
| Step (every nth line) | `sed '1~2d' file.txt` | starting at line 1, every 2nd line |
| Negation | `sed '3!d' file.txt` | act on every line EXCEPT line 3 |

> 💡 If **no address** is given, the command applies to **every line**.

---

## 6. sed Commands — One by One (Detailed)

### 6.1 `s` → Substitute (find & replace)
The most commonly used sed command. Covered in full detail in Section 7.
```bash
sed 's/Delhi/Noida/' students.txt
```

---

### 6.2 `d` → Delete Lines

Deletes the matched line(s) from the output.

```bash
sed '/Failed/d' students.txt
```
**Output (Failed lines removed):**
```
Ravi,25,Delhi,Passed
Aman,28,Delhi,Passed
Neha,21,Pune,Passed
Priya,24,Mumbai,Passed
```

Delete a specific line number:
```bash
sed '2d' students.txt        # deletes line 2
sed '2,4d' students.txt      # deletes lines 2 to 4
sed '$d' students.txt        # deletes the last line
```

---

### 6.3 `p` → Print

Prints the matched line. Usually used **with `-n`** (otherwise the line gets printed twice — once normally, once due to `p`).

```bash
sed -n '/Delhi/p' students.txt
```
**Output:**
```
Ravi,25,Delhi,Passed
Aman,28,Delhi,Passed
Karan,19,Delhi,Failed
```
👉 `-n` suppresses default output, and `p` explicitly prints only the matched lines.

---

### 6.4 `a` → Append (insert text AFTER a line)

```bash
sed '/Ravi/a Newly Added Line' students.txt
```
👉 Inserts "Newly Added Line" **after** every line containing "Ravi".

---

### 6.5 `i` → Insert (insert text BEFORE a line)

```bash
sed '/Sonia/i This is inserted before Sonia line' students.txt
```
👉 Inserts the text **before** the matching line.

---

### 6.6 `c` → Change (replace the entire line)

```bash
sed '/Karan/c This line has been changed' students.txt
```
👉 Replaces the **whole matching line** with new text (not just part of it).

---

### 6.7 `y` → Transliterate (character by character replace, like `tr`)

```bash
echo "hello" | sed 'y/el/ip/'
```
**Output:**
```
hippo... wait let's verify: h-e-l-l-o -> e→i, l→p
```
Actual output: `hippo` is wrong — let's be precise:
```
h e l l o
h i p p o
```
**Output:** `hippo`
👉 Every `e` becomes `i`, every `l` becomes `p`. Position-by-position character mapping — NOT regex.

---

### 6.8 `q` → Quit (stop processing after a point)

```bash
sed '3q' students.txt
```
👉 Prints lines 1–3 and then **stops** (like a `head -n 3` behavior).

---

### 6.9 `n` → Next (move to next line in pattern space)

Used in advanced multi-line processing scripts (loading the next line into the working buffer). Mostly used with more advanced sed scripting — good to know it exists as a beginner.

---

### 6.10 `=` → Print Line Number

```bash
sed '/Delhi/=' students.txt
```
👉 Prints the **line number** just before printing the matching line itself.

---

## 7. Substitution (`s///`) in Deep Detail

This is the **heart of sed** — you will use this the most.

### 7.1 Basic Syntax
```bash
sed 's/PATTERN/REPLACEMENT/FLAGS'
```

- `s` → substitute command
- `PATTERN` → what to search for
- `REPLACEMENT` → what to replace it with
- `FLAGS` → optional modifiers (explained below)

### 7.2 Basic Example
```bash
sed 's/Delhi/Noida/' students.txt
```
👉 Replaces the **first** occurrence of "Delhi" in **each line** with "Noida".

### 7.3 `g` flag → Replace ALL occurrences (Global)
```bash
echo "cat bat cat" | sed 's/cat/dog/g'
```
**Output:** `dog bat dog`
👉 Without `g`, only the **first** "cat" per line would be replaced.

### 7.4 `Ng` flag → Replace from Nth occurrence onward
```bash
echo "cat bat cat sat cat" | sed 's/cat/dog/2g'
```
**Output:** `cat bat dog sat dog`
👉 Replaces from the **2nd occurrence onward**.

### 7.5 `i` (or `I`) flag → Case-insensitive match
```bash
sed 's/ravi/RAVI/gi' students.txt
```
👉 Matches "Ravi", "ravi", "RAVI" etc., regardless of case.

### 7.6 `p` flag → Print the result (with `-n`)
```bash
sed -n 's/Delhi/Noida/p' students.txt
```
👉 Only prints lines where substitution actually happened.

### 7.7 Using different delimiters (helpful for paths with `/`)
Normally `/` is the delimiter, but if your text contains `/` (like a file path), it becomes messy. You can use another character:
```bash
sed 's|/home/user|/home/admin|' file.txt
```
👉 Here `|` is used as delimiter instead of `/`, avoiding confusion.

### 7.8 Using Capture Groups `\(...\)` and Backreferences `\1`
```bash
echo "Ravi,25" | sed 's/\(.*\),\(.*\)/\2-\1/'
```
**Output:** `25-Ravi`
👉 `\(.*\)` captures a group, `\1` and `\2` refer back to them (BRE style — needs escaped parentheses).

In **extended regex mode** (`sed -E`), you don't need backslashes:
```bash
echo "Ravi,25" | sed -E 's/(.*),(.*)/\2-\1/'
```
Same output: `25-Ravi`

---

## 8. sed Options (Flags)

| Option | Meaning |
|--------|---------|
| `-n` | Suppress automatic printing (only print what's explicitly told via `p`) |
| `-e` | Allows multiple sed commands/expressions in one call |
| `-f` | Read sed commands from a script file |
| `-i` | Edit file **in-place** (actually saves changes to the file) |
| `-r` or `-E` | Use Extended Regular Expressions (no need for backslashes with `+ ? | ( )`) |
| `-s` | Treat multiple files as separate streams (not as one continuous stream) |
| `--posix` | Disable GNU sed extensions, strict POSIX behavior |

### Example: `-n` and `p` together
```bash
sed -n '2p' students.txt
```
👉 Prints only line 2 (without `-n`, every line would print once normally + line 2 would print twice).

### Example: `-e` for multiple commands
```bash
sed -e 's/Delhi/Noida/' -e 's/Mumbai/Pune/' students.txt
```
👉 Runs two substitutions in a single command.

---

## 9. Regex in sed (BRE vs ERE)

By default, `sed` uses **Basic Regular Expressions (BRE)** — same concept as plain `grep`.

| Symbol | BRE (default sed) | ERE (`sed -E` / `sed -r`) |
|--------|--------------------|----------------------------|
| OR | `\|` | `|` |
| One or more | `\+` | `+` |
| Zero or one | `\?` | `?` |
| Grouping | `\( \)` | `( )` |
| Repeat n times | `\{n\}` | `{n}` |

### Example — BRE vs ERE for OR condition
```bash
sed 's/Delhi\|Mumbai/City/' students.txt      # BRE (default)
sed -E 's/Delhi|Mumbai/City/' students.txt    # ERE (cleaner)
```
Both do the same thing — `-E` just avoids backslashes, similar to `egrep` vs `grep`.

---

## 10. In-place Editing (`-i`)

By default, sed **prints to screen only**. To actually **save changes into the file**, use `-i`.

```bash
sed -i 's/Delhi/Noida/' students.txt
```
👉 Now `students.txt` itself is modified permanently.

### ⚠️ Important: Always take a backup first!
```bash
sed -i.bak 's/Delhi/Noida/' students.txt
```
👉 This creates a backup file `students.txt.bak` **before** modifying the original — a safety net for beginners.

> 🧠 On **macOS/BSD sed**, `-i` requires an explicit (even empty) backup extension: `sed -i '' 's/old/new/' file.txt`. On **Linux (GNU sed)**, `sed -i 's/old/new/' file.txt` works directly.

---

## 11. Using Multiple sed Commands Together

You can chain commands using `;` or multiple `-e` flags.

```bash
sed 's/Delhi/Noida/; s/Failed/Fail/' students.txt
```
or
```bash
sed -e 's/Delhi/Noida/' -e 's/Failed/Fail/' students.txt
```
Both do two substitutions in one pass.

You can also write multi-line scripts using `{ }`:
```bash
sed '/Delhi/{s/Passed/PASSED/}' students.txt
```
👉 Only applies the substitution to lines that first match "Delhi".

---

## 12. sed Scripts (`-f`)

For big/complex editing tasks, you can store sed commands in a separate script file.

**File: `myscript.sed`**
```
s/Delhi/Noida/
s/Mumbai/Pune/
/Failed/d
```

Run it:
```bash
sed -f myscript.sed students.txt
```
👉 Cleaner than typing long commands directly in the terminal, especially for reusable transformations.

---

## 13. Practice Problems with Solutions

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

**Problem 1:** Replace all occurrences of "Delhi" with "Noida" (only on screen, don't save).

**Solution:**
```bash
sed 's/Delhi/Noida/' students.txt
```

---

**Problem 2:** Delete all lines that contain "Failed".

**Solution:**
```bash
sed '/Failed/d' students.txt
```

---

**Problem 3:** Print only line number 3 from the file.

**Solution:**
```bash
sed -n '3p' students.txt
```

---

**Problem 4:** Replace "Passed" with "PASS" permanently in the file (with a backup).

**Solution:**
```bash
sed -i.bak 's/Passed/PASS/' students.txt
```

---

**Problem 5:** Delete the last line of the file.

**Solution:**
```bash
sed '$d' students.txt
```

---

**Problem 6:** Print all lines EXCEPT line 1 (i.e., skip line 1).

**Solution:**
```bash
sed '1d' students.txt
```

---

**Problem 7:** Replace "Mumbai" with "Pune" only in lines that contain "Failed".

**Solution:**
```bash
sed '/Failed/s/Mumbai/Pune/' students.txt
```

---

**Problem 8:** Add a new line "--- END OF RECORD ---" after every line containing "Passed".

**Solution:**
```bash
sed '/Passed/a --- END OF RECORD ---' students.txt
```

---

**Problem 9:** Replace the entire line containing "Karan" with "RECORD REMOVED".

**Solution:**
```bash
sed '/Karan/c RECORD REMOVED' students.txt
```

---

**Problem 10:** Show line numbers along with lines containing "Delhi".

**Solution:**
```bash
sed -n '/Delhi/{=;p}' students.txt
```

---

**Problem 11:** Delete lines 2 through 4.

**Solution:**
```bash
sed '2,4d' students.txt
```

---

**Problem 12:** Swap the first two comma-separated fields (Name and Age) on each line — e.g. `Ravi,25,...` → `25,Ravi,...`

**Solution:**
```bash
sed -E 's/([^,]*),([^,]*)/\2,\1/' students.txt
```

---

**Problem 13:** Replace "Delhi" OR "Mumbai" with "Metro" using extended regex.

**Solution:**
```bash
sed -E 's/Delhi|Mumbai/Metro/' students.txt
```

---

**Problem 14:** Print only the first 3 lines of the file, then stop.

**Solution:**
```bash
sed '3q' students.txt
```

---

**Problem 15:** Replace "ravi" with "RAVI", matching regardless of uppercase/lowercase, and save the change to the file.

**Solution:**
```bash
sed -i 's/ravi/RAVI/gi' students.txt
```

---

## 14. Common Mistakes Beginners Make

1. ❌ Forgetting `-i` and expecting the file to be permanently changed — by default sed **only prints to screen**, it never touches the file unless `-i` is used.

2. ❌ Using `-i` directly without a backup — always test with plain `sed 's/.../.../'` (no `-i`) first, then add `-i.bak` once you're confident.

3. ❌ Forgetting the `g` flag when expecting **all** occurrences to be replaced — without `g`, sed replaces only the **first match per line**.

4. ❌ Using `sed -n` but forgetting `p` — with `-n` alone, sed prints **nothing** unless you explicitly add `p`.

5. ❌ Using `+`, `?`, `|`, `()` in default (BRE) sed without escaping them — these need `\+`, `\?`, `\|`, `\( \)` unless you use `-E`/`-r`.

6. ❌ Confusing `d` (delete) with `c` (change) — `d` removes the line completely, `c` replaces its content.

7. ❌ Not quoting the sed command properly — always wrap the command in **single quotes** `'...'` to prevent the shell from interpreting special characters.

---

## 15. Quick Cheat Sheet

### Commands
| Command | Meaning |
|---------|---------|
| `s/old/new/` | substitute (find & replace) |
| `d` | delete line(s) |
| `p` | print line(s) |
| `a text` | append text after line |
| `i text` | insert text before line |
| `c text` | change (replace) whole line |
| `y/abc/xyz/` | transliterate characters |
| `q` | quit after this point |
| `=` | print line number |

### Flags for `s///`
| Flag | Meaning |
|------|---------|
| `g` | replace all occurrences in line |
| `Ng` | replace from Nth occurrence onward |
| `i` / `I` | case-insensitive match |
| `p` | print line if substitution happened (use with `-n`) |

### Options
| Option | Meaning |
|--------|---------|
| `-n` | suppress automatic printing |
| `-e` | multiple commands in one call |
| `-f file` | read commands from script file |
| `-i` | edit file in-place (save changes) |
| `-i.bak` | edit in-place + create backup |
| `-E` / `-r` | extended regex (no backslash needed) |

### Addressing
| Address | Meaning |
|---------|---------|
| `3` | line 3 |
| `2,4` | lines 2 to 4 |
| `$` | last line |
| `/pattern/` | lines matching pattern |
| `/p1/,/p2/` | from first match of p1 to first match of p2 |
| `3!` | every line except line 3 |

---

## 16. Summary

- **sed** = Stream Editor — edits text automatically, line by line, without opening the file manually.
- Basic syntax: `sed [options] 'command' filename`.
- By **default**, sed only prints the result to the screen — the original file is **untouched** unless you use `-i`.
- The most important and most-used command is **substitution**: `s/pattern/replacement/flags`.
- Common flags for substitution: `g` (all matches), `i` (ignore case), `p` (print if changed).
- Other essential commands: `d` (delete), `p` (print), `a` (append), `i` (insert), `c` (change).
- **Addressing** (line number, range, or pattern) lets you control exactly which lines a command applies to.
- Default sed uses **BRE** (Basic Regex) — special symbols like `+ ? | ( )` need backslashes. Use `-E`/`-r` for **ERE** (Extended Regex) to avoid backslashes, similar to how `egrep` relates to `grep`.
- Always test your sed command **without `-i` first**, and use `-i.bak` for a safety backup when you're ready to save changes.
- sed is extremely powerful for automating text edits in scripts, cleaning data files, and editing configuration files in bulk.

---

### 🎯 Final Tip for a Beginner
Always practice sed commands **without `-i` first** (just let it print to the screen). Once you're 100% sure the command does what you want, then add `-i` (with a backup like `-i.bak`) to actually modify the file. This habit will save you from accidentally destroying important files while learning!

---
*Made for revision purposes — practice each example on your own terminal for best learning.*
