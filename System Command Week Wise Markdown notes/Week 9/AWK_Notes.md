

# AWK Programming — Week 9 (Lecture 1 & 2)

**Topic:** A language for processing fields and records
**Coverage:** Part 1 and Part 2 combined, structured for exam revision

---

## 1. Introduction to AWK

**Concept:**
AWK is a **programming language** specifically designed for processing text that is organized into **fields and records** (i.e., structured or semi-structured text such as log files, CSVs, and configuration files).

**Key facts:**

| Point | Detail |
|---|---|
| Name origin | AWK is an abbreviation formed from the surnames of its three creators: **A**ho, **W**einberger, and **K**ernighan |
| Standardization | AWK is part of the **POSIX** standard, specifically **IEEE 1003.1-2008** |
| Common variants | `nawk` (New AWK), `gawk` (GNU AWK), `mawk` (a fast, minimalist AWK), and others |
| Extended variant | `gawk` contains extra features that go **beyond** (extend) the POSIX specification |

**Notes for exam:**

- AWK is *not* just a command — it is a full scripting/programming language.
- Remember the acronym expansion: **A**ho, **W**einberger, **K**ernighan — a common one-mark question.
- `gawk` is the most feature-rich and most commonly available variant on Linux systems.

---

## 2. Execution Model

**Concept:**
AWK's entire design revolves around how it reads input and breaks it down before running any code.

**Core ideas:**

1. The **input stream** is treated as a set of **records**.
   - Example: If the **record separator** is the newline character (`"\n"`), then each **line** of input becomes one record.
2. Each **record** is further broken down into a sequence of **fields**.

   - Example: If the **field separator** is a space (`" "`), then each **word** in a line becomes one field.
3. The **splitting of records into fields happens automatically** — the programmer does not need to write manual parsing code.

4. Every **code block** in an AWK program executes **once per matching record**, but only if that block's **pattern** matches the current record.

**Simple flow diagram (conceptual):**

```
Input Stream
    │
    ▼
Split into RECORDS  (default separator = newline "\n")
    │
    ▼
Split each record into FIELDS  (default separator = whitespace " ")
    │
    ▼
For each record → check PATTERN → if TRUE → run the associated CODE BLOCK
```

**Example:**

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

*Explanation:* Each line is a record (default `RS = "\n"`), and each word is a field (default `FS = " "`). `$1` refers to the first field, `$2` to the second field.

---

## 3. Usage of AWK

AWK can be used in **two main ways**: directly on the command line, or as a standalone interpreted script.

### 3.1 Running AWK from the Command Line

**Syntax:**
```bash
command | awk -F"delimiter" '{ action }'
```

**Example (from the source material):**
```bash
cat /etc/passwd | awk -F":" '{print $1}'
```

*Explanation:*

- `cat /etc/passwd` streams the contents of the password file.
- `-F":"` tells AWK to use `:` (colon) as the **field separator** instead of the default whitespace.
- `{print $1}` prints only the **first field** of every record — in this case, the username.

**Another example:**
```bash
echo "John:25:Manager" | awk -F":" '{print $2}'
```
Output:
```
25
```

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

**Note:** The typographic quotes (`"` or `”`) shown in some textbook slides must be replaced with standard straight quotes (`"`) when typing real AWK code, otherwise the shell/AWK interpreter will throw a syntax error. This is a common typo to watch for in exams and in real practice.

---

## 4. Built-in Variables

**Concept:**
AWK automatically maintains a set of special variables that describe the current state of parsing (current record, current field separator, counts, etc.). These do **not** need to be declared — they are ready to use.

Because this is a large topic, it is split into **two tables** exactly as it should be understood and revised.

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

**Key distinction to remember (common exam trap):**

- `NR` = record number across **all** input files combined.
- `FNR` = record number **within the current file only** (resets when a new file starts).

**Example demonstrating `NR` vs `FNR`:**

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

**Example demonstrating `NF`, `OFS`, and `$0`:**
```bash
echo "one two three" | awk '{print "Number of fields:", NF}'
```
Output:
```
Number of fields: 3
```

```bash
echo "one two three" | awk 'BEGIN{OFS="-"} {print $1,$2,$3}'
```
Output:
```
one-two-three
```

---

## 5. Structure of an AWK Script

**Concept:**
Every AWK program is fundamentally a series of **pattern–action pairs**.

**General syntax:**
```awk
pattern { procedure }
```

An AWK script can be composed of the following building blocks:

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

**Note:** All of the above can be freely combined inside pattern–action blocks to build complex text-processing logic.

---

## 6. Execution Blocks — BEGIN, END, Pattern Blocks

**Concept:**
AWK programs are organized into blocks that run at specific times during execution.

**Syntax of the four core block types:**
```awk
BEGIN { commands; }        # runs once, before any file is read

END { commands; }          # runs once, after all files are read

{ commands; }               # runs on every record (no pattern = always true)

pattern { commands; }      # runs only on records where 'pattern' evaluates true
```

### Rules and Properties

| Block Type | When It Runs | Notes |
|---|---|---|
| `BEGIN { }` | **Once**, before any input is read | Used for initialization (e.g., setting `FS`, printing headers) |
| `END { }` | **Once**, after all input has been read | Used for summaries, totals, final reports |
| `{ }` (no pattern) | For **every** record | Runs unconditionally |
| `pattern { }` | Once per record **where the pattern evaluates to true** | The script can contain multiple such blocks |

**Additional rules (important for exams):**

- These blocks **can appear anywhere** in the script.
- They **can appear multiple times** — e.g., you may have more than one `BEGIN` block; they all run in the order written.
- Patterns can be **combined** using logical operators: `&&` (AND), `||` (OR), `!` (NOT).
- A **range of records** can be specified using a **comma** between two patterns (range pattern).

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

**Example — Range pattern (comma-separated):**
```bash
awk '/START/,/END/ { print }' logfile.txt
```
*Explanation:* Prints all lines starting from the line that matches `/START/` up to (and including) the line that matches `/END/`.

---

## 7. Operators in AWK

AWK supports several categories of operators. These are grouped below exactly as they should be studied, with each category expanded into its own explanation table.

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

**Example:**
```bash
awk 'BEGIN{ x=5; x+=3; print x }'   # Output: 8
awk 'BEGIN{ x=5; x*=2; print x }'   # Output: 10
```

### 7.2 Logical Operators

| Operator | Meaning |
|---|---|
| Logical OR (two pipe characters together) | Logical OR |
| `&&` | Logical AND |

**Example:**
```bash
awk 'BEGIN{ if (5>3 && 2<4) print "Both true" }'
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

**Example:**
```bash
awk 'BEGIN{ print 2^10 }'    # Output: 1024
awk 'BEGIN{ print 10 % 3 }'  # Output: 1
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

**Example:**
```bash
awk '$1 == "Alice" { print "Found Alice" }' data.txt
```

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
| *(blank / space)* | String **concatenation** operator (juxtaposition, no explicit symbol) |

**Example — Ternary operator:**
```bash
awk '{ print ($2 > 25 ? "Senior" : "Junior") }' data.txt
```

**Example — Regex match operator `~`:**
```bash
awk '$1 ~ /^A/ { print $1, "starts with A" }' data.txt
```

**Example — Array membership `in`:**
```bash
awk 'BEGIN{ arr["x"]=1; if ("x" in arr) print "x exists" }'
```

**Example — Concatenation (blank operator):**
```bash
awk 'BEGIN{ a="Hello"; b="World"; print a " " b }'
# Output: Hello World
```
*Explanation:* There is no `+` for joining strings in AWK — simply placing two values next to each other (separated by a space) concatenates them.

---

## 8. Functions and Commands

AWK ships with many built-in functions, grouped by category below. Each group is expanded into its own explanation table with a short description and example, since the original material lists them only by name.

### 8.1 Arithmetic Functions

| Function | Purpose | Example |
|---|---|---|
| `atan2(y,x)` | Arctangent of `y/x` | `awk 'BEGIN{print atan2(1,1)}'` |
| `cos(x)` | Cosine of `x` (radians) | `awk 'BEGIN{print cos(0)}'` → `1` |
| `exp(x)` | Exponential, e^x | `awk 'BEGIN{print exp(1)}'` |
| `int(x)` | Truncates `x` to an integer | `awk 'BEGIN{print int(4.9)}'` → `4` |
| `log(x)` | Natural logarithm of `x` | `awk 'BEGIN{print log(1)}'` → `0` |
| `rand()` | Random number between 0 and 1 | `awk 'BEGIN{print rand()}'` |
| `sin(x)` | Sine of `x` (radians) | `awk 'BEGIN{print sin(0)}'` → `0` |
| `sqrt(x)` | Square root of `x` | `awk 'BEGIN{print sqrt(16)}'` → `4` |
| `srand()` | Seeds the random number generator | `awk 'BEGIN{srand(); print rand()}'` |

### 8.2 String Functions

| Function | Purpose | Example |
|---|---|---|
| `asort(arr)` | Sorts array values (gawk extension) | `asort(myarr)` |
| `asorti(arr)` | Sorts array indices (gawk extension) | `asorti(myarr)` |
| `gsub(re,rep,tgt)` | Global substitution of `re` with `rep` in `tgt` | `gsub(/o/,"0",$0)` |
| `index(s,t)` | Position of substring `t` inside `s` | `index("hello","ll")` → `3` |
| `length(s)` | Length of string `s` (or `$0` if omitted) | `length("hello")` → `5` |
| `match(s,re)` | Tests if `s` contains a match for `re`; sets `RSTART`/`RLENGTH` | `match("hello",/l+/)` |
| `split(s,arr,fs)` | Splits string `s` into array `arr` using separator `fs` | `split("a:b:c",parts,":")` |
| `sprintf(fmt,...)` | Returns a formatted string (like `printf` but returns instead of prints) | `x=sprintf("%05d",7)` |
| `sub(re,rep,tgt)` | Substitutes the **first** match of `re` with `rep` in `tgt` | `sub(/o/,"0",$0)` |
| `strtonum(s)` | Converts string `s` to a number (gawk extension) | `strtonum("0x1A")` |
| `substr(s,m,n)` | Extracts substring of `s` starting at `m`, length `n` | `substr("hello",2,3)` → `ell` |
| `tolower(s)` | Converts string to lowercase | `tolower("ABC")` → `abc` |
| `toupper(s)` | Converts string to uppercase | `toupper("abc")` → `ABC` |

**Worked example combining several string functions:**
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

### 8.4 Input / Output Commands

| Command | Purpose |
|---|---|
| `close(file)` | Closes an open file or pipe |
| `fflush()` | Flushes buffered output |
| `getline` | Reads the next line of input into `$0` (or a variable) |
| `next` | Skips remaining code and moves to the **next record** |
| `nextfile` | Skips remaining input in the current file and moves to the next file *(commonly written `nextfile`, sometimes referred to loosely as "nextline" in course slides)* |
| `print` | Outputs data, using `OFS` and `ORS` |
| `printf` | Outputs formatted data (C-style formatting) |

> **Correction / typo fix:** The source slide lists a command called *"nextline"*. The correct, standard AWK/POSIX command is **`nextfile`**, which skips the rest of the current input file and moves on to the next file. There is no built-in command literally named `nextline`; this has been corrected here for exam accuracy.

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

**Example — `delete`:**
```bash
awk 'BEGIN{ arr["a"]=1; delete arr["a"] }'
```

---

## 9. Arrays in AWK

**Concept:**
AWK provides **associative arrays** — arrays indexed by arbitrary strings (or numbers), not just sequential integers.

**Key properties:**

| Property | Explanation |
|---|---|
| Associative | Every array in AWK is a key–value (associative) structure |
| Sparse storage | Only the indices you actually assign are stored — there are no "empty slots" to worry about |
| Non-integer index | The index does **not** need to be an integer — it can be any string |
| Assignment syntax | `arr[index] = value` |
| Iteration syntax | `for (var in arr) { ... }` |
| Deletion syntax | `delete arr[index]` |

**Syntax summary:**
```awk
arr[index] = value        # assign
for (var in arr) { ... }  # iterate over all indices
delete arr[index]         # remove one element
delete arr                # remove entire array (gawk extension)
```

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

**Example — counting word frequency using an array (very common exam question):**
```bash
awk '{ for (i=1; i<=NF; i++) count[$i]++ }
     END { for (word in count) print word, count[word] }' textfile.txt
```
*Explanation:* For every word (`$i`) in every record, the array `count` is incremented. At the end, all words and their frequencies are printed.

---

## 10. Loops and Conditionals

**Concept:**
AWK supports the same fundamental control-flow constructs found in C-like languages.

### Syntax of All Loop/Conditional Types

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

### Explanation Table

| Construct | Runs Body | Typical Use |
|---|---|---|
| `for (a in array)` | Once per index in the array | Iterating over associative arrays |
| `for (i=1;i<n;i++)` | While the condition is true, with an update step each time | Counting loops, fixed number of iterations |
| `if / else` | Only if the condition is true | Conditional branching |
| `while` | While the condition is true (checked **before** each iteration) | Loops where the condition might be false at the start |
| `do ... while` | **At least once**, then while the condition remains true | Loops where the body must run at least one time |

**Example — `if / else`:**
```bash
awk '{ if ($2 > 25) print $1, "Senior"; else print $1, "Junior" }' data.txt
```

**Example — `while` loop:**
```bash
awk 'BEGIN{ i=1; while (i<=5) { print i; i++ } }'
```

**Example — `do...while` loop:**
```bash
awk 'BEGIN{ i=1; do { print i; i++ } while (i<=5) }'
```

**Example — `for` loop over fields:**
```bash
awk '{ for (i=1; i<=NF; i++) print "Field", i, "=", $i }' data.txt
```

---

## 11. User-Defined Functions

**Concept:**
Besides built-in functions, AWK allows programmers to define their **own** functions, which can be reused across scripts and even shared via separate "library" files.

**Syntax:**
```awk
function function_name(parameters)
{
    # function body
    return value   # optional
}
```

**Combining multiple script files (a library + a main script):**
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

**Example — a simple standalone user-defined function:**
```bash
awk 'function square(x) { return x*x } BEGIN{ print square(5) }'
```
Output:
```
25
```

---

## 12. Pretty Printing with printf

**Concept:**
While `print` gives simple output, `printf` gives **fine-grained, C-style formatted output** — essential for aligning columns, controlling decimal places, and producing report-style output.

**Syntax:**
```awk
printf "format", a, b, c
```

The **format string** contains one or more format specifiers of the form:
```
%[modifier]control-letter
```

### 12.1 Format Specifiers (Control Letters)

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

### 12.2 Width and Precision Modifiers

| Modifier | Meaning |
|---|---|
| `width` | Minimum field width (pads with spaces, or zeros if `0` prefix is used) |
| `prec` (precision) | For floats: number of digits after the decimal point; for strings: maximum number of characters printed |

**Example — basic `printf`:**
```bash
awk 'BEGIN{ printf "%s is %d years old\n", "Alice", 25 }'
```
Output:
```
Alice is 25 years old
```

**Example — controlling width and precision:**
```bash
awk 'BEGIN{ printf "%-10s%5.2f\n", "Price:", 3.14159 }'
```
Output:
```
Price:      3.14
```
*Explanation:* `%-10s` left-aligns the string in a 10-character-wide field; `%5.2f` right-aligns the float in a 5-character field with 2 digits after the decimal point.

**Example — hexadecimal and octal output:**
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

---

## 13. Combining Bash and AWK

**Concept:**
AWK is rarely used in complete isolation — it is most powerful when combined with shell scripting.

**Common integration techniques:**

| Technique | Description |
|---|---|
| Including AWK inside a shell script | An AWK command or script is embedded directly inside a `.sh` file |
| Heredoc feature | Multi-line AWK programs are embedded in bash using `<<` heredoc syntax, avoiding the need for a separate `.awk` file |
| Piping with other commands | AWK is combined with other shell utilities (`grep`, `sort`, `cut`, etc.) on the command line using the pipe operator |

**Example — AWK inside a bash script using heredoc:**
```bash
#!/bin/bash
awk << 'EOF'
BEGIN { print "Report generated from heredoc" }
EOF
```

**Example — chaining AWK with other Unix tools:**
```bash
cat access.log | grep "ERROR" | awk '{ print $1, $NF }' | sort
```
*Explanation:* `grep` filters only lines containing "ERROR", `awk` extracts the first field and the last field (`$NF` = last field of the record), and `sort` orders the final output.

---

## 14. Why AWK Matters

**Concept (closing remark from the source material):**

> "AWK is available everywhere! AWK is a programming language, quick to code and fast in execution. Combine it on the command line with other scripts."

**Key exam takeaways from this statement:**

| Point | Explanation |
|---|---|
| Universally available | AWK (or a variant like `gawk`/`mawk`) is installed by default on virtually every Unix/Linux system |
| Quick to code | Its concise pattern–action syntax means small text-processing tasks can be written in a single line |
| Fast in execution | AWK is implemented efficiently and is well-suited for processing large text files quickly |
| Composable | It integrates smoothly with shell pipelines and other command-line tools |

---

## 15. Summary

- **AWK** is a POSIX-standard programming language (IEEE 1003.1-2008) created by **A**ho, **W**einberger, and **K**ernighan, designed to process text organized into **records** and **fields**.
- Its **execution model** automatically splits input into records (default: lines) and fields (default: whitespace-separated words), and runs matching pattern–action blocks on each record.
- AWK can be invoked **directly on the command line** (`awk -F"..." '{...}'`) or as a **standalone script** with a shebang line (`#!/usr/bin/gawk -f`).
- A rich set of **built-in variables** (`NR`, `FNR`, `NF`, `FS`, `OFS`, `RS`, `ORS`, `$0`, `$n`, etc.) describe the current parsing state without needing manual bookkeeping.
- Scripts are structured as **pattern { action }** pairs, with special blocks `BEGIN` (runs once before input) and `END` (runs once after input) for setup and summary tasks.
- AWK provides a full set of **assignment, logical, arithmetic, relational,** and **special operators** (ternary `?:`, array membership `in`, regex match `~`/`!~`, increment/decrement, field reference `$`, and string concatenation by juxtaposition).
- A large **standard library of functions** is available: arithmetic (`sqrt`, `int`, `rand`, ...), string (`substr`, `split`, `gsub`, `sub`, `length`, ...), control flow (`if`, `for`, `while`, `break`, `continue`, `return`), I/O (`print`, `printf`, `getline`, `next`, `nextfile`), and extensions (`delete`, `function`, `system`, bit-wise operations).
- **Arrays** in AWK are **associative** and **sparse**, allowing any string as an index, with `for (var in array)` for iteration and `delete` for removal.
- All standard **loop constructs** (`for`, `for-in`, `while`, `do-while`) and **conditionals** (`if/else`) are supported, mirroring C-like syntax.
- **User-defined functions** let programmers build reusable logic, which can even be split across multiple files and combined using multiple `-f` flags.
- **`printf`** enables precise, C-style formatted output using control letters (`d`, `s`, `f`, `x`, `o`, etc.) along with width and precision modifiers — essential for producing clean, tabular reports.
- AWK integrates naturally with **bash**, either embedded directly in shell scripts (including via heredocs) or chained with other Unix tools using pipes.
- Overall, AWK's greatest strengths are that it is **universally available**, **quick to write**, and **fast to execute**, making it a core tool for text processing on any Unix-like system.

---

*End of Notes — Week 9, AWK Programming, Lecture 1 & 2 (Part 1 & Part 2)*
