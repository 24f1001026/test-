# Shell Programming — Bash Scripting (Part 2, 3 & 4)
### Week 7 | Lectures 1, 2 & 3 | Advanced Bash Script Features
### Exam Preparation Notes

---

## Table of Contents

1. [Debugging Bash Scripts](#1-debugging-bash-scripts)
2. [Combining Conditions](#2-combining-conditions)
3. [Shell Arithmetic](#3-shell-arithmetic)
   - 3.1 [Using `let`](#31-using-let)
   - 3.2 [Using `expr`](#32-using-expr)
   - 3.3 [Using `$(( expression ))`](#33-using--expression-)
   - 3.4 [Using `$[ expression ]`](#34-using--expression--1)
4. [`expr` Command Operators (Reference Tables)](#4-expr-command-operators-reference-tables)
   - 4.1 [Arithmetic & Relational Operators](#41-arithmetic--relational-operators)
   - 4.2 [Logical & String Operators](#42-logical--string-operators)
5. [The Heredoc Feature](#5-the-heredoc-feature)
6. [Conditional Statements](#6-conditional-statements)
   - 6.1 [`if-elif-else-fi`](#61-if-elif-else-fi)
   - 6.2 [`case` Statement](#62-case-statement)
7. [Loops in Bash](#7-loops-in-bash)
   - 7.1 [C-Style `for` Loop — One Variable](#71-c-style-for-loop--one-variable)
   - 7.2 [C-Style `for` Loop — Two Variables](#72-c-style-for-loop--two-variables)
   - 7.3 [Redirecting Loop Output](#73-redirecting-loop-output)
8. [Loop Control Statements](#8-loop-control-statements)
   - 8.1 [`break`](#81-break)
   - 8.2 [`break n` (Breaking Nested Loops)](#82-break-n-breaking-nested-loops)
   - 8.3 [`continue`](#83-continue)
9. [Positional Parameters — `shift`](#9-positional-parameters--shift)
10. [The `exec` Command](#10-the-exec-command)
11. [The `eval` Command](#11-the-eval-command)
12. [Parsing Options — `getopts`](#12-parsing-options--getopts)
13. [The `select` Loop (Text Menus)](#13-the-select-loop-text-menus)
14. [Overall Summary](#14-overall-summary)

---

## 1. Debugging Bash Scripts

### Concept
Debugging lets you trace exactly what a script is doing, step by step, by printing each command **before** it is executed. This is essential for finding logic errors, incorrect variable expansions, or unexpected control flow.

### Syntax
There are three common ways to enable debug (trace) mode:

```bash
# Method 1: run script normally, but pass -x while calling bash
bash -x ./myscript.sh

# Method 2: same as above, alternate invocation
./myscript.sh   # (script must have set -x inside, see Method 3)

# Method 3: place set -x inside the script itself
set -x
# ... rest of the script
```

| Method | How it works |
|---|---|
| `bash -x ./myscript.sh` | Runs the whole script in debug mode from the command line, without editing the file |
| `set -x` inside script | Turns on tracing from that line onward (useful to debug only a portion of the script) |
| `set +x` inside script | Turns **off** tracing (used after `set -x` to stop tracing a section) |

### Example
```bash
#!/bin/bash
set -x
a=5
b=10
sum=$(( a + b ))
echo "Sum is $sum"
set +x
```
**Output (trace shown with `+` prefix):**
```
+ a=5
+ b=10
+ sum=15
+ echo 'Sum is 15'
Sum is 15
```

> 💡 **Exam Tip:** `set -x` prints commands *with variable values substituted*, prefixed by `+`, which makes it different from simply reading the script's source code.

---

## 2. Combining Conditions

### Concept
Bash allows combining multiple test conditions (`[ ]`) using logical **AND** (`&&`) and logical **OR** (`||`) operators, similar to combining conditions in high-level programming languages.

### Syntax
```bash
[ condition1 ] && [ condition2 ]     # AND — both must be true
[ condition1 ] || [ condition2 ]     # OR  — at least one must be true
```

### Example
```bash
a=5

# AND example
if [ $a -gt 3 ] && [ $a -gt 7 ]
then
    echo "a is greater than 3 AND greater than 7"
else
    echo "Condition failed"
fi

# OR example
if [ $a -lt 3 ] || [ $a -gt 7 ]
then
    echo "a is less than 3 OR greater than 7"
else
    echo "a is between 3 and 7"
fi
```
**Output:**
```
Condition failed
a is between 3 and 7
```

---

## 3. Shell Arithmetic

### Concept
Bash provides **four** different ways to perform arithmetic operations on integers: `let`, `expr`, `$(( expression ))`, and `$[ expression ]`. Each has slightly different syntax rules.

| Method | Style | Notes |
|---|---|---|
| `let` | `let a=$1+5` | No spaces needed around operators; quotes optional |
| `expr` | `expr $a + 20` | Spaces **mandatory** around operators and operands |
| `$(( ))` | `b=$(( $a + 10 ))` | Most common/modern method; spacing flexible |
| `$[ ]` | `b=$[ $a + 10 ]` | Older/deprecated equivalent of `$(( ))` |

### 3.1 Using `let`

**Syntax:**
```bash
let a=$1+5
let "a = $1 + 5"
```

**Example:**
```bash
#!/bin/bash
# Run as: ./script.sh 10
let a=$1+5
echo "Value of a: $a"

let "b = $1 + 5"
echo "Value of b: $b"
```
**Output (for input 10):**
```
Value of a: 15
Value of b: 15
```

### 3.2 Using `expr`

**Syntax:**
```bash
expr $a + 20
expr "$a + 20"
b=$( expr $a + 20 )
```

> ⚠️ **Important:** `expr` requires **spaces** between the operator and the operands, otherwise it will treat the whole string as a single token and throw an error.

**Example:**
```bash
a=5
expr $a + 20        # prints 25 directly to terminal
b=$( expr $a + 20 ) # captures result into variable b
echo "b is $b"
```
**Output:**
```
25
b is 25
```

### 3.3 Using `$(( expression ))`

**Syntax:**
```bash
b=$(( $a + 10 ))
(( b++ ))
```

This is the **most widely used and recommended** method for arithmetic in modern Bash scripts. It also supports increment (`++`), decrement (`--`), and compound assignment operators.

**Example:**
```bash
a=5
b=$(( a + 10 ))
echo "b is $b"        # 15

(( b++ ))
echo "b after increment: $b"   # 16
```

### 3.4 Using `$[ expression ]`

**Syntax:**
```bash
b=$[ $a + 10 ]
```

This is an **older, deprecated** syntax that behaves like `$(( ))`. It still works in Bash but is not recommended for new scripts.

**Example:**
```bash
a=5
b=$[ $a + 10 ]
echo "b is $b"   # 15
```

---

## 4. `expr` Command Operators (Reference Tables)

Because the `expr` command supports many operators, they are split here into two categories for clarity, as required for detailed exam reference.

### 4.1 Arithmetic & Relational Operators

| Operator | Description |
|---|---|
| `a + b` | Returns the arithmetic sum of `a` and `b` |
| `a - b` | Returns the arithmetic difference of `a` and `b` |
| `a * b` | Returns the arithmetic product of `a` and `b` |
| `a / b` | Returns the arithmetic quotient of `a` divided by `b` |
| `a % b` | Returns the arithmetic remainder of `a` divided by `b` |
| `a > b` | Returns 1 if `a` is greater than `b`; else returns 0 |
| `a >= b` | Returns 1 if `a` is greater than or equal to `b`; else returns 0 |
| `a < b` | Returns 1 if `a` is less than `b`; else returns 0 |
| `a <= b` | Returns 1 if `a` is less than or equal to `b`; else returns 0 |
| `a = b` | Returns 1 if `a` equals `b`; else returns 0 |

**Example:**
```bash
a=10
b=20
expr $a + $b     # 30
expr $a \* $b    # 200 (note: * must be escaped with backslash for expr)
expr $b % $a     # 0
expr $a \< $b    # 1 (a is less than b)
```

### 4.2 Logical & String Operators

| Operator | Description |
|---|---|
| `a \| b` | Returns `a` if neither argument is null or 0; else returns `b` |
| `a & b` | Returns `a` if neither argument is null or 0; else returns 0 |
| `a != b` | Returns 1 if `a` is not equal to `b`; else returns 0 |
| `str : reg` | Returns the position up to the anchored pattern match with a Basic Regular Expression (BRE) `str` |
| `match str reg` | Returns the pattern match if `reg` matches the pattern in `str` |
| `substr str n m` | Returns the substring `m` characters in length, starting at position `n` |
| `index str chars` | Returns the position in `str` where any one of `chars` is first found; else returns 0 |
| `length str` | Returns the numeric length of the string `str` |
| `+ token` | Interprets `token` as a string even if it is a keyword |
| `(exprn)` | Returns the value of the expression `exprn` (used for grouping/precedence) |

**Example:**
```bash
str="HelloWorld"

expr length "$str"            # 10
expr substr "$str" 1 5        # Hello
expr index "$str" "oW"        # 5 (position of first match: 'o')
```

> 📝 **Note:** `str : reg` and `match str reg` use **Basic Regular Expressions (BRE)**, not extended regex, so special characters like `+`, `?`, `|` need escaping.

---

## 5. The Heredoc Feature

### Concept
A **heredoc** (`<<`) lets you feed multiple lines of input directly into a command (commonly `bc`, `cat`, `mail`, etc.) without needing a separate file. It is extremely useful for feeding multi-line scripts to calculators like `bc`.

### Syntax
```bash
command << MARKER
line 1
line 2
MARKER
```
- The **marker** (delimiter) can be **any word** — it does not have to be `EOF`.
- Using `<<-` (hyphen) instead of `<<` tells Bash to **ignore leading tab characters** in the heredoc body — useful when the heredoc is indented inside a function or loop.

### Example 1 — Basic Heredoc with `EOF`
```bash
a=2.5
b=3.2
c=4
d=$(bc -l << EOF
scale = 5
($a+$b)^$c
EOF
)
echo $d
```
**Output:**
```
1055.6001
```

### Example 2 — Heredoc with Custom Marker and Leading-Tab Ignore
```bash
a=2.5
b=3.2
c=4
d=$(bc -l <<- ABC
	scale = 5
	($a+$b)^$c
	ABC
)
echo $d
```
**Output:**
```
1055.6001
```

> 💡 **Exam Tip:** The marker name (`EOF`, `ABC`, or any custom word) must appear **alone on its own line** to close the heredoc, and it must match exactly (case-sensitive) with the opening marker.

---

## 6. Conditional Statements

### 6.1 `if-elif-else-fi`

#### Concept
Used for multi-way branching based on conditions, evaluated top to bottom. As soon as one condition is true, its command block executes and the rest are skipped.

#### Syntax
```bash
# Simple if-else
if condition1
then
    commandset1
else
    commandset2
fi

# if-elif-else chain
if condition1
then
    commandset1
elif condition2
then
    commandset2
elif condition3
then
    commandset3
else
    commandset4
fi
```

#### Example
```bash
#!/bin/bash
marks=75

if [ $marks -ge 90 ]
then
    echo "Grade: A"
elif [ $marks -ge 75 ]
then
    echo "Grade: B"
elif [ $marks -ge 60 ]
then
    echo "Grade: C"
else
    echo "Grade: F"
fi
```
**Output:**
```
Grade: B
```

### 6.2 `case` Statement

#### Concept
The `case` statement is Bash's equivalent of `switch` in other languages. It compares a variable against several patterns and executes the matching block. Multiple values can be grouped for a single block using `|` (pipe/OR). `*` acts as the **default/catch-all** case.

#### Syntax
```bash
case $var in
    op1)
        commandset1;;
    op2 | op3)
        commandset2;;
    op4 | op5 | op6)
        commandset3;;
    *)
        commandset4;;
esac
```
> 📝 **Note:** `commandset4` (the `*` pattern) is the **default** block, executed only when `$var` does not match any of the listed patterns.

#### Example
```bash
#!/bin/bash
read -p "Enter a day number (1-7): " day

case $day in
    1)
        echo "Monday";;
    2 | 3 | 4 | 5)
        echo "Mid-week day";;
    6 | 7)
        echo "Weekend";;
    *)
        echo "Invalid day number";;
esac
```
**Sample run (input 6):**
```
Weekend
```

---

## 7. Loops in Bash

### 7.1 C-Style `for` Loop — One Variable

#### Concept
Bash supports a C-language-style `for` loop with an initializer, a condition, and an increment/decrement expression, all inside double parentheses `(( ))`.

#### Syntax
```bash
for (( initializer; condition; increment ))
do
    commands
done
```

#### Example
```bash
begin=1
finish=10
for (( a = $begin; a < $finish; a++ ))
do
    echo $a
done
```
**Output:**
```
1
2
3
4
5
6
7
8
9
```

### 7.2 C-Style `for` Loop — Two Variables

#### Concept
The C-style `for` loop can track **two (or more) variables simultaneously**, each with its own initializer and increment/decrement, separated by commas. However, only **one condition** is allowed to control loop termination.

#### Syntax
```bash
for (( var1=init1, var2=init2; condition; var1++, var2-- ))
do
    commands
done
```

> ⚠️ **Exam Tip:** Even with multiple variables, the loop uses **only one condition** to decide when to stop — this is a commonly asked exam point.

#### Example
```bash
begin1=1
begin2=10
finish=10
for (( a=$begin1, b=$begin2; a < $finish; a++, b-- ))
do
    echo $a $b
done
```
**Output:**
```
1 10
2 9
3 8
4 7
5 6
6 5
7 4
8 3
9 2
```

### 7.3 Redirecting Loop Output

#### Concept
The **entire output** of a loop can be redirected to a file by placing the redirection operator (`>` or `>>`) right after the `done` keyword — this redirects the combined output of every iteration, not just the last one.

#### Syntax
```bash
for (( initializer; condition; increment ))
do
    commands
done > filename
```

#### Example
```bash
filename=tmp.$$
begin=1
finish=10
for (( a = $begin; a < $finish; a++ ))
do
    echo $a
done > $filename

cat $filename
```
**Explanation:** `tmp.$$` creates a unique temp filename using the current shell's Process ID (`$$`). All numbers 1–9 are written into this file instead of being printed to the screen.

---

## 8. Loop Control Statements

### 8.1 `break`

#### Concept
`break` immediately exits the **innermost** enclosing loop, skipping any remaining iterations.

#### Syntax
```bash
while condition
do
    commands
    if some_condition
    then
        break
    fi
done
```

#### Example
```bash
n=10
i=0
while [ $i -lt $n ]
do
    echo $i
    (( i++ ))
    if [ $i -eq 5 ]
    then
        break
    fi
done
```
**Output:**
```
0
1
2
3
4
```

### 8.2 `break n` (Breaking Nested Loops)

#### Concept
By default, `break` only exits the innermost loop. To break out of **multiple nested loops** at once, you can specify a number `n` with `break`, indicating how many levels of enclosing loops to exit.

#### Syntax
```bash
break n     # breaks out of n levels of enclosing loops
```

#### Example — Breaking Out of the Inner Loop Only (`break`)
```bash
n=10
i=0
while [ $i -lt $n ]
do
    echo $i
    j=0
    while [ $j -le $i ]
    do
        printf "$j "
        (( j++ ))
        if [ $j -eq 7 ]
        then
            break
        fi
    done
    (( i++ ))
done
```

#### Example — Breaking Out of Both Loops (`break 2`)
```bash
n=10
i=0
while [ $i -lt $n ]
do
    echo $i
    j=0
    while [ $j -le $i ]
    do
        printf "$j "
        (( j++ ))
        if [ $j -eq 7 ]
        then
            break 2
        fi
    done
    (( i++ ))
done
```
**Output:**
```
0
0 1
0 1 2
0 1 2 3
0 1 2 3 4
0 1 2 3 4 5
0 1 2 3 4 5 6
0 1 2 3 4 5 6
```
> 📝 As soon as the inner counter `j` reaches 7, `break 2` exits **both** the inner and outer `while` loops immediately — this is why the output stops after the 7th line even though `n=10`.

### 8.3 `continue`

#### Concept
`continue` skips the **remaining commands** in the current loop iteration and jumps straight to the next iteration's condition check — it does **not** exit the loop.

#### Syntax
```bash
while condition
do
    if some_condition
    then
        continue
    fi
    commands
done
```

#### Example
```bash
n=9
i=0
while [ $i -lt $n ]
do
    printf "\n loop $i:"
    j=0
    (( i++ ))
    while [ $j -le $i ]
    do
        (( j++ ))
        if [ $j -gt 3 ] && [ $j -lt 6 ]
        then
            continue
        fi
        printf "$j "
    done
done
```
**Output:**
```
loop 0:1 2 
loop 1:1 2 3 
loop 2:1 2 3 
loop 3:1 2 3 
loop 4:1 2 3 6 
loop 5:1 2 3 6 7 
loop 6:1 2 3 6 7 8 
loop 7:1 2 3 6 7 8 9 
loop 8:1 2 3 6 7 8 9 10
```
> 📝 Values `4` and `5` are skipped in every inner loop's printed sequence because the `continue` statement fires whenever `j` is greater than 3 **and** less than 6, jumping past the `printf` line for those values.

---

## 9. Positional Parameters — `shift`

### Concept
`shift` shifts all **command-line arguments** (positional parameters `$1, $2, $3, ...`) one position to the **left**. After a `shift`, what was `$2` becomes `$1`, `$3` becomes `$2`, and so on. `$1` before the shift is discarded.

### Syntax
```bash
shift          # shifts by 1 (default)
shift n        # shifts by n positions
```

### Example
```bash
#!/bin/bash
# Run as: ./script.sh apple banana cherry
i=1
while [ -n "$1" ]
do
    echo "argument $i is $1"
    shift
    (( i++ ))
done
```
**Output (for `apple banana cherry`):**
```
argument 1 is apple
argument 2 is banana
argument 3 is cherry
```
> 💡 **Exam Tip:** `[ -n "$1" ]` checks whether `$1` is a **non-empty string** — the loop naturally ends once all arguments have been shifted out and `$1` becomes empty.

---

## 10. The `exec` Command

### Concept
`exec` **replaces** the current shell process with a new program, instead of creating a child process. Key behavior points:
- If the new program launches successfully, control **never returns** to the original shell/script (because the shell itself has been replaced).
- If the new program **fails** to launch, the original shell continues executing normally.
- `exec` is also used to **change I/O redirection settings** for the remainder of a script.

### Syntax
```bash
exec ./my-executable --my-options --my-args
```

### Example
```bash
#!/bin/bash
echo "About to replace this shell..."
exec ls -l /home
echo "This line will NEVER execute if exec succeeds"
```
**Explanation:** Once `exec ls -l /home` runs successfully, the shell process is replaced entirely by `ls`, so the final `echo` statement never runs.

---

## 11. The `eval` Command

### Concept
`eval` takes its arguments, **combines them into a single string**, and then executes that string **as a shell command**. Unlike `exec`, `eval` **returns control** back to the shell once the command finishes (returning an exit status).

### Syntax
```bash
eval my-arg
```

### Example
```bash
cmd="echo"
arg="Hello, World!"
eval $cmd $arg
```
**Output:**
```
Hello, World!
```

**Practical use-case example (dynamic variable name):**
```bash
varname="greeting"
eval $varname="HelloThere"
eval echo \$$varname
```
**Output:**
```
HelloThere
```
> 📝 `eval` is often used when a variable itself holds the **name of another variable or a command**, and you need Bash to resolve it dynamically at runtime.

---

## 12. Parsing Options — `getopts`

### Concept
`getopts` is used to parse **command-line options** (flags) passed to a script, similar to how standard UNIX commands accept flags like `-a`, `-b value`. It differentiates between:
- Options that take **no argument** (e.g., `a`)
- Options that **require an argument** (indicated by a following colon `:`, e.g., `b:`)

### Syntax
```bash
while getopts "ab:c:" options
do
    case "${options}" in
        a)
            # option -a (no argument)
            ;;
        b)
            barg=${OPTARG}
            ;;
        c)
            carg=${OPTARG}
            ;;
        *)
            echo "Usage: -a -b barg -c carg"
            ;;
    esac
done
```
- `"ab:c:"` is the **option string**: `a` takes no argument; `b:` and `c:` each require an argument (the colon after a letter means "this option needs a value").
- `${OPTARG}` automatically holds the argument value supplied for the current option.

### Example
```bash
#!/bin/bash
while getopts "ab:c:" options
do
    case "${options}" in
        b)
            barg=${OPTARG}
            echo "accepted: -b $barg"
            ;;
        c)
            carg=${OPTARG}
            echo "accepted: -c $carg"
            ;;
        a)
            echo "accepted: -a"
            ;;
        *)
            echo "Usage: -a -b barg -c carg"
            ;;
    esac
done
```
**Sample run:**
```bash
./script.sh -a -b hello -c world
```
**Output:**
```
accepted: -a
accepted: -b hello
accepted: -c world
```
> ⚠️ **Exam Tip:** This script can be invoked **only** with the three defined options — `a`, `b`, and `c`. Any other flag falls into the `*` (default/usage) case.

---

## 13. The `select` Loop (Text Menus)

### Concept
The `select` loop is a built-in Bash construct specifically designed to build **interactive numbered text menus**. It automatically displays a numbered list of choices, prompts the user for input, and repeats until explicitly stopped (typically with `break`).

### Syntax
```bash
select variable in list_of_options
do
    commands   # normally includes a case statement + break
done
```

### Example
```bash
echo "Select a middle one"
select i in {1..10}
do
    case $i in
        1 | 2 | 3)
            echo "you picked a small one";;
        8 | 9 | 10)
            echo "you picked a big one";;
        4 | 5 | 6 | 7)
            echo "you picked the right one"
            break;;
    esac
done
echo "selection completed with $i"
```
**Explanation of behavior:**
- `select i in {1..10}` displays a numbered menu of options 1 through 10.
- The user's numeric choice is stored in `$i`.
- If the user picks a "small" (1–3) or "big" (8–10) number, a message is shown but the **menu repeats** (no `break`).
- Only when the user picks a "right" number (4–7) does the loop `break` and terminate.

**Sample run (user selects 5):**
```
Select a middle one
1) 1   3) 3   5) 5   7) 7   9) 9
2) 2   4) 4   6) 6   8) 8   10) 10
#? 5
you picked the right one
selection completed with 5
```

> 💡 **Text Menu Tip:** `select` is one of the most exam-relevant constructs for building simple CLI menus — remember that it **loops indefinitely** unless a `break` (or similar exit mechanism) is included inside the loop body.

---

## 14. Overall Summary

This set of notes (Week 7, Lectures 1–3, Bash Scripting Part 2/3/4) covers the **advanced building blocks** needed to write professional, production-quality Bash scripts. Key takeaways, in the recommended learning order:

1. **Debugging (`set -x` / `bash -x`)** — Always debug before submitting/using a script; trace execution line by line.
2. **Combining Conditions (`&&`, `||`)** — Build compound logical tests without nested `if` statements.
3. **Shell Arithmetic** — Four methods exist (`let`, `expr`, `$(( ))`, `$[ ]`); **`$(( ))` is the modern standard**, while `expr` needs careful spacing and escaping.
4. **`expr` Operators** — A rich operator set exists beyond simple math: relational, logical, and string operators (`length`, `substr`, `index`, `match`).
5. **Heredoc (`<<`)** — Feed multi-line input to commands like `bc`; use `<<-` to strip leading tabs; marker name is flexible.
6. **Conditionals** — `if-elif-else-fi` for range/threshold-based logic; `case` for clean multi-way pattern matching with `|` for grouping and `*` for defaults.
7. **Loops** — The C-style `for (( ))` loop supports single or multiple counter variables (but only **one** stopping condition); loop output can be redirected as a whole using `done > file`.
8. **Loop Control** — `break` exits the loop entirely (use `break n` for nested loops); `continue` skips only the current iteration, not the whole loop.
9. **`shift`** — Essential for processing an unknown/variable number of command-line arguments one at a time.
10. **`exec` vs `eval`** — `exec` **replaces** the running shell (no return unless it fails); `eval` **executes a built string as a command** and always returns control back to the shell.
11. **`getopts`** — The standard, professional way to parse `-flag value` style command-line options in scripts.
12. **`select`** — Built specifically for building interactive, numbered text menus in the terminal.

> ✅ **Final Exam Tip:** Practice writing each construct (`if`, `case`, `for`, `while`, `getopts`, `select`) from memory, and be ready to **trace through nested loop output** by hand (especially `break n` and `continue` examples), since these are the most common types of "predict the output" questions.

---

*End of Notes — Bash Scripting Part 2, 3 & 4 (Week 7)*
