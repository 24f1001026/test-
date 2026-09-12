# Shell Programming — Bash Scripting (Part 2, 3 & 4)
### Week 7 | Lectures 1, 2 & 3 | Advanced Bash Script Features
### Detailed Exam Preparation Notes (Expanded Edition)

---

## Table of Contents

1. [Debugging Bash Scripts](#1-debugging-bash-scripts)
2. [Combining Conditions](#2-combining-conditions)
   - 2.1 [Using `[[ ]]` with `&&` and `||` (Extended Test)](#21-using--with--and--extended-test)
3. [Shell Arithmetic](#3-shell-arithmetic)
   - 3.1 [Using `let`](#31-using-let)
   - 3.2 [Using `expr`](#32-using-expr)
   - 3.3 [Using `$(( expression ))`](#33-using--expression-)
   - 3.4 [Using `$[ expression ]`](#34-using--expression--1)
   - 3.5 [Comparison of All Four Methods](#35-comparison-of-all-four-methods)
4. [`expr` Command Operators (Reference Tables)](#4-expr-command-operators-reference-tables)
   - 4.1 [Arithmetic & Relational Operators — with Examples of Each](#41-arithmetic--relational-operators--with-examples-of-each)
   - 4.2 [Logical & String Operators — with Examples of Each](#42-logical--string-operators--with-examples-of-each)
   - 4.3 [Pattern Matching with `str =~ regex` Inside `[[ ]]`](#43-pattern-matching-with-str--regex-inside-)
5. [The Heredoc Feature](#5-the-heredoc-feature)
   - 5.1 [`bc` — The Bench Calculator](#51-bc--the-bench-calculator)
   - 5.2 [Changing the Field Separator — `IFS`](#52-changing-the-field-separator--ifs)
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
   - 8.4 [`continue n` (Nested Continue)](#84-continue-n-nested-continue)
9. [Positional Parameters — `shift`](#9-positional-parameters--shift)
10. [The `exec` Command](#10-the-exec-command)
11. [The `eval` Command](#11-the-eval-command)
12. [Parsing Options — `getopts`](#12-parsing-options--getopts)
13. [The `select` Loop (Text Menus)](#13-the-select-loop-text-menus)
14. [Common Mistakes & Exam Pitfalls](#14-common-mistakes--exam-pitfalls)
15. [Overall Summary](#15-overall-summary)

---

## 1. Debugging Bash Scripts

### Concept
Debugging lets you trace exactly what a script is doing, step by step, by printing each command **before** it is executed (with variables already substituted). This is essential for finding logic errors, incorrect variable expansions, or unexpected control flow — a script may look correct but behave incorrectly if a variable holds an unexpected value, and tracing reveals this immediately.

### Syntax — Three Ways to Enable Debug (Trace) Mode

```bash
# Method 1: from the terminal, without editing the script file
bash -x ./myscript.sh

# Method 2: make the script executable and pass -x as an option
./myscript.sh          # only works if set -x is already inside the script

# Method 3: place set -x inside the script itself
set -x
# ... commands to trace ...
set +x                 # turns tracing back OFF
```

| Method | How it works | When to use |
|---|---|---|
| `bash -x ./myscript.sh` | Traces the **entire** script from a fresh shell, no file editing needed | Quick one-off debugging session |
| `set -x` inside script | Turns on tracing from that exact line onward | Debug only a **specific section** of a long script |
| `set +x` inside script | Turns tracing **off** again | Stop tracing after the problem area, to avoid a noisy full trace |

### Example 1 — Whole Script Debug via Command Line
`myscript.sh`:
```bash
#!/bin/bash
a=5
b=10
sum=$(( a + b ))
echo "Sum is $sum"
```
Run with:
```bash
bash -x ./myscript.sh
```
**Output:**
```
+ a=5
+ b=10
+ sum=15
+ echo 'Sum is 15'
Sum is 15
```

### Example 2 — Debugging Only Part of a Script (`set -x` / `set +x`)
```bash
#!/bin/bash
echo "Start of script (not traced)"
a=100

set -x                      # tracing begins here
b=200
total=$(( a + b ))
set +x                      # tracing ends here

echo "Total is $total (not traced)"
```
**Output:**
```
Start of script (not traced)
+ b=200
+ total=300
+ set +x
Total is 300 (not traced)
```

### Example 3 — Debugging a Loop to Find a Logic Bug
```bash
#!/bin/bash
set -x
for (( i = 1; i <= 3; i++ ))
do
    echo "Iteration: $i"
done
set +x
```
**Output:**
```
+ (( i = 1 ))
+ (( i <= 3 ))
+ echo 'Iteration: 1'
Iteration: 1
+ (( i++ ))
+ (( i <= 3 ))
+ echo 'Iteration: 2'
Iteration: 2
+ (( i++ ))
+ (( i <= 3 ))
+ echo 'Iteration: 3'
Iteration: 3
+ (( i++ ))
+ (( i <= 3 ))
+ set +x
```

> 💡 **Exam Tip:** The `+` symbol before each traced line is the default `PS4` prompt. Each `+` shows the **actual command executed**, with variables already expanded to their real values — this is different from just reading the script's source code, which shows `$variable`, not its value.

---

## 2. Combining Conditions

### Concept
Bash allows combining multiple `[ ]` test conditions using logical **AND** (`&&`) and logical **OR** (`||`), instead of writing nested `if` statements. This keeps conditional logic compact and readable, and mirrors how AND/OR work in languages like C or Java.

### Syntax
```bash
[ condition1 ] && [ condition2 ]     # AND — TRUE only if BOTH are true
[ condition1 ] || [ condition2 ]     # OR  — TRUE if AT LEAST ONE is true
```

### Example 1 — AND (`&&`) — Both Conditions Must Be True
```bash
a=5
if [ $a -gt 3 ] && [ $a -gt 7 ]
then
    echo "a is greater than 3 AND greater than 7"
else
    echo "Condition failed"
fi
```
**Output:**
```
Condition failed
```
*(Reason: `a=5` is greater than 3, but NOT greater than 7, so the AND condition fails.)*

### Example 2 — OR (`||`) — At Least One Condition Must Be True
```bash
a=5
if [ $a -lt 3 ] || [ $a -gt 7 ]
then
    echo "a is less than 3 OR greater than 7"
else
    echo "a is between 3 and 7"
fi
```
**Output:**
```
a is between 3 and 7
```

### Example 3 — Combining AND and OR Together (Range Check)
```bash
age=25
if [ $age -ge 18 ] && [ $age -le 60 ]
then
    echo "Eligible (working age)"
else
    echo "Not eligible"
fi
```
**Output:**
```
Eligible (working age)
```

### Example 4 — Using `&&` for Simple Command Chaining (Not Just Conditions)
```bash
mkdir project_folder && cd project_folder && echo "Folder created and entered"
```
*(If `mkdir` fails — e.g., folder already exists — the rest of the chain is skipped, because `&&` only proceeds if the previous command succeeded, i.e., returned exit status 0.)*

> 📝 **Exam Tip:** `&&` and `||` can combine not just `[ ]` test conditions but **any commands**, based on their exit status (0 = success/true, non-zero = failure/false).

### 2.1 Using `[[ ]]` with `&&` and `||` (Extended Test)

#### Concept
`[[ ]]` is Bash's **extended test command**. Unlike the POSIX `[ ]` test, `&&` and `||` can be written **directly inside** a single pair of `[[ ]]` brackets, instead of chaining two separate `[ ]` blocks together. `[[ ]]` is also safer with unquoted variables (no word-splitting/globbing surprises) and additionally supports pattern matching (`==`, `!=` with wildcards) and regex matching (`=~`, covered in Section 4.3).

#### Syntax
```bash
[[ condition1 && condition2 ]]     # AND — both must be true, evaluated in ONE [[ ]]
[[ condition1 || condition2 ]]     # OR  — at least one must be true, evaluated in ONE [[ ]]
```

#### Example 1 — AND Inside a Single `[[ ]]` (Range Check)
```bash
age=25
if [[ $age -ge 18 && $age -le 60 ]]
then
    echo "Eligible (working age)"
else
    echo "Not eligible"
fi
```
**Output:**
```
Eligible (working age)
```

#### Example 2 — OR Inside a Single `[[ ]]`
```bash
a=5
if [[ $a -lt 3 || $a -gt 7 ]]
then
    echo "a is less than 3 OR greater than 7"
else
    echo "a is between 3 and 7"
fi
```
**Output:**
```
a is between 3 and 7
```

#### Example 3 — `[ ]` vs `[[ ]]` Side by Side

| Feature | `[ ]` (POSIX test) | `[[ ]]` (Bash extended test) |
|---|---|---|
| Combine conditions | `[ c1 ] && [ c2 ]` (two brackets) | `[[ c1 && c2 ]]` (one bracket) |
| Unquoted variable with spaces | Can break/error | Safe, no word-splitting |
| Wildcard pattern matching | Not supported | `[[ $f == *.txt ]]` supported |
| Regex matching | Not supported | `[[ $s =~ regex ]]` supported (Section 4.3) |
| Portability | POSIX — works in `sh` too | Bash/ksh/zsh only |

> 💡 **Exam Tip:** `[ c1 ] && [ c2 ]` and `[[ c1 && c2 ]]` are logically equivalent and both commonly appear in exams — recognize both forms, but prefer `[[ ]]` in your own Bash-specific scripts.

---

## 3. Shell Arithmetic

### Concept
Bash provides **four** different ways to perform arithmetic operations on integers: `let`, `expr`, `$(( expression ))`, and `$[ expression ]`. Each has slightly different syntax rules regarding spacing, quoting, and modern usage recommendations.

### 3.1 Using `let`

#### Concept
`let` evaluates an arithmetic expression and assigns the result directly to a variable. Spaces around operators are **optional**, but if the expression contains spaces, it **must** be quoted.

#### Syntax
```bash
let a=$1+5
let "a = $1 + 5"
```

#### Example 1 — Basic Addition
```bash
#!/bin/bash
# Run as: ./script.sh 10
let a=$1+5
echo "Value of a: $a"
```
**Output (input 10):**
```
Value of a: 15
```

#### Example 2 — Using Quotes with Spaces
```bash
let "b = $1 + 5"
echo "Value of b: $b"
```
**Output (input 10):**
```
Value of b: 15
```

#### Example 3 — Multiple Operations with `let`
```bash
x=4
let "y = x * x + 2"
echo "y is $y"          # (4*4)+2 = 18
```
**Output:**
```
y is 18
```

#### Example 4 — `let` with Increment/Decrement
```bash
count=0
let count++
let count++
echo "count is $count"    # 2
```
**Output:**
```
count is 2
```

---

### 3.2 Using `expr`

#### Concept
`expr` evaluates its arguments as an expression and prints the result to standard output. It requires **spaces** between every operator and operand — without spaces, `expr` treats the input as a single literal string and fails.

#### Syntax
```bash
expr $a + 20
expr "$a + 20"
b=$( expr $a + 20 )
```

#### Example 1 — Printing Result Directly
```bash
a=5
expr $a + 20
```
**Output:**
```
25
```

#### Example 2 — Capturing Result Into a Variable
```bash
a=5
b=$( expr $a + 20 )
echo "b is $b"
```
**Output:**
```
b is 25
```

#### Example 3 — Common Mistake: Missing Spaces
```bash
a=5
expr $a+20        # WRONG — no spaces around '+'
```
**Output:**
```
5+20
```
*(`expr` cannot evaluate this — it just echoes the literal string back because there is no space to separate tokens.)*

#### Example 4 — Multiplication Needs Escaping
```bash
a=5
expr $a \* 4       # must escape * to avoid shell wildcard expansion
```
**Output:**
```
20
```

#### Example 5 — Using `expr` Inside a Loop Counter (classic legacy style)
```bash
i=1
while [ $i -le 5 ]
do
    echo "i is $i"
    i=$( expr $i + 1 )
done
```
**Output:**
```
i is 1
i is 2
i is 3
i is 4
i is 5
```

---

### 3.3 Using `$(( expression ))`

#### Concept
This is the **modern, most-recommended** method of doing arithmetic in Bash. Spacing is flexible, no escaping of `*` is required, and it supports the full C-style operator set including `++`, `--`, `+=`, `-=`, etc.

#### Syntax
```bash
b=$(( $a + 10 ))
b=$(( a + 10 ))     # $ before variable name is optional inside $(( ))
(( b++ ))
```

#### Example 1 — Basic Arithmetic
```bash
a=5
b=$(( a + 10 ))
echo "b is $b"
```
**Output:**
```
b is 15
```

#### Example 2 — Increment and Decrement
```bash
b=15
(( b++ ))
echo "After increment: $b"      # 16
(( b-- ))
(( b-- ))
echo "After two decrements: $b" # 14
```
**Output:**
```
After increment: 16
After two decrements: 14
```

#### Example 3 — Compound Assignment Operators
```bash
total=100
(( total += 50 ))
echo "Total after += : $total"    # 150
(( total -= 30 ))
echo "Total after -= : $total"    # 120
(( total *= 2 ))
echo "Total after *= : $total"    # 240
```
**Output:**
```
Total after += : 150
Total after -= : 120
Total after *= : 240
```

#### Example 4 — Multiplication and Modulus (No Escaping Needed)
```bash
a=7
b=3
echo $(( a * b ))    # 21
echo $(( a % b ))    # 1
```
**Output:**
```
21
1
```

#### Example 5 — Using `$(( ))` Directly Inside `echo`
```bash
echo "5 squared is $(( 5 * 5 ))"
```
**Output:**
```
5 squared is 25
```

---

### 3.4 Using `$[ expression ]`

#### Concept
This is an **older, deprecated** syntax that behaves the same as `$(( ))`. It is still functional in Bash for backward compatibility, but modern scripts should always prefer `$(( ))` instead.

#### Syntax
```bash
b=$[ $a + 10 ]
```

#### Example 1 — Basic Usage
```bash
a=5
b=$[ $a + 10 ]
echo "b is $b"
```
**Output:**
```
b is 15
```

#### Example 2 — Nested Expression
```bash
a=2
b=3
c=$[ ($a + $b) * 2 ]
echo "c is $c"      # (2+3)*2 = 10
```
**Output:**
```
c is 10
```

> ⚠️ **Exam Tip:** `$[ ]` may still appear in legacy exam questions/scripts — recognize it, but always **prefer `$(( ))`** when writing your own answers, since `$[ ]` is considered obsolete.

### 3.5 Comparison of All Four Methods

| Method | Example Syntax | Spacing Rules | Modern/Recommended? |
|---|---|---|---|
| `let` | `let a=$1+5` | Optional (quote if spaces used) | Yes, still common |
| `expr` | `expr $a + 20` | **Mandatory** spaces around operator | Legacy, but still tested |
| `$(( ))` | `b=$(( $a + 10 ))` | Flexible | ✅ **Best practice / most recommended** |
| `$[ ]` | `b=$[ $a + 10 ]` | Flexible | ❌ Deprecated, avoid in new scripts |

---

## 4. `expr` Command Operators (Reference Tables)

The `expr` command supports a rich set of operators beyond basic math. These are grouped below into two categories for clarity, with a worked example for **every single operator**, as required for detailed exam reference.

### 4.1 Arithmetic & Relational Operators — with Examples of Each

| Operator | Description | Example | Result |
|---|---|---|---|
| `a + b` | Returns the arithmetic sum of `a` and `b` | `expr 10 + 5` | `15` |
| `a - b` | Returns the arithmetic difference of `a` and `b` | `expr 10 - 5` | `5` |
| `a * b` | Returns the arithmetic product of `a` and `b` | `expr 10 \* 5` | `50` |
| `a / b` | Returns the arithmetic quotient of `a` divided by `b` | `expr 10 / 5` | `2` |
| `a % b` | Returns the arithmetic remainder of `a` divided by `b` | `expr 10 % 3` | `1` |
| `a > b` | Returns 1 if `a` is greater than `b`; else returns 0 | `expr 10 \> 5` | `1` |
| `a >= b` | Returns 1 if `a` is greater than or equal to `b`; else returns 0 | `expr 5 \>= 5` | `1` |
| `a < b` | Returns 1 if `a` is less than `b`; else returns 0 | `expr 3 \< 5` | `1` |
| `a <= b` | Returns 1 if `a` is less than or equal to `b`; else returns 0 | `expr 5 \<= 5` | `1` |
| `a = b` | Returns 1 if `a` equals `b`; else returns 0 | `expr 5 = 5` | `1` |

> ⚠️ **Important:** In `bash`, symbols like `*`, `<`, `>` have special meaning (wildcard/redirection), so they **must be escaped with a backslash** (`\*`, `\<`, `\>`) when used with `expr`.

**Full worked example, using every arithmetic/relational operator in one script:**
```bash
a=10
b=3

echo "Sum        : $(expr $a + $b)"
echo "Difference : $(expr $a - $b)"
echo "Product    : $(expr $a \* $b)"
echo "Quotient   : $(expr $a / $b)"
echo "Remainder  : $(expr $a % $b)"
echo "a > b ?    : $(expr $a \> $b)"
echo "a >= b ?   : $(expr $a \>= $b)"
echo "a < b ?    : $(expr $a \< $b)"
echo "a <= b ?   : $(expr $a \<= $b)"
echo "a = b ?    : $(expr $a = $b)"
```
**Output:**
```
Sum        : 13
Difference : 7
Product    : 30
Quotient   : 3
Remainder  : 1
a > b ?    : 1
a >= b ?   : 1
a < b ?    : 0
a <= b ?   : 0
a = b ?    : 0
```

### 4.2 Logical & String Operators — with Examples of Each

| Operator | Description | Example | Result |
|---|---|---|---|
| `a \| b` | Returns `a` if neither argument is null or 0; else returns `b` | `expr 0 \| 5` | `5` |
| `a & b` | Returns `a` if neither argument is null or 0; else returns 0 | `expr 4 & 5` | `4` |
| `a != b` | Returns 1 if `a` is not equal to `b`; else returns 0 | `expr 4 != 5` | `1` |
| `str : reg` | Returns the position up to the anchored pattern match with a Basic Regular Expression (BRE) `str` | `expr "HelloWorld" : "Hello"` | `5` |
| `match str reg` | Returns the pattern match if `reg` matches the pattern in `str` | `expr match "abc123" '[a-z]*'` | `3` |
| `substr str n m` | Returns the substring `m` characters in length, starting at position `n` | `expr substr "HelloWorld" 1 5` | `Hello` |
| `index str chars` | Returns the position in `str` where any one of `chars` is first found; else returns 0 | `expr index "HelloWorld" "oW"` | `5` |
| `length str` | Returns the numeric length of the string `str` | `expr length "HelloWorld"` | `10` |
| `+ token` | Interprets `token` as a string even if it is a keyword | `expr + match` | `match` |
| `(exprn)` | Returns the value of the expression `exprn` (used for grouping/precedence) | `expr \( 2 + 3 \) \* 4` | `20` |

**Full worked example, using every logical/string operator in one script:**
```bash
str="HelloWorld"

echo "OR (a|b)         : $(expr 0 \| 9)"
echo "AND (a&b)         : $(expr 7 & 3)"
echo "Not equal (!=)    : $(expr 5 != 5)"
echo "Anchored match(:) : $(expr "$str" : "Hello")"
echo "match function    : $(expr match "$str" 'Hello')"
echo "Substring         : $(expr substr "$str" 1 5)"
echo "Index of chars    : $(expr index "$str" "oW")"
echo "Length            : $(expr length "$str")"
echo "Grouping ()       : $(expr \( 2 + 3 \) \* 4)"
```
**Output:**
```
OR (a|b)         : 9
AND (a&b)         : 7
Not equal (!=)    : 0
Anchored match(:) : 5
match function    : 5
Substring         : Hello
Index of chars    : 5
Length            : 10
Grouping ()       : 20
```

> 📝 **Note:** `str : reg` and `match str reg` use **Basic Regular Expressions (BRE)**, not extended regex — special characters like `+`, `?`, `|` inside a pattern need escaping (`\+`, `\?`, `\|`) to be treated literally as regex metacharacters. When the pattern itself is stored in a variable, the same operator works with variables on both sides, e.g. `expr $str : $regex`, provided `$regex` expands to a valid BRE pattern.

### 4.3 Pattern Matching with `str =~ regex` Inside `[[ ]]`

#### Concept
Bash's `[[ ]]` extended test supports the `=~` operator to match a string against an **Extended Regular Expression (ERE)** — the same regex flavor used by `egrep`/`grep -E`. This is different from `expr`'s `str : reg`, which only checks an **anchored match at the start** of the string using BRE syntax. With `=~`, the match can occur **anywhere** in the string unless you explicitly anchor it with `^` and `$`, and ERE metacharacters like `+`, `?`, `|` work directly without backslash-escaping.

#### Syntax
```bash
[[ $string =~ $regex ]]     # returns true (exit 0) if regex matches anywhere in string
                             # $regex should be unquoted (or in a variable) to be treated as a pattern, not a literal
```

#### Example 1 — Validating That a Line Contains Only Digits (`^[0-9]+$`)
```bash
check_digits() {
    local line="$1"
    if [[ $line =~ ^[0-9]+$ ]]
    then
        echo "\"$line\" -> valid (digits only)"
    else
        echo "\"$line\" -> invalid"
    fi
}

check_digits "12345"
check_digits "123a45"
check_digits ""
```
**Output:**
```
"12345" -> valid (digits only)
"123a45" -> invalid
"" -> invalid
```
*(`^` anchors the match to the start of the string, `[0-9]+` requires one-or-more digits, and `$` anchors to the end — together they force the **entire** string to be numeric.)*

#### Example 2 — Matching Word Variants: `[oO]ctav[aeiou]*`
```bash
words=("Octave" "octave" "Octav" "Octavio" "banana")

for w in "${words[@]}"
do
    if [[ $w =~ [oO]ctav[aeiou]* ]]
    then
        echo "$w -> MATCH"
    else
        echo "$w -> no match"
    fi
done
```
**Output:**
```
Octave -> MATCH
octave -> MATCH
Octav -> MATCH
Octavio -> MATCH
banana -> no match
```
*(`[oO]` matches an uppercase or lowercase `O`, `ctav` matches literally, and `[aeiou]*` matches **zero or more** trailing vowels — so it matches `Octav` with no trailing vowel at all, as well as `Octave` and `Octavio` with extra letters after the vowel run, since `=~` doesn't require the whole string to match unless anchored.)*

#### Example 3 — Capturing Matched Groups with `BASH_REMATCH`
```bash
line="Name: John, Age: 25"
if [[ $line =~ Age:\ ([0-9]+) ]]
then
    echo "Full match : ${BASH_REMATCH[0]}"
    echo "Captured age: ${BASH_REMATCH[1]}"
fi
```
**Output:**
```
Full match : Age: 25
Captured age: 25
```
*(Parentheses `( )` in an ERE define a capture group; after a successful `=~` match, Bash automatically populates the `BASH_REMATCH` array — index `0` holds the whole match, and `1`, `2`, ... hold each capture group.)*

#### Example 4 — `=~` vs `expr str : reg` vs `expr match`

| Feature | `expr str : reg` / `expr match` | `[[ str =~ regex ]]` |
|---|---|---|
| Regex flavor | Basic Regular Expression (BRE) | Extended Regular Expression (ERE) |
| Match position | Anchored at the **start** of `str` | Anywhere in the string (unless `^`/`$` used) |
| Metacharacter escaping | `+`, `?`, `|` need `\` to act as regex operators | `+`, `?`, `|` work directly, no escaping |
| Captured groups | Only returns match length/substring | Full groups available via `BASH_REMATCH` |
| Needs external process | Yes (`expr` is a separate binary) | No — built into the shell (`[[ ]]` is a keyword) |

> ⚠️ **Exam Tip:** A very common exam trap is forgetting to anchor with `^...$` — `[[ "abc123xyz" =~ [0-9]+ ]]` still returns **true** because `123` appears somewhere in the string, even though the string as a whole is not purely numeric.

---

## 5. The Heredoc Feature

### Concept
A **heredoc** (`<<`) lets you feed multiple lines of input directly into a command (commonly `bc`, `cat`, `mail`, `sort`, etc.) without needing a separate input file. It is extremely useful for feeding multi-line scripts/data to interactive command-line calculators or tools like `bc`.

### Syntax
```bash
command << MARKER
line 1
line 2
MARKER
```
- The **marker/delimiter** can be **any word** — it does not have to be `EOF`.
- Using `<<-` (with a hyphen) tells Bash to **ignore leading TAB characters** (not spaces) in the heredoc body — useful when the heredoc is indented inside a function or loop for readability.

### 5.1 `bc` — The Bench Calculator

#### Concept
`bc` (short for **"bench calculator"**) is an arbitrary-precision command-line calculator. Bash's built-in arithmetic (`$(( ))`, `let`, `expr`) only handles **integers** — for floating-point/decimal math, `bc` is the standard tool, which is why it's so often paired with heredocs (as shown throughout this section) to feed it multi-line expressions.

#### Syntax
```bash
echo "expression" | bc          # pipe a single expression to bc
bc -l <<< "expression"          # here-string, one-liner
bc -l << EOF                    # heredoc, multi-line
...
EOF
```
- `-l` loads the math library, enabling decimal precision and functions like `sqrt()`, `sin()`, `c()` (cosine).
- `scale = N` (set inside the expression) controls the number of decimal places shown in the result.

#### Example 0 — Quick One-Liner Uses of `bc`
```bash
echo "10 / 3" | bc -l
echo "sqrt(49)" | bc -l
bc -l <<< "scale=2; 22/7"
```
**Output:**
```
3.33333333333333333333
7.00000000000000000000
3.14
```

### Example 1 — Basic Heredoc with `EOF` (Compound Interest Calculation)
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

### Example 2 — Heredoc with a Custom Marker Name and Leading-Tab Ignore
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

### Example 3 — Heredoc Used with `cat` to Print a Multi-Line Message
```bash
cat << MSG
========================
   Welcome to my script
   Author: Student
========================
MSG
```
**Output:**
```
========================
   Welcome to my script
   Author: Student
========================
```

### Example 4 — Heredoc for Sending Multiple Commands to `bc` at Once
```bash
result=$(bc << CALC
5 + 3
10 * 2
100 / 4
CALC
)
echo "$result"
```
**Output:**
```
8
20
25
```
*(Each line inside the heredoc is treated as a separate `bc` command, and each result is printed on its own line.)*

> 💡 **Exam Tip:** The marker name (`EOF`, `ABC`, `MSG`, `CALC`, or any custom word) must appear **alone on its own line** to close the heredoc, and it must match exactly (case-sensitive) with the opening marker. If the closing marker has leading whitespace and you did NOT use `<<-`, the heredoc will not close correctly.

### 5.2 Changing the Field Separator — `IFS`

#### Concept
`IFS` (**Internal Field Separator**) is a special Bash variable that tells the shell which character(s) to treat as **word/field boundaries** when splitting a string — for example, in `for` loop word-splitting, `read`, or command substitution. By **default**, `IFS` is a space, a tab, and a newline. Temporarily changing `IFS` lets you correctly split lines that use a different delimiter, such as `:` in `/etc/passwd` or `$PATH`, or `,` in a CSV line.

#### Syntax
```bash
IFS=:                     # change IFS for the rest of the script/session
old_ifs=$IFS               # save the original value first, if you plan to restore it
IFS=$old_ifs                # restore default IFS afterward

IFS=: command              # change IFS only for the duration of ONE command (safer, no leakage)
```

#### Example 1 — Splitting `$PATH` on `:` with a `for` Loop
```bash
IFS=:
for dir in $PATH
do
    echo "Directory: $dir"
done
IFS=$' \t\n'      # restore default IFS (space, tab, newline)
```
**Sample Output (truncated):**
```
Directory: /usr/local/bin
Directory: /usr/bin
Directory: /bin
```

#### Example 2 — Reading a Colon-Delimited `/etc/passwd`-Style Line
```bash
line="john:x:1001:1001:John Doe:/home/john:/bin/bash"
IFS=: read -r username _ uid gid fullname home shell <<< "$line"
echo "User  : $username"
echo "UID   : $uid"
echo "Home  : $home"
echo "Shell : $shell"
```
**Output:**
```
User  : john
UID   : 1001
Home  : /home/john
Shell : /bin/bash
```

#### Example 3 — Scoping `IFS` to a Single Command (No Leakage)
```bash
csv_line="apple,banana,cherry"

IFS=, read -ra fruits <<< "$csv_line"
echo "First fruit : ${fruits[0]}"
echo "All fruits  : ${fruits[@]}"

for word in "one two" three
do
    echo "Word: $word"
done
```
**Output:**
```
First fruit : apple
All fruits  : apple banana cherry
Word: one two
Word: three
```
*(Prefixing a single command with `IFS=,` — with no semicolon — changes `IFS` **only for that command's environment**; the shell's own `IFS` is unaffected afterward, which is why the later `for` loop still splits on the **default** space/tab/newline. This is the safest way to use a custom delimiter without remembering to restore it manually.)*

> ⚠️ **Exam Tip:** If you set `IFS=:` directly (without scoping it to one command) and **forget to restore it**, every subsequent word-splitting operation in the script — including plain `for word in $sentence` loops over normal text — will silently break, since spaces will no longer be treated as separators.

---

## 6. Conditional Statements

### 6.1 `if-elif-else-fi`

#### Concept
Used for multi-way branching based on conditions, evaluated **top to bottom**. As soon as one condition evaluates to true, its command block executes and **all remaining branches are skipped** — even if a later condition would also be true.

#### Syntax
```bash
# Simple if-else
if condition1
then
    commandset1
else
    commandset2
fi

# if-elif-else chain (multi-way)
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

#### Example 1 — Simple `if-else` (Even/Odd Check)
```bash
n=7
if [ $(( n % 2 )) -eq 0 ]
then
    echo "$n is even"
else
    echo "$n is odd"
fi
```
**Output:**
```
7 is odd
```

#### Example 2 — Grading System with `elif` Chain
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

#### Example 3 — Checking File Existence and Type
```bash
file="/etc/passwd"
if [ -f "$file" ]
then
    echo "$file is a regular file"
elif [ -d "$file" ]
then
    echo "$file is a directory"
else
    echo "$file does not exist"
fi
```
**Output:**
```
/etc/passwd is a regular file
```

#### Example 4 — Nested `if` Inside `if`
```bash
num=15
if [ $num -gt 0 ]
then
    if [ $(( num % 2 )) -eq 0 ]
    then
        echo "Positive and Even"
    else
        echo "Positive and Odd"
    fi
else
    echo "Not Positive"
fi
```
**Output:**
```
Positive and Odd
```

---

### 6.2 `case` Statement

#### Concept
The `case` statement is Bash's equivalent of `switch` in other languages. It compares a variable against several **patterns** (which can include wildcards) and executes the matching block. Multiple values can be grouped for a single block using `|` (pipe = OR). `*` acts as the **default/catch-all** case, matched only if nothing else matches.

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

#### Example 1 — Day-of-Week Menu
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

#### Example 2 — Using Wildcard Patterns (File Extension Check)
```bash
filename="report.pdf"
case $filename in
    *.txt)
        echo "Text file";;
    *.pdf)
        echo "PDF document";;
    *.jpg | *.png)
        echo "Image file";;
    *)
        echo "Unknown file type";;
esac
```
**Output:**
```
PDF document
```

#### Example 3 — Simple Command-Line Menu Using `case`
```bash
#!/bin/bash
echo "1. Start Service"
echo "2. Stop Service"
echo "3. Restart Service"
read -p "Choose an option: " choice

case $choice in
    1) echo "Starting service...";;
    2) echo "Stopping service...";;
    3) echo "Restarting service...";;
    *) echo "Invalid option";;
esac
```
**Sample run (input 3):**
```
Restarting service...
```

#### Example 4 — Case-Insensitive Matching (Yes/No Prompt)
```bash
read -p "Continue? (y/n): " ans
case $ans in
    y | Y | yes | Yes | YES)
        echo "Continuing...";;
    n | N | no | No | NO)
        echo "Stopping...";;
    *)
        echo "Please answer y or n";;
esac
```
**Sample run (input Y):**
```
Continuing...
```

---

## 7. Loops in Bash

### 7.1 C-Style `for` Loop — One Variable

#### Concept
Bash supports a C-language-style `for` loop with an **initializer**, a **condition**, and an **increment/decrement expression**, all inside double parentheses `(( ))` — very similar to `for` loops in C, Java, or JavaScript.

#### Syntax
```bash
for (( initializer; condition; increment ))
do
    commands
done
```

#### Example 1 — Counting Up from 1 to 9
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

#### Example 2 — Counting Down
```bash
for (( i = 10; i >= 1; i-- ))
do
    echo $i
done
```
**Output:**
```
10
9
8
7
6
5
4
3
2
1
```

#### Example 3 — Stepping by More Than 1 (Even Numbers)
```bash
for (( i = 0; i <= 10; i += 2 ))
do
    echo $i
done
```
**Output:**
```
0
2
4
6
8
10
```

#### Example 4 — Sum of First N Natural Numbers Using a `for` Loop
```bash
n=5
sum=0
for (( i = 1; i <= n; i++ ))
do
    (( sum += i ))
done
echo "Sum of first $n numbers is $sum"
```
**Output:**
```
Sum of first 5 numbers is 15
```

---

### 7.2 C-Style `for` Loop — Two Variables

#### Concept
The C-style `for` loop can track **two (or more) variables simultaneously**, each with its own initializer and increment/decrement expression, separated by commas. However, only **one condition** is allowed to control loop termination — this is a very common exam trick question.

#### Syntax
```bash
for (( var1=init1, var2=init2; condition; var1++, var2-- ))
do
    commands
done
```

> ⚠️ **Exam Tip:** Even with multiple variables, the loop uses **only one condition** to decide when to stop.

#### Example 1 — Two Counters Moving in Opposite Directions
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

#### Example 2 — Two Counters Moving in the Same Direction
```bash
for (( i = 0, j = 100; i < 5; i++, j += 10 ))
do
    echo "i=$i  j=$j"
done
```
**Output:**
```
i=0  j=100
i=1  j=110
i=2  j=120
i=3  j=130
i=4  j=140
```

#### Example 3 — Three Variables in One `for` Loop
```bash
for (( a=1, b=2, c=3; a <= 3; a++, b+=1, c+=1 ))
do
    echo "a=$a b=$b c=$c"
done
```
**Output:**
```
a=1 b=2 c=3
a=2 b=3 c=4
a=3 b=4 c=5
```

---

### 7.3 Redirecting Loop Output

#### Concept
The **entire output** of a loop can be redirected to a file by placing the redirection operator (`>` or `>>`) right after the `done` keyword — this redirects the **combined output of every iteration**, not just the last one, because the redirection applies to the whole loop as a single unit.

#### Syntax
```bash
for (( initializer; condition; increment ))
do
    commands
done > filename          # overwrite file
done >> filename          # append to file
```

#### Example 1 — Basic Loop Output Redirection
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
**Output (contents of `tmp.<pid>`):**
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

#### Example 2 — Appending Instead of Overwriting
```bash
for (( i = 1; i <= 3; i++ ))
do
    echo "First batch: $i"
done > numbers.txt

for (( i = 4; i <= 6; i++ ))
do
    echo "Second batch: $i"
done >> numbers.txt

cat numbers.txt
```
**Output:**
```
First batch: 1
First batch: 2
First batch: 3
Second batch: 4
Second batch: 5
Second batch: 6
```

#### Example 3 — Redirecting While Also Piping Through Another Command
```bash
for (( i = 5; i >= 1; i-- ))
do
    echo $i
done | sort -n > sorted_output.txt

cat sorted_output.txt
```
**Output:**
```
1
2
3
4
5
```

> 📝 **Explanation of `tmp.$$`:** `$$` holds the current shell/script's Process ID (PID), so `tmp.$$` creates a **unique temporary filename** every time the script runs — a common professional pattern to avoid filename collisions.

---

## 8. Loop Control Statements

### 8.1 `break`

#### Concept
`break` immediately exits the **innermost** enclosing loop, skipping any remaining iterations — execution continues with the first statement **after** the loop's `done`.

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

#### Example 1 — Break on Reaching a Value
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

#### Example 2 — Break Inside a `for` Loop (Search Example)
```bash
numbers=(4 8 15 16 23 42)
target=16
for n in "${numbers[@]}"
do
    if [ $n -eq $target ]
    then
        echo "Found $target!"
        break
    fi
    echo "Checked $n, not a match"
done
```
**Output:**
```
Checked 4, not a match
Checked 8, not a match
Checked 15, not a match
Found 16!
```

#### Example 3 — Break Inside an Infinite Loop (common pattern)
```bash
count=1
while true
do
    echo "Count: $count"
    (( count++ ))
    if [ $count -gt 4 ]
    then
        break
    fi
done
```
**Output:**
```
Count: 1
Count: 2
Count: 3
Count: 4
```

---

### 8.2 `break n` (Breaking Nested Loops)

#### Concept
By default, `break` only exits the **innermost** loop. To break out of **multiple nested loops at once**, specify a number `n` with `break`, indicating **how many levels** of enclosing loops to exit (counted from the innermost outward).

#### Syntax
```bash
break n     # breaks out of n levels of enclosing loops
```

#### Example 1 — Breaking Out of the Inner Loop Only (Plain `break`)
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
*(This continues all the way to `i=9` since only the inner loop breaks each time; the outer loop is unaffected.)*

#### Example 2 — Breaking Out of Both Loops at Once (`break 2`)
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
> 📝 As soon as the inner counter `j` reaches 7, `break 2` exits **both** the inner and outer `while` loops immediately — this is why the output stops after the 7th "row" even though `n=10`.

#### Example 3 — Triple-Nested Loop with `break 3`
```bash
for (( a = 1; a <= 3; a++ ))
do
    for (( b = 1; b <= 3; b++ ))
    do
        for (( c = 1; c <= 3; c++ ))
        do
            echo "a=$a b=$b c=$c"
            if [ $a -eq 2 ] && [ $b -eq 2 ] && [ $c -eq 2 ]
            then
                echo "Breaking out of all 3 loops!"
                break 3
            fi
        done
    done
done
```
**Output:**
```
a=1 b=1 c=1
a=1 b=1 c=2
a=1 b=1 c=3
a=1 b=2 c=1
a=1 b=2 c=2
a=1 b=2 c=3
a=1 b=3 c=1
a=1 b=3 c=2
a=1 b=3 c=3
a=2 b=1 c=1
a=2 b=1 c=2
a=2 b=1 c=3
a=2 b=2 c=1
a=2 b=2 c=2
Breaking out of all 3 loops!
```

---

### 8.3 `continue`

#### Concept
`continue` skips the **remaining commands** in the current loop iteration and jumps straight to the next iteration's condition check — it does **not** exit the loop, unlike `break`.

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

#### Example 1 — Skipping Even Numbers
```bash
for (( i = 1; i <= 10; i++ ))
do
    if [ $(( i % 2 )) -eq 0 ]
    then
        continue
    fi
    echo $i
done
```
**Output:**
```
1
3
5
7
9
```

#### Example 2 — Skipping a Specific Value in a List
```bash
for fruit in apple banana grape mango banana kiwi
do
    if [ "$fruit" == "banana" ]
    then
        continue
    fi
    echo "Fruit: $fruit"
done
```
**Output:**
```
Fruit: apple
Fruit: grape
Fruit: mango
Fruit: kiwi
```

#### Example 3 — Complex Original Example (Skipping a Range 4–5)
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
> 📝 Values `4` and `5` are skipped in every inner loop's printed sequence because the `continue` statement fires whenever `j` is greater than 3 **and** less than 6, jumping past the `printf` line for those two values only.

---

### 8.4 `continue n` (Nested Continue)

#### Concept
Just like `break n`, `continue` also accepts a numeric argument `n`, meaning "skip to the next iteration of the loop that is `n` levels up" — this resumes the **outer** loop's next iteration, instead of just the inner one.

#### Syntax
```bash
continue n     # skips to next iteration of the n-th enclosing loop
```

#### Example — `continue 2` Skipping to the Outer Loop's Next Iteration
```bash
for (( i = 1; i <= 3; i++ ))
do
    for (( j = 1; j <= 3; j++ ))
    do
        if [ $j -eq 2 ]
        then
            continue 2
        fi
        echo "i=$i j=$j"
    done
done
```
**Output:**
```
i=1 j=1
i=2 j=1
i=3 j=1
```
*(As soon as `j` reaches 2, `continue 2` skips the rest of the inner loop **and** moves the outer loop `i` to its next value — so `j=3` is never printed for any `i`.)*

---

## 9. Positional Parameters — `shift`

### Concept
`shift` shifts all **command-line arguments** (positional parameters `$1, $2, $3, ...`) one position to the **left**. After a `shift`, what was `$2` becomes `$1`, `$3` becomes `$2`, and so on. The original `$1` before the shift is permanently discarded, and `$#` (the argument count) decreases by the shift amount.

### Syntax
```bash
shift          # shifts by 1 (default)
shift n        # shifts by n positions
```

### Example 1 — Processing All Arguments One by One
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

### Example 2 — Using `shift 2` to Skip Two Arguments at Once
```bash
#!/bin/bash
# Run as: ./script.sh one two three four five
echo "Before shift: $1 $2 $3 $4 $5"
shift 2
echo "After shift 2: $1 $2 $3"
```
**Output:**
```
Before shift: one two three four five
After shift 2: three four five
```

### Example 3 — Using `shift` to Separate a "Command" from Its "Options"
```bash
#!/bin/bash
# Run as: ./script.sh deploy --env production --verbose
action=$1
shift
echo "Action: $action"
echo "Remaining options: $@"
```
**Output:**
```
Action: deploy
Remaining options: --env production --verbose
```

### Example 4 — Counting Down Arguments with `$#`
```bash
#!/bin/bash
# Run as: ./script.sh a b c
while [ $# -gt 0 ]
do
    echo "Remaining args: $# -> current: $1"
    shift
done
```
**Output:**
```
Remaining args: 3 -> current: a
Remaining args: 2 -> current: b
Remaining args: 1 -> current: c
```

> 💡 **Exam Tip:** `[ -n "$1" ]` checks whether `$1` is a **non-empty string**; `[ $# -gt 0 ]` checks the **argument count** directly — both are valid ways to control a `shift`-based loop, but `$#` is generally considered more robust.

---

## 10. The `exec` Command

### Concept
`exec` **replaces** the current shell process with a new program, instead of spawning a separate child process. Key behavior points:
- If the new program launches **successfully**, control **never returns** to the original shell/script — because the shell process itself has been replaced by the new program.
- If the new program **fails** to launch (e.g., file not found, no permission), the original shell **continues** executing normally, right after the `exec` line.
- `exec` is also used to **change I/O redirection** for the remainder of a script, without replacing the process (a special case, see Example 3).

### Syntax
```bash
exec ./my-executable --my-options --my-args
```

### Example 1 — Successful `exec` (Process Replacement)
```bash
#!/bin/bash
echo "About to replace this shell..."
exec ls -l /home
echo "This line will NEVER execute if exec succeeds"
```
**Explanation:** Once `exec ls -l /home` runs successfully, the shell process is entirely replaced by `ls`, so the final `echo` statement never runs — the script effectively "becomes" the `ls` command.

### Example 2 — Failed `exec` (Shell Continues)
```bash
#!/bin/bash
echo "Trying to exec a non-existent program..."
exec ./this-program-does-not-exist
echo "This line WILL execute, because exec failed"
```
**Output:**
```
Trying to exec a non-existent program...
bash: ./this-program-does-not-exist: No such file or directory
This line WILL execute, because exec failed
```

### Example 3 — Using `exec` to Redirect I/O for Remainder of Script (No Process Replacement)
```bash
#!/bin/bash
exec > output.log      # redirect all subsequent stdout to output.log
echo "This goes into output.log, not the terminal"
echo "So does this line"
```
**Explanation:** When `exec` is given only a redirection (no command), it changes the shell's own I/O settings permanently for the rest of the script's execution — this is a very common professional logging pattern.

---

## 11. The `eval` Command

### Concept
`eval` takes its arguments, **combines them into a single string**, and then **executes that string as a shell command**. Unlike `exec`, `eval` **returns control** back to the shell once the command finishes, along with a normal exit status.

### Syntax
```bash
eval my-arg
```

### Example 1 — Basic `eval`
```bash
cmd="echo"
arg="Hello, World!"
eval $cmd $arg
```
**Output:**
```
Hello, World!
```

### Example 2 — Dynamic Variable Assignment (Indirect Reference)
```bash
varname="greeting"
eval $varname="HelloThere"
eval echo \$$varname
```
**Output:**
```
HelloThere
```

### Example 3 — Building and Running a Command String Dynamically
```bash
command_string="ls -l /tmp | wc -l"
eval $command_string
```
**Explanation:** `eval` interprets the pipe (`|`) inside the string as an actual shell pipe operator, executing `ls -l /tmp` and piping its output into `wc -l`, exactly as if you had typed the full command directly.

### Example 4 — Using `eval` to Access an Array Element by a Dynamic Index
```bash
arr=(10 20 30 40)
idx=2
eval echo \${arr[$idx]}
```
**Output:**
```
30
```

> 📝 **Exam Tip:** `eval` is powerful but should be used carefully — since it executes arbitrary strings as commands, feeding it untrusted/user input can be a security risk (command injection).

---

## 12. Parsing Options — `getopts`

### Concept
`getopts` is used to parse **command-line options/flags** passed to a script, similar to how standard UNIX commands accept flags like `-a`, `-b value`. It differentiates between:
- Options that take **no argument** (e.g., `a`)
- Options that **require an argument** (indicated by a following colon `:` in the option string, e.g., `b:`)

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
- `${OPTIND}` holds the index of the next argument to be processed (useful after the options loop finishes, to access remaining non-option arguments).

### Example 1 — Basic Three-Flag Script (Original Example)
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

### Example 2 — Handling an Invalid Option
```bash
./script.sh -z
```
**Output:**
```
Usage: -a -b barg -c carg
```

### Example 3 — Practical Script: Backup Utility with Options
```bash
#!/bin/bash
verbose=0
while getopts "vs:d:" opt
do
    case "$opt" in
        v)
            verbose=1
            ;;
        s)
            source_dir=${OPTARG}
            ;;
        d)
            dest_dir=${OPTARG}
            ;;
        *)
            echo "Usage: -v -s source_dir -d dest_dir"
            exit 1
            ;;
    esac
done

if [ $verbose -eq 1 ]
then
    echo "Verbose mode ON"
fi
echo "Copying from $source_dir to $dest_dir"
```
**Sample run:**
```bash
./backup.sh -v -s /data -d /backup
```
**Output:**
```
Verbose mode ON
Copying from /data to /backup
```

### Example 4 — Using `$OPTIND` to Access Remaining Non-Option Arguments
```bash
#!/bin/bash
while getopts "a:" opt
do
    case "$opt" in
        a) aval=${OPTARG};;
    esac
done
shift $(( OPTIND - 1 ))
echo "Option a = $aval"
echo "Remaining arguments: $@"
```
**Sample run:**
```bash
./script.sh -a hello file1.txt file2.txt
```
**Output:**
```
Option a = hello
Remaining arguments: file1.txt file2.txt
```

> ⚠️ **Exam Tip:** The script in Example 1 can be invoked **only** with the three defined options — `a`, `b`, and `c`. Any other flag falls into the `*` (default/usage) case, which is why validating the option string carefully is important.

---

## 13. The `select` Loop (Text Menus)

### Concept
The `select` loop is a built-in Bash construct specifically designed to build **interactive numbered text menus**. It automatically displays a numbered list of choices, prompts the user (using the `PS3` prompt variable) for input, and **repeats indefinitely** until explicitly stopped (typically with `break`).

### Syntax
```bash
select variable in list_of_options
do
    commands   # normally includes a case statement + break
done
```

### Example 1 — Original "Pick a Number" Menu
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
**Sample run (user selects 5):**
```
Select a middle one
1) 1   3) 3   5) 5   7) 7   9) 9
2) 2   4) 4   6) 6   8) 8   10) 10
#? 5
you picked the right one
selection completed with 5
```

### Example 2 — Simple Menu with a Custom Prompt (`PS3`)
```bash
PS3="Please choose an option: "
select opt in "Start" "Stop" "Restart" "Quit"
do
    case $opt in
        "Start")
            echo "Starting service..."
            ;;
        "Stop")
            echo "Stopping service..."
            ;;
        "Restart")
            echo "Restarting service..."
            ;;
        "Quit")
            echo "Goodbye!"
            break
            ;;
        *)
            echo "Invalid option, try again"
            ;;
    esac
done
```
**Sample run:**
```
1) Start
2) Stop
3) Restart
4) Quit
Please choose an option: 2
Stopping service...
Please choose an option: 4
Goodbye!
```

### Example 3 — Building a File-Selection Menu
```bash
echo "Select a file to view:"
select file in *.txt "Cancel"
do
    if [ "$file" == "Cancel" ]
    then
        echo "Cancelled."
        break
    elif [ -n "$file" ]
    then
        echo "You chose: $file"
        break
    else
        echo "Invalid selection, try again."
    fi
done
```
**Explanation:** `select file in *.txt "Cancel"` automatically expands the wildcard to list every `.txt` file in the current directory as a numbered option, plus a manual "Cancel" choice.

### Example 4 — Infinite Menu Without `break` (Demonstrating the Pitfall)
```bash
select choice in "Option A" "Option B"
do
    echo "You picked: $choice"
    # NOTE: no break here!
done
```
**Explanation:** Because there is **no `break`** anywhere in the loop body, this menu will **keep reprinting itself forever**, asking the user to choose again and again — this is a classic exam "spot the bug" question.

> 💡 **Text Menu Tip:** `select` is one of the most exam-relevant constructs for building simple CLI menus — remember that it **loops indefinitely** unless a `break` (or an `exit`) is included inside the loop body.

---

## 14. Common Mistakes & Exam Pitfalls

This section consolidates the most frequently tested "gotchas" across all topics above — review this list right before your exam.

| # | Mistake | Why It's Wrong | Correct Approach |
|---|---|---|---|
| 1 | `expr $a+20` | No spaces — `expr` treats it as one literal string, not an expression | `expr $a + 20` (always use spaces) |
| 2 | `expr $a * $b` | `*` is a shell wildcard character if unescaped | `expr $a \* $b` |
| 3 | Multiple conditions in a two-variable `for` loop, e.g. `a < finish; b > 0` | C-style `for` in Bash allows only **one** condition even with multiple counters | Use one condition only: `a < finish` |
| 4 | Forgetting `break` inside a `select` loop | Menu will repeat **forever** | Always include a `break` (or `exit`) path |
| 5 | Using `break` when you meant `break 2` in nested loops | Only exits the innermost loop, outer loop keeps running | Use `break n` to specify how many loop levels to exit |
| 6 | Using `continue` when you meant `continue 2` | Only skips to the next iteration of the innermost loop | Use `continue n` to skip an outer loop's iteration |
| 7 | Expecting code after `exec` to run | If `exec` succeeds, the shell process is **replaced** — nothing after it executes | Know that only a **failed** `exec` allows the next line to run |
| 8 | Forgetting to quote a heredoc's closing marker's indentation | If body lines are indented with **spaces** and you didn't use `<<-`, tabs vs spaces mismatch may break heredoc closing | Use `<<-` and TAB-indent when a heredoc appears inside an indented block |
| 9 | Missing colon after option letter in `getopts` string | Without `:`, `getopts` assumes the option takes **no argument**, so `OPTARG` stays empty | Use `"b:"` (with colon) if the option needs a value |
| 10 | Confusing `$[ ]` with `$(( ))` | `$[ ]` is deprecated; some exam answers may still test recognition of it | Use `$(( ))` in your own scripts, but recognize `$[ ]` if it appears |
| 11 | Using `eval` on untrusted/user input directly | Can lead to unintended command execution (security risk) | Validate/sanitize input before passing to `eval` |

---

## 15. Overall Summary

This set of notes (Week 7, Lectures 1–3, Bash Scripting Part 2/3/4) covers the **advanced building blocks** needed to write professional, production-quality Bash scripts. Key takeaways, in the recommended learning order:

1. **Debugging (`set -x` / `bash -x`)** — Always debug before submitting/using a script; trace execution line by line with real variable values shown.
2. **Combining Conditions (`&&`, `||`)** — Build compound logical tests without deeply nested `if` statements; works on any command's exit status, not just `[ ]` tests.
3. **Shell Arithmetic** — Four methods exist (`let`, `expr`, `$(( ))`, `$[ ]`); **`$(( ))` is the modern standard**, while `expr` needs careful spacing and escaping of special characters like `*`.
4. **`expr` Operators** — A rich operator set exists beyond simple math: relational (`>`, `<`, `=`), logical (`|`, `&`, `!=`), and string operators (`length`, `substr`, `index`, `match`, anchored `:`).
5. **Heredoc (`<<`)** — Feed multi-line input to commands like `bc`; use `<<-` to strip leading tabs; marker name is fully flexible (not limited to `EOF`).
6. **Conditionals** — `if-elif-else-fi` for range/threshold-based logic; `case` for clean multi-way pattern matching, using `|` to group values and `*` as the default.
7. **Loops** — The C-style `for (( ))` loop supports single or multiple counter variables (but only **one** stopping condition); loop output can be redirected as a whole using `done > file`.
8. **Loop Control** — `break`/`break n` exit loops (single or multiple nested levels); `continue`/`continue n` skip only the current iteration (of the current or an outer loop), not the whole loop.
9. **`shift` / `shift n`** — Essential for processing an unknown/variable number of command-line arguments one (or more) at a time; combine with `$#` for robust argument-count checks.
10. **`exec` vs `eval`** — `exec` **replaces** the running shell process (no return, unless it fails to launch); `eval` **executes a built string as a command** and always returns control back to the shell.
11. **`getopts`** — The standard, professional way to parse `-flag value` style command-line options in scripts; use `OPTARG` for values and `OPTIND`/`shift` to access remaining arguments.
12. **`select`** — Built specifically for building interactive, numbered text menus in the terminal; **always** requires a `break` to avoid an infinite menu loop.

> ✅ **Final Exam Tip:** Practice writing each construct (`if`, `case`, `for`, `while`, `getopts`, `select`) from memory, and be ready to **trace through nested loop output by hand** (especially `break n`/`continue n` and heredoc examples), since these are the most common types of "predict the output" questions. Review Section 14 (Common Mistakes) one final time right before the exam — most marks are lost on small syntax slips, not conceptual misunderstanding.

---

*End of Detailed Notes — Bash Scripting Part 2, 3 & 4 (Week 7)*
