# Shell Programming — Bash Scripting (Complete Notes)
### Week 7 | Lectures 1, 2 & 3 | Advanced Bash Script Features
### Professional Exam Preparation Notes — Final Edition

---

## Table of Contents

1. [Debugging Bash Scripts](#1-debugging-bash-scripts)
2. [Combining Conditions](#2-combining-conditions)
   - 2.1 [AND / OR Outside `[ ]`](#21-and--or-outside--)
   - 2.2 [AND / OR Inside `[[ ]]`](#22-and--or-inside--)
3. [Shell Arithmetic](#3-shell-arithmetic)
   - 3.1 [Arithmetic Expansion `(( ))`](#31-arithmetic-expansion--)
   - 3.2 [Using `let`](#32-using-let)
   - 3.3 [Using `expr`](#33-using-expr)
   - 3.4 [Using `$(( expression ))`](#34-using--expression-)
   - 3.5 [Using `$[ expression ]` (Legacy)](#35-using--expression--legacy)
   - 3.6 [Using `bc` — Basic/Bench Calculator](#36-using-bc--basicbench-calculator)
   - 3.7 [Operators Inside `(( ))` and `let`](#37-operators-inside--and-let)
   - 3.8 [Comparison of All Arithmetic Methods](#38-comparison-of-all-arithmetic-methods)
4. [`expr` Command Operators (Reference Tables)](#4-expr-command-operators-reference-tables)
   - 4.1 [Arithmetic & Relational Operators](#41-arithmetic--relational-operators)
   - 4.2 [Logical Operators](#42-logical-operators)
5. [Pattern Matching & Regular Expressions](#5-pattern-matching--regular-expressions)
   - 5.1 [`str : regex` — Anchored Pattern Match](#51-str--regex--anchored-pattern-match)
   - 5.2 [`str =~ regex` Inside `[[ ]]`](#52-str--regex-inside--)
   - 5.3 [`match str regex`](#53-match-str-regex)
   - 5.4 [`substr str start length`](#54-substr-str-start-length)
   - 5.5 [`index str chars`](#55-index-str-chars)
   - 5.6 [`length str`](#56-length-str)
   - 5.7 [Practical Regex Patterns](#57-practical-regex-patterns)
6. [The Heredoc Feature](#6-the-heredoc-feature)
7. [Changing the Field Separator — `IFS`](#7-changing-the-field-separator--ifs)
8. [Conditional Statements](#8-conditional-statements)
   - 8.1 [`if` / `if-else` / `if-elif-else-fi`](#81-if--if-else--if-elif-else-fi)
   - 8.2 [`case` Statement](#82-case-statement)
9. [Loops in Bash](#9-loops-in-bash)
   - 9.1 [C-Style `for` Loop — One Variable](#91-c-style-for-loop--one-variable)
   - 9.2 [C-Style `for` Loop — Two Variables](#92-c-style-for-loop--two-variables)
   - 9.3 [Redirecting Loop Output](#93-redirecting-loop-output)
10. [Measuring Execution Time — `time`](#10-measuring-execution-time--time)
11. [Loop Control Statements](#11-loop-control-statements)
    - 11.1 [`break`](#111-break)
    - 11.2 [`break n` (Nested Loops)](#112-break-n-nested-loops)
    - 11.3 [`continue`](#113-continue)
    - 11.4 [`continue n` (Nested Loops)](#114-continue-n-nested-loops)
12. [Positional Parameters — `shift`](#12-positional-parameters--shift)
13. [The `exec` Command](#13-the-exec-command)
14. [The `eval` Command](#14-the-eval-command)
15. [Parsing Options — `getopts`](#15-parsing-options--getopts)
16. [The `select` Loop (Text Menus)](#16-the-select-loop-text-menus)
17. [Common Mistakes & Exam Pitfalls](#17-common-mistakes--exam-pitfalls)
18. [Overall Summary](#18-overall-summary)

---

## 1. Debugging Bash Scripts

### Definition
**Debugging** in shell scripting is the process of tracing a script's execution, command by command, so that logic errors, wrong variable expansions, and unexpected control flow can be identified and corrected. Bash provides a built-in tracing mechanism, activated by the `-x` option, which prints every command **after variable substitution and before execution**, prefixed with a `+` symbol (the default `PS4` prompt).

### Syntax
```bash
set -x            # turn tracing ON (place inside the script)
set +x            # turn tracing OFF
bash -x ./script.sh   # trace the entire script from the command line, no editing needed
```

| Method | Scope | Typical Use |
|---|---|---|
| `bash -x ./myscript.sh` | Entire script, from a fresh shell | Quick one-off debugging without modifying the file |
| `set -x` (inside script) | From that line onward | Debug only a specific, suspicious section |
| `set +x` (inside script) | Cancels a prior `set -x` | Stop tracing once the problem area has passed |

### Example 1 — Whole-Script Debugging via Command Line
```bash
#!/bin/bash
a=5
b=10
sum=$(( a + b ))
echo "Sum is $sum"
```
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

### Example 2 — Debugging Only a Section with `set -x` / `set +x`
```bash
#!/bin/bash
echo "Start of script (not traced)"
a=100

set -x
b=200
total=$(( a + b ))
set +x

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

### Example 3 — Debugging a Loop
```bash
#!/bin/bash
set -x
for (( i = 1; i <= 3; i++ ))
do
    echo "Iteration: $i"
done
set +x
```
**Output (excerpt):**
```
+ (( i = 1 ))
+ (( i <= 3 ))
+ echo 'Iteration: 1'
Iteration: 1
+ (( i++ ))
...
```

> 💡 **Exam Tip:** The traced output shows the **actual values** substituted into each command — this is different from simply reading the script source, which only shows `$variablename`.

---

## 2. Combining Conditions

### Definition
Bash allows two or more conditional tests to be logically combined using **AND** (`&&`) and **OR** (`||`), so that compound decisions can be expressed without deeply nested `if` statements. These operators can be used **between separate `[ ]` test commands**, or **inside a single `[[ ]]` extended test command**, which is a more modern and safer construct.

### 2.1 AND / OR Outside `[ ]`

#### Syntax
```bash
[ condition1 ] && [ condition2 ]     # AND — TRUE only if BOTH are true
[ condition1 ] || [ condition2 ]     # OR  — TRUE if AT LEAST ONE is true
```
Here, `[ ]` is the classic POSIX `test` command. Each `[ ]` is evaluated as an **independent command**, and `&&` / `||` chain their **exit statuses** together (0 = success/true).

#### Example 1 — AND
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

#### Example 2 — OR
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

#### Example 3 — Command Chaining with `&&` (Not Just Tests)
```bash
mkdir project_folder && cd project_folder && echo "Folder created and entered"
```
*(If `mkdir` fails, the chain stops — `&&` only continues if the previous command's exit status was 0/success.)*

### 2.2 AND / OR Inside `[[ ]]`

#### Definition
`[[ ]]` is Bash's **extended test command**. Unlike `[ ]`, it is a shell **keyword**, not an external/builtin command with word-splitting rules — this means `&&` and `||` can be written **directly inside a single `[[ ]]`**, and it also supports pattern matching and regex (see Section 5.2), without needing to escape special characters like `<` or `>`.

#### Syntax
```bash
[[ condition1 && condition2 ]]     # AND, written inside one [[ ]]
[[ condition1 || condition2 ]]     # OR, written inside one [[ ]]
```

#### Example 1 — AND Inside a Single `[[ ]]`
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
grade="B"
if [[ $grade == "A" || $grade == "B" ]]
then
    echo "Good performance"
else
    echo "Needs improvement"
fi
```
**Output:**
```
Good performance
```

#### Example 3 — `[[ ]]` Allows Unescaped `<` / `>` for String Comparison
```bash
str1="apple"
str2="banana"
if [[ $str1 < $str2 ]]
then
    echo "$str1 comes before $str2 alphabetically"
fi
```
**Output:**
```
apple comes before banana alphabetically
```
*(Inside `[ ]`, `<` would need to be escaped as `\<` and would need `expr`-style handling; `[[ ]]` handles it natively for string comparison.)*

> 📝 **Exam Tip:** `[ ]` and `[[ ]]` are **not interchangeable** in every context — `[[ ]]` is safer (no need to quote variables against word-splitting, supports `&&`/`||`/regex natively), while `[ ]` is more portable across POSIX-compliant shells (e.g., `sh`, `dash`).

---

## 3. Shell Arithmetic

### Definition
**Shell arithmetic** refers to the evaluation of numeric (integer, and in some cases floating-point) expressions inside a Bash script. Because the shell's core language only manipulates strings by default, Bash provides several dedicated constructs — `(( ))`, `let`, `expr`, `$(( ))`, `$[ ]`, and the external `bc` utility — each suited to slightly different situations.

### 3.1 Arithmetic Expansion `(( ))`

#### Definition
`(( ))` is used to **evaluate** an arithmetic expression as a **command** (not to produce a value directly like `$(( ))`). Its exit status is 0 (true) if the resulting value is non-zero, and 1 (false) if the result is zero. It is most commonly used for **assignment**, **increment/decrement**, and as the **condition inside `if`/`while`**.

#### Syntax
```bash
(( expression ))
```

#### Example 1 — Assignment
```bash
(( total = 10 + 20 ))
echo $total
```
**Output:**
```
30
```

#### Example 2 — Using `(( ))` as a Condition
```bash
a=5
if (( a > 3 ))
then
    echo "a is greater than 3"
fi
```
**Output:**
```
a is greater than 3
```

#### Example 3 — Increment Directly
```bash
count=10
(( count++ ))
echo "count is $count"
```
**Output:**
```
count is 11
```

### 3.2 Using `let`

#### Definition
`let` evaluates one or more arithmetic expressions and assigns the result to a variable. Spaces around operators are optional; if the expression contains spaces, the whole expression must be quoted.

#### Syntax
```bash
let a=$1+5
let "a = $1 + 5"
```

#### Example 1 — Basic Addition
```bash
#!/bin/bash
let a=$1+5
echo "Value of a: $a"
```
**Output (input 10):**
```
Value of a: 15
```

#### Example 2 — Multiple Operations
```bash
x=4
let "y = x * x + 2"
echo "y is $y"
```
**Output:**
```
y is 18
```

#### Example 3 — Increment with `let`
```bash
count=0
let count++
let count++
echo "count is $count"
```
**Output:**
```
count is 2
```

### 3.3 Using `expr`

#### Definition
`expr` is an **external command** that evaluates its arguments as an expression and prints the result to standard output. It strictly requires **whitespace** between every operator and operand, since it parses each argument as a separate token.

#### Syntax
```bash
expr $a + 20
expr "$a + 20"
b=$( expr $a + 20 )
```

#### Example 1 — Direct Output
```bash
a=5
expr $a + 20
```
**Output:**
```
25
```

#### Example 2 — Capturing Result
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
expr $a+20
```
**Output:**
```
5+20
```
*(Treated as one literal string, since there is no space to separate the operator.)*

### 3.4 Using `$(( expression ))`

#### Definition
`$(( ))` is **arithmetic expansion** — it evaluates the enclosed expression and **substitutes the numeric result** directly into the command line, similar to how `$( )` substitutes command output. This is the **modern, most recommended** method for arithmetic in Bash, supporting the full C-style operator set.

#### Syntax
```bash
b=$(( $a + 10 ))
b=$(( a + 10 ))       # $ before the variable name is optional inside $(( ))
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

#### Example 2 — Direct Use Inside `echo`
```bash
echo "5 squared is $(( 5 * 5 ))"
```
**Output:**
```
5 squared is 25
```

#### Example 3 — Compound Assignment
```bash
total=100
(( total += 50 ))
echo "$total"
```
**Output:**
```
150
```

### 3.5 Using `$[ expression ]` (Legacy)

#### Definition
`$[ ]` is a **deprecated** arithmetic construct that behaves identically to `$(( ))`. It remains functional for backward compatibility with older scripts but should not be used in new code.

#### Syntax
```bash
b=$[ $a + 10 ]
```

#### Example
```bash
a=5
b=$[ $a + 10 ]
echo "b is $b"
```
**Output:**
```
b is 15
```

### 3.6 Using `bc` — Basic/Bench Calculator

#### Definition
`bc` (**"Basic Calculator"**, sometimes called the **bench calculator**) is a standalone, arbitrary-precision command-line calculator program. Unlike `(( ))`, `let`, or `expr`, which are limited to **integer arithmetic**, `bc` natively supports **floating-point/decimal arithmetic**, controlled by its `scale` variable (which sets the number of decimal digits shown in results). It is typically invoked with the `-l` flag to load the standard math library (enabling functions like `sqrt()`, `sin()`, etc.), and is most commonly fed input via a **heredoc** (see Section 6) or a pipe.

#### Syntax
```bash
echo "expression" | bc
echo "expression" | bc -l
bc -l << EOF
scale = 5
expression
EOF
```

#### Example 1 — Simple Floating-Point Division
```bash
echo "10 / 3" | bc -l
```
**Output:**
```
3.33333333333333333333
```

#### Example 2 — Controlling Decimal Precision with `scale`
```bash
echo "scale=2; 10 / 3" | bc
```
**Output:**
```
3.33
```

#### Example 3 — Using `bc` with Shell Variables via a Pipe
```bash
a=15
b=4
result=$(echo "$a / $b" | bc -l)
echo "Result: $result"
```
**Output:**
```
Result: 3.75000000000000000000
```

#### Example 4 — `bc` with a Heredoc for a Multi-Step Calculation
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

> 📝 **Exam Tip:** `bc` is the **only** one of the arithmetic tools covered here that natively handles **decimal/floating-point** numbers — `(( ))`, `let`, `expr`, and `$(( ))` are all restricted to **integers only**.

### 3.7 Operators Inside `(( ))` and `let`

Both `(( ))` and `let` support the **full C-style operator set**, summarized below.

| Category | Operators | Example | Meaning |
|---|---|---|---|
| Arithmetic | `+  -  *  /  %` | `(( r = 10 % 3 ))` | Remainder → `r = 1` |
| Increment/Decrement | `++  --` | `(( a++ ))` | Post-increment `a` |
| Compound Assignment | `+= -= *= /= %=` | `(( a += 5 ))` | Add 5 to `a` |
| Relational | `<  <=  >  >=  ==  !=` | `(( a == b ))` | True if `a` equals `b` |
| Logical | `&&  \|\|  !` | `(( a > 0 && b > 0 ))` | True if both positive |
| Bitwise | `&  \|  ^  ~  <<  >>` | `(( a << 2 ))` | Left-shift `a` by 2 bits |
| Ternary | `cond ? val1 : val2` | `(( max = (a>b) ? a : b ))` | Pick the larger value |
| Exponent | `**` | `(( sq = 5 ** 2 ))` | `sq = 25` |

#### Example 1 — Relational Operator Inside `(( ))`
```bash
a=10
b=20
(( a == b )) && echo "Equal" || echo "Not Equal"
```
**Output:**
```
Not Equal
```

#### Example 2 — Ternary Operator
```bash
a=7
b=12
(( max = (a > b) ? a : b ))
echo "Max is $max"
```
**Output:**
```
Max is 12
```

#### Example 3 — Exponentiation
```bash
echo $(( 2 ** 10 ))
```
**Output:**
```
1024
```

#### Example 4 — `let` with a Relational Operator
```bash
a=5
b=5
let "result = (a == b)"
echo $result
```
**Output:**
```
1
```

### 3.8 Comparison of All Arithmetic Methods

| Method | Example | Integer Only? | Spacing Rule | Recommended? |
|---|---|---|---|---|
| `(( ))` | `(( b = a + 10 ))` | Yes | Flexible | ✅ Yes (for conditions/assignment as a command) |
| `let` | `let a=$1+5` | Yes | Optional (quote if spaced) | Yes, still common |
| `expr` | `expr $a + 20` | Yes | **Mandatory spaces** | Legacy, still tested |
| `$(( ))` | `b=$(( a + 10 ))` | Yes | Flexible | ✅ **Best practice overall** |
| `$[ ]` | `b=$[ a + 10 ]` | Yes | Flexible | ❌ Deprecated |
| `bc` | `echo "$a/$b" \| bc -l` | **No** (decimals supported) | N/A | ✅ Yes, for floating-point math |

---

## 4. `expr` Command Operators (Reference Tables)

### Definition
Beyond simple math, `expr` supports a broad set of **relational**, **logical**, and **string-processing** operators, making it a compact, all-purpose expression evaluator commonly used in older/POSIX-portable shell scripts.

### 4.1 Arithmetic & Relational Operators

| Operator | Description | Example | Result |
|---|---|---|---|
| `a + b` | Sum of `a` and `b` | `expr 10 + 5` | `15` |
| `a - b` | Difference of `a` and `b` | `expr 10 - 5` | `5` |
| `a * b` | Product of `a` and `b` | `expr 10 \* 5` | `50` |
| `a / b` | Quotient of `a` divided by `b` | `expr 10 / 5` | `2` |
| `a % b` | Remainder of `a` divided by `b` | `expr 10 % 3` | `1` |
| `a > b` | 1 if `a` greater than `b`; else 0 | `expr 10 \> 5` | `1` |
| `a >= b` | 1 if `a` ≥ `b`; else 0 | `expr 5 \>= 5` | `1` |
| `a < b` | 1 if `a` less than `b`; else 0 | `expr 3 \< 5` | `1` |
| `a <= b` | 1 if `a` ≤ `b`; else 0 | `expr 5 \<= 5` | `1` |
| `a = b` | 1 if `a` equals `b`; else 0 | `expr 5 = 5` | `1` |

> ⚠️ **Note:** `*`, `<`, `>` must be **escaped** (`\*`, `\<`, `\>`) because Bash would otherwise interpret them as a wildcard or redirection operator.

#### Full Worked Example
```bash
a=10
b=3
echo "Sum        : $(expr $a + $b)"
echo "Difference : $(expr $a - $b)"
echo "Product    : $(expr $a \* $b)"
echo "Quotient   : $(expr $a / $b)"
echo "Remainder  : $(expr $a % $b)"
echo "a > b ?    : $(expr $a \> $b)"
echo "a < b ?    : $(expr $a \< $b)"
```
**Output:**
```
Sum        : 13
Difference : 7
Product    : 30
Quotient   : 3
Remainder  : 1
a > b ?    : 1
a < b ?    : 0
```

### 4.2 Logical Operators

| Operator | Description | Example | Result |
|---|---|---|---|
| `a \| b` | Returns `a` if neither argument is null/0; else returns `b` | `expr 0 \| 9` | `9` |
| `a & b` | Returns `a` if neither argument is null/0; else returns 0 | `expr 7 & 3` | `7` |
| `a != b` | 1 if `a` not equal to `b`; else 0 | `expr 5 != 5` | `0` |
| `+ token` | Interprets `token` as a plain string, even if it's a keyword | `expr + match` | `match` |
| `(exprn)` | Groups a sub-expression to control precedence | `expr \( 2 + 3 \) \* 4` | `20` |

*(String-processing operators `str : reg`, `match`, `substr`, `index`, `length` are now covered in full detail, with dedicated regex context, in **Section 5 — Pattern Matching & Regular Expressions**, since the lecture treats them as a distinct topic area.)*

#### Full Worked Example
```bash
echo "OR (a|b)      : $(expr 0 \| 9)"
echo "AND (a&b)      : $(expr 7 & 3)"
echo "Not equal (!=) : $(expr 5 != 5)"
echo "Grouping ()    : $(expr \( 2 + 3 \) \* 4)"
```
**Output:**
```
OR (a|b)      : 9
AND (a&b)      : 7
Not equal (!=) : 0
Grouping ()    : 20
```

---

## 5. Pattern Matching & Regular Expressions

### Definition
Bash provides multiple mechanisms to test whether a string matches a given pattern, ranging from simple **substring extraction** to full **Basic Regular Expression (BRE)** matching via `expr`, and modern **Extended Regular Expression (ERE)** matching via the `=~` operator inside `[[ ]]`. This is one of the most heavily tested practical skill areas, since it is used constantly for input validation (e.g., checking that a string is a valid number, email, or filename pattern).

### 5.1 `str : regex` — Anchored Pattern Match

#### Definition
The `expr` operator `str : reg` compares `str` against the Basic Regular Expression `reg`. The match is **automatically anchored at the beginning** of `str` (as if the pattern were prefixed with `^`). It returns the **length of the matched portion**, or `0` if no match is found. If the pattern contains a parenthesized sub-expression `\( \)`, `expr` returns the **matched substring** instead of a length.

#### Syntax
```bash
expr "$str" : "regex"
```

#### Example 1 — Length of Matching Prefix
```bash
expr "HelloWorld" : "Hello"
```
**Output:**
```
5
```

#### Example 2 — No Match at the Beginning Returns 0
```bash
expr "HelloWorld" : "World"
```
**Output:**
```
0
```
*(Fails because the match must start at position 1; "World" only appears later in the string.)*

#### Example 3 — Extracting a Substring Using Parentheses
```bash
expr "HelloWorld" : '\(Hello\)'
```
**Output:**
```
Hello
```

### 5.2 `str =~ regex` Inside `[[ ]]`

#### Definition
The `=~` operator, used **only inside `[[ ]]`**, performs **Extended Regular Expression (ERE)** matching — the same regex flavor used by `egrep`/`grep -E`. Unlike `str : reg`, the match is **not anchored** by default (it can match anywhere in the string, unless `^`/`$` anchors are explicitly included in the pattern). If the match succeeds, capture groups are automatically stored in the `BASH_REMATCH` array.

#### Syntax
```bash
[[ $str =~ regex ]]
```

#### Example 1 — Basic Match Anywhere in the String
```bash
str="HelloWorld"
if [[ $str =~ World ]]
then
    echo "Match found"
fi
```
**Output:**
```
Match found
```

#### Example 2 — Validating Digits-Only Input (see also Section 5.7)
```bash
input="12345"
if [[ $input =~ ^[0-9]+$ ]]
then
    echo "Valid number"
else
    echo "Not a valid number"
fi
```
**Output:**
```
Valid number
```

#### Example 3 — Using `BASH_REMATCH` to Capture a Group
```bash
str="Order-4521"
if [[ $str =~ Order-([0-9]+) ]]
then
    echo "Order ID: ${BASH_REMATCH[1]}"
fi
```
**Output:**
```
Order ID: 4521
```

> 📝 **Exam Tip:** `str : reg` (via `expr`) uses **BRE** and is **anchored at the start**; `str =~ reg` (via `[[ ]]`) uses **ERE** and is **unanchored** unless you add `^`/`$` yourself. These are two genuinely different matching mechanisms and are commonly confused on exams.

### 5.3 `match str regex`

#### Definition
`match str reg` is functionally **identical** to `str : reg` — both perform an anchored BRE match and return the length of the match (or the captured group if parentheses are used). `match` is simply an alternate, more readable keyword form of the same `expr` operator.

#### Syntax
```bash
expr match "$str" "regex"
```

#### Example
```bash
expr match "abc123" '[a-z]*'
```
**Output:**
```
3
```
*(Matches the leading lowercase-letter run "abc"; stops at the first digit.)*

### 5.4 `substr str start length`

#### Definition
`substr str n m` extracts a **substring** of `m` characters from `str`, starting at 1-indexed position `n`.

#### Syntax
```bash
expr substr "$str" n m
```

#### Example 1 — Basic Extraction
```bash
expr substr "HelloWorld" 1 5
```
**Output:**
```
Hello
```

#### Example 2 — Extracting From the Middle
```bash
expr substr "HelloWorld" 6 5
```
**Output:**
```
World
```

#### Example 3 — Requesting More Characters Than Available
```bash
expr substr "Hi" 1 10
```
**Output:**
```
Hi
```
*(`expr` simply returns whatever characters are available, without erroring.)*

### 5.5 `index str chars`

#### Definition
`index str chars` searches `str` for **any one** of the characters listed in `chars`, and returns the **position of the first matching character found** (1-indexed). If none of the characters are found, it returns `0`.

#### Syntax
```bash
expr index "$str" "chars"
```

#### Example 1 — Basic Usage
```bash
expr index "HelloWorld" "oW"
```
**Output:**
```
5
```
*(The letter `o` is found first, at position 5, before `W` at position 6.)*

#### Example 2 — No Match Found
```bash
expr index "HelloWorld" "xyz"
```
**Output:**
```
0
```

### 5.6 `length str`

#### Definition
`length str` returns the total number of characters in `str`.

#### Syntax
```bash
expr length "$str"
```

#### Example
```bash
expr length "HelloWorld"
```
**Output:**
```
10
```

### 5.7 Practical Regex Patterns

The following are commonly tested, real-world regex patterns using both `expr` (BRE) and `[[ =~ ]]` (ERE) styles.

#### Example 1 — Matching Only Digits in a Line: `^[0-9]+$`

**Definition:** This pattern anchors at the start (`^`) and end (`$`) of the string, requiring **one or more** (`+`) digit characters (`[0-9]`) and **nothing else** — it validates that the entire line consists purely of numbers.

```bash
line="98765"
if [[ $line =~ ^[0-9]+$ ]]
then
    echo "Line contains only digits"
else
    echo "Line contains non-digit characters"
fi
```
**Output:**
```
Line contains only digits
```

**Testing a failing case:**
```bash
line="987a65"
if [[ $line =~ ^[0-9]+$ ]]
then
    echo "Line contains only digits"
else
    echo "Line contains non-digit characters"
fi
```
**Output:**
```
Line contains non-digit characters
```

#### Example 2 — Matching Name Variants: `[oO]ctav[aeiou]*`

**Definition:** This pattern matches either an uppercase or lowercase leading `O`/`o` (`[oO]`), followed by the literal text `ctav`, followed by **zero or more** vowels (`[aeiou]*`) — this single pattern matches words like `Octave`, `octave`, `Octav`, `Octavio`, `Octavia`, etc.

```bash
for word in "Octave" "octave" "Octav" "Octavio" "Octavia" "banana"
do
    if [[ $word =~ ^[oO]ctav[aeiou]*$ ]]
    then
        echo "$word -> MATCH"
    else
        echo "$word -> no match"
    fi
done
```
**Output:**
```
Octave -> MATCH
octave -> MATCH
Octav -> MATCH
Octavio -> no match
Octavia -> MATCH
banana -> no match
```
*(Note: "Octavio" fails because `[aeiou]*` matches only vowels, not the consonant `v` followed by `io` after "Octav" — the trailing part must consist purely of vowel characters for a full match. This demonstrates why careful boundary/anchor testing matters in regex.)*

#### Example 3 — Matching Using `expr $str : $regex` (BRE Style)

**Definition:** As introduced in Section 5.1, `expr` can also be used directly with a variable holding the regex pattern, which is useful for dynamically constructed patterns.

```bash
str="Octave"
regex="[oO]ctav[aeiou]*"
result=$(expr "$str" : "$regex")
echo "Matched length: $result"
```
**Output:**
```
Matched length: 6
```
*(The entire 6-character string "Octave" matches the pattern, so `expr` returns the full matched length.)*

#### Example 4 — Combining Regex Validation Inside a Script (Practical Use Case)
```bash
read -p "Enter a numeric ID: " id
if [[ $id =~ ^[0-9]+$ ]]
then
    echo "Valid ID: $id"
else
    echo "Error: ID must contain digits only"
fi
```
**Sample run (input `abc12`):**
```
Error: ID must contain digits only
```

---

## 6. The Heredoc Feature

### Definition
A **heredoc** (`<<`) is a form of I/O redirection that allows a block of text spanning multiple lines to be fed directly into a command's standard input, without creating a separate file. It is especially useful for feeding multi-line input to interactive utilities like `bc`, `cat`, `mail`, or `sort`.

### Syntax
```bash
command << MARKER
line 1
line 2
MARKER
```
- The **marker/delimiter** can be any word (not limited to `EOF`).
- `<<-` (with a hyphen) tells Bash to **strip leading TAB characters** from each line of the heredoc body (not spaces) — useful for keeping indentation readable inside functions/loops.

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

### Example 2 — Heredoc with Custom Marker and `<<-` (Ignoring Leading Tabs)
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

### Example 3 — Multi-Line Message with `cat`
```bash
cat << MSG
========================
   Welcome to my script
========================
MSG
```
**Output:**
```
========================
   Welcome to my script
========================
```

### Example 4 — Sending Multiple Commands to `bc` at Once
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

> 💡 **Exam Tip:** The closing marker must appear **alone on its own line**, matching the opening marker **exactly** (case-sensitive). If body lines are indented with spaces (not tabs) and `<<-` is used, the spaces will **not** be stripped — only tabs are removed.

---

## 7. Changing the Field Separator — `IFS`

### Definition
`IFS` (**Internal Field Separator**) is a special shell variable that defines the character(s) Bash uses to **split a string into words/fields** — for example, when splitting the output of a command inside a `for` loop, or parsing a delimited line of data (like a CSV or `/etc/passwd`-style colon-separated record). By default, `IFS` is set to a space, tab, and newline. Temporarily changing `IFS` allows scripts to correctly parse data that uses a different delimiter, such as a colon (`:`) or comma (`,`).

### Syntax
```bash
IFS=:                  # change delimiter to colon for current shell/script
IFS=$'\n'               # change delimiter to newline only
OLDIFS=$IFS; IFS=:; ...; IFS=$OLDIFS    # safely restore IFS afterward
```

### Example 1 — Splitting a Colon-Separated String (e.g., `/etc/passwd` style)
```bash
line="john:x:1001:1001:John Doe:/home/john:/bin/bash"
IFS=:
read -ra fields <<< "$line"
echo "Username : ${fields[0]}"
echo "UID      : ${fields[2]}"
echo "Home Dir : ${fields[5]}"
```
**Output:**
```
Username : john
UID      : 1001
Home Dir : /home/john
```

### Example 2 — Looping Over Colon-Separated Values
```bash
IFS=:
data="apple:banana:cherry"
for item in $data
do
    echo "Fruit: $item"
done
```
**Output:**
```
Fruit: apple
Fruit: banana
Fruit: cherry
```

### Example 3 — Safely Saving and Restoring the Original `IFS`
```bash
OLDIFS=$IFS
IFS=,
csv_line="Name,Age,City"
for field in $csv_line
do
    echo "Field: $field"
done
IFS=$OLDIFS      # restore default IFS so the rest of the script behaves normally
```
**Output:**
```
Field: Name
Field: Age
Field: City
```

> ⚠️ **Exam Tip:** Forgetting to restore `IFS` back to its default value after changing it can silently break the rest of the script (e.g., normal space-separated word splitting will stop working) — always restore it once the parsing task is done.

---

## 8. Conditional Statements

### 8.1 `if` / `if-else` / `if-elif-else-fi`

#### Definition
The `if` construct evaluates a condition's exit status; if it is `0` (true), the associated command block runs. `if-else` adds an alternate block for when the condition is false, and `if-elif-else` allows chaining multiple conditions for multi-way branching, evaluated top to bottom, stopping at the **first** true condition.

#### Syntax
```bash
# Simple if
if condition
then
    commandset
fi

# if-else
if condition
then
    commandset1
else
    commandset2
fi

# if-elif-else
if condition1
then
    commandset1
elif condition2
then
    commandset2
else
    commandset3
fi
```

#### Example 1 — Simple `if` (No Else)
```bash
n=10
if [ $n -gt 5 ]
then
    echo "$n is greater than 5"
fi
```
**Output:**
```
10 is greater than 5
```

#### Example 2 — `if-else` (Even/Odd Check)
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

#### Example 3 — `if-elif-else` Grading Chain
```bash
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

#### Example 4 — Nested `if`
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

### 8.2 `case` Statement

#### Definition
`case` is Bash's pattern-matching multi-way branch statement, equivalent to `switch` in C-family languages. It compares a variable against a series of **patterns** (which may include wildcards), executing the first matching block. `|` groups multiple values into one branch; `*` serves as the catch-all default.

#### Syntax
```bash
case $var in
    op1)
        commandset1;;
    op2 | op3)
        commandset2;;
    *)
        commandset3;;
esac
```

#### Example 1 — Day-of-Week Menu
```bash
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

#### Example 2 — Wildcard Pattern Matching (File Extension)
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

#### Example 3 — Case-Insensitive Yes/No Prompt
```bash
read -p "Continue? (y/n): " ans
case $ans in
    y | Y | yes | Yes)
        echo "Continuing...";;
    n | N | no | No)
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

## 9. Loops in Bash

### 9.1 C-Style `for` Loop — One Variable

#### Definition
Bash supports a C-language-style `for (( ))` loop, consisting of an **initializer**, a **condition**, and an **increment/decrement expression**, all inside double parentheses.

#### Syntax
```bash
for (( initializer; condition; increment ))
do
    commands
done
```

#### Example 1 — Counting Up
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
**Output (excerpt):**
```
10
9
8
...
1
```

#### Example 3 — Stepping by 2 (Even Numbers)
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

### 9.2 C-Style `for` Loop — Two Variables

#### Definition
The C-style `for` loop can initialize and update **two or more variables simultaneously**, separated by commas, but is controlled by only **one** terminating condition.

#### Syntax
```bash
for (( var1=init1, var2=init2; condition; var1++, var2-- ))
do
    commands
done
```

#### Example 1 — Two Counters Moving Oppositely
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

#### Example 2 — Two Counters Moving Together
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

### 9.3 Redirecting Loop Output

#### Definition
Placing a redirection operator (`>` or `>>`) immediately after a loop's `done` redirects the **combined output of all iterations** to a file, treating the entire loop as a single unit for I/O purposes.

#### Syntax
```bash
for (( ... ))
do
    commands
done > filename     # overwrite
done >> filename     # append
```

#### Example 1 — Basic Redirection
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

#### Example 2 — Appending
```bash
for (( i = 1; i <= 3; i++ ))
do
    echo "Batch1: $i"
done > numbers.txt

for (( i = 4; i <= 6; i++ ))
do
    echo "Batch2: $i"
done >> numbers.txt

cat numbers.txt
```
**Output:**
```
Batch1: 1
Batch1: 2
Batch1: 3
Batch2: 4
Batch2: 5
Batch2: 6
```

> 📝 **Explanation of `tmp.$$`:** `$$` is the current process's PID, so `tmp.$$` generates a unique filename per run, avoiding collisions.

---

## 10. Measuring Execution Time — `time`

### Definition
The `time` keyword/command is used to measure how long a given command or script takes to execute. It reports three values: **real** time (actual wall-clock time elapsed), **user** time (CPU time spent in user-mode code), and **sys** time (CPU time spent in kernel/system calls on behalf of the process).

### Syntax
```bash
time <command>
```

### Example 1 — Timing a Simple Command
```bash
time sleep 2
```
**Output:**
```
real    0m2.003s
user    0m0.001s
sys     0m0.002s
```

### Example 2 — Timing a Loop Inside a Script
```bash
time (
for (( i = 0; i < 100000; i++ ))
do
    :   # no-op command
done
)
```
**Output (approximate, will vary by machine):**
```
real    0m0.045s
user    0m0.043s
sys     0m0.001s
```

### Example 3 — Comparing the Performance of Two Approaches
```bash
echo "Using expr:"
time ( for (( i=0; i<1000; i++ )); do expr $i + 1 > /dev/null; done )

echo "Using \$(( )):"
time ( for (( i=0; i<1000; i++ )); do echo $(( i + 1 )) > /dev/null; done )
```
*(This demonstrates practically why `$(( ))` is preferred over `expr` for performance — `expr` spawns an external process on every call, making it significantly slower in loops.)*

> 💡 **Exam Tip:** `real` time can be **less than** `user + sys` on multi-core systems (parallel work), or **greater than** `user + sys` when the process is waiting on I/O — understanding this distinction is a common short-answer exam question.

---

## 11. Loop Control Statements

### 11.1 `break`

#### Definition
`break` immediately terminates the **innermost** enclosing loop, transferring control to the first statement after the loop's `done`.

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

#### Example 2 — Break Inside a `for` Loop (Search Pattern)
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

### 11.2 `break n` (Nested Loops)

#### Definition
`break n` exits `n` levels of enclosing loops at once, counted from the innermost loop outward — needed when a single `break` (which only exits the innermost loop) is not sufficient.

#### Syntax
```bash
break n
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

### 11.3 `continue`

#### Definition
`continue` skips the remaining commands in the **current** iteration and jumps directly to the next iteration's condition check, without exiting the loop.

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

#### Example 2 — Original Complex Example (Skipping Range 4–5)
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

### 11.4 `continue n` (Nested Loops)

#### Definition
`continue n` skips to the next iteration of the loop that is `n` levels up from the innermost loop, rather than just the current innermost loop.

#### Syntax
```bash
continue n
```

#### Example
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

---

## 12. Positional Parameters — `shift`

### Definition
`shift` shifts all positional parameters (`$1, $2, $3, ...`) one position to the **left**, discarding the original `$1`. An optional numeric argument `n` shifts by `n` positions at once. `$#` (the argument count) decreases correspondingly.

### Syntax
```bash
shift          # shift by 1
shift n        # shift by n positions
```

### Example 1 — Processing All Arguments
```bash
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

### Example 2 — `shift 2` to Skip Two at Once
```bash
echo "Before: $1 $2 $3 $4 $5"
shift 2
echo "After shift 2: $1 $2 $3"
```
**Output (for `one two three four five`):**
```
Before: one two three four five
After shift 2: three four five
```

### Example 3 — Separating a Command From Its Options
```bash
action=$1
shift
echo "Action: $action"
echo "Remaining options: $@"
```
**Output (for `deploy --env production --verbose`):**
```
Action: deploy
Remaining options: --env production --verbose
```

---

## 13. The `exec` Command

### Definition
`exec` **replaces the current shell process** with a new program, rather than spawning a child process. If the new program launches successfully, control **never returns** to the original script. If it fails to launch, the shell continues with the next line. `exec` (used without a command) can also permanently change the shell's own I/O redirection for the rest of the script.

### Syntax
```bash
exec ./my-executable --my-options --my-args
```

### Example 1 — Successful `exec`
```bash
echo "About to replace this shell..."
exec ls -l /home
echo "This line will NEVER execute if exec succeeds"
```

### Example 2 — Failed `exec`
```bash
echo "Trying a non-existent program..."
exec ./this-program-does-not-exist
echo "This line WILL execute, because exec failed"
```
**Output:**
```
Trying a non-existent program...
bash: ./this-program-does-not-exist: No such file or directory
This line WILL execute, because exec failed
```

### Example 3 — `exec` for I/O Redirection Only
```bash
exec > output.log
echo "This goes into output.log, not the terminal"
```

---

## 14. The `eval` Command

### Definition
`eval` combines its arguments into a single string and executes that string as a shell command, then **returns control** back to the shell with a normal exit status — unlike `exec`, which replaces the process entirely.

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

### Example 2 — Dynamic Variable Assignment
```bash
varname="greeting"
eval $varname="HelloThere"
eval echo \$$varname
```
**Output:**
```
HelloThere
```

### Example 3 — Building and Running a Piped Command String
```bash
command_string="ls -l /tmp | wc -l"
eval $command_string
```

---

## 15. Parsing Options — `getopts`

### Definition
`getopts` parses command-line **options/flags** passed to a script, distinguishing between flags that take no argument (e.g., `a`) and flags that require an argument, marked with a trailing colon in the option string (e.g., `b:`). Matched argument values are stored automatically in `${OPTARG}`.

### Syntax
```bash
while getopts "ab:c:" options
do
    case "${options}" in
        a) ;;
        b) barg=${OPTARG};;
        c) carg=${OPTARG};;
        *) echo "Usage: -a -b barg -c carg";;
    esac
done
```

### Example 1 — Three-Flag Script
```bash
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

### Example 2 — Practical Backup Script
```bash
while getopts "vs:d:" opt
do
    case "$opt" in
        v) verbose=1;;
        s) source_dir=${OPTARG};;
        d) dest_dir=${OPTARG};;
        *) echo "Usage: -v -s source_dir -d dest_dir"; exit 1;;
    esac
done
echo "Copying from $source_dir to $dest_dir"
```
**Sample run:**
```bash
./backup.sh -v -s /data -d /backup
```
**Output:**
```
Copying from /data to /backup
```

### Example 3 — Using `$OPTIND` to Access Remaining Arguments
```bash
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

---

## 16. The `select` Loop (Text Menus)

### Definition
`select` is a Bash construct purpose-built for creating **interactive, numbered text menus**. It displays a numbered list, prompts the user via the `PS3` prompt variable, stores the chosen value in the loop variable, and **loops indefinitely** until explicitly exited (typically via `break`).

### Syntax
```bash
select variable in list_of_options
do
    commands
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
**Sample run (input 5):**
```
Select a middle one
1) 1   3) 3   5) 5   7) 7   9) 9
2) 2   4) 4   6) 6   8) 8   10) 10
#? 5
you picked the right one
selection completed with 5
```

### Example 2 — Custom Prompt with `PS3`
```bash
PS3="Please choose an option: "
select opt in "Start" "Stop" "Restart" "Quit"
do
    case $opt in
        "Start") echo "Starting service...";;
        "Stop") echo "Stopping service...";;
        "Restart") echo "Restarting service...";;
        "Quit") echo "Goodbye!"; break;;
        *) echo "Invalid option, try again";;
    esac
done
```

### Example 3 — File-Selection Menu Using a Wildcard
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

> 💡 **Exam Tip:** `select` **loops forever** if no `break` (or `exit`) exists anywhere in its body — this is a classic "spot the bug" exam question.

---

## 17. Common Mistakes & Exam Pitfalls

| # | Mistake | Why It's Wrong | Correct Approach |
|---|---|---|---|
| 1 | `expr $a+20` | No spaces — treated as one literal string | `expr $a + 20` |
| 2 | `expr $a * $b` | `*` is a shell wildcard if unescaped | `expr $a \* $b` |
| 3 | Multiple conditions in a two-variable `for` loop | Bash's C-style `for` allows only **one** condition, regardless of variable count | Use a single condition, e.g. `a < finish` |
| 4 | Forgetting `break` inside `select` | Menu repeats **forever** | Always include a `break`/`exit` path |
| 5 | Using `break` when `break 2` was needed | Only exits the innermost loop | Use `break n` for the correct nesting level |
| 6 | Confusing `str : reg` (BRE, anchored) with `str =~ reg` (ERE, unanchored) | Different regex flavors and matching behavior | Use `expr :`/`match` for anchored BRE; `[[ =~ ]]` for flexible ERE |
| 7 | Forgetting to restore `IFS` after changing it | Breaks default word-splitting for the rest of the script | Save original with `OLDIFS=$IFS` and restore it afterward |
| 8 | Expecting code after a successful `exec` to run | The shell process is **replaced**, so nothing after it executes | Know that only a **failed** `exec` allows the next line to run |
| 9 | Missing colon after an option letter in `getopts` | Without `:`, the option is assumed to take **no argument** | Use `"b:"` if the flag needs a value |
| 10 | Assuming `(( ))`, `let`, `expr`, `$(( ))` handle decimals | All four are **integer-only** | Use `bc -l` for floating-point arithmetic |
| 11 | Misreading `time` output when processes run in parallel | `real` time is not simply `user + sys` | Understand `real` = wall-clock; `user`/`sys` = CPU time |

---

## 18. Overall Summary

This complete set of notes (Week 7, Lectures 1–3, Bash Scripting) integrates every construct from the lecture recordings into a single, logically ordered reference, from **basic debugging** through to **advanced menu-driven scripts**:

1. **Debugging (`set -x`/`bash -x`)** — trace real, substituted command execution to catch logic errors early.
2. **Combining Conditions (`&&`, `||`, `[[ ]]`)** — build compound logic outside or inside test brackets; `[[ ]]` is the safer, more feature-rich modern choice.
3. **Shell Arithmetic** — six related tools exist (`(( ))`, `let`, `expr`, `$(( ))`, `$[ ]`, `bc`); `$(( ))`/`(( ))` are the modern standard for integers, and `bc` is required for decimals.
4. **`expr` Operators** — arithmetic, relational, and logical operators, all requiring careful spacing/escaping.
5. **Pattern Matching & Regex** — `str : reg` and `match` (anchored BRE via `expr`) versus `str =~ reg` (unanchored ERE via `[[ ]]`); `substr`, `index`, `length` for direct string inspection; practical patterns like `^[0-9]+$` for digit-only validation.
6. **Heredoc (`<<`, `<<-`)** — feed multi-line input to commands like `bc`; flexible marker names; tab-stripping variant.
7. **`IFS`** — controls how strings are split into fields/words; must be restored after temporary changes.
8. **Conditionals (`if`/`case`)** — range-based branching versus clean pattern-based branching.
9. **Loops (`for (( ))`)** — single or multi-variable C-style loops, constrained to one stopping condition; output redirection applies to the whole loop via `done > file`.
10. **`time`** — measures real/user/sys execution time, useful for performance comparisons (e.g., `expr` vs. `$(( ))`).
11. **Loop Control (`break`/`break n`/`continue`/`continue n`)** — exit or skip iterations at any nesting depth.
12. **`shift`/`shift n`** — process a variable number of command-line arguments.
13. **`exec` vs `eval`** — `exec` replaces the shell process (no return on success); `eval` executes a built string and returns control normally.
14. **`getopts`** — the professional standard for parsing `-flag value` options, paired with `OPTARG`/`OPTIND`.
15. **`select`** — purpose-built interactive numbered menus; always needs an explicit `break`.

> ✅ **Final Exam Tip:** Be ready to trace nested loop output by hand (`break n`/`continue n`), distinguish BRE (`expr`) from ERE (`[[ =~ ]]`) regex behavior, and remember which arithmetic tools are integer-only versus decimal-capable (`bc`). Review Section 17 (Common Mistakes) as your final pass before the exam — most lost marks come from small syntax slips, not conceptual gaps.

---

*End of Complete Notes — Bash Scripting, Week 7 (Lectures 1–3)*
