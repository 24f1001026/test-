# Shell Variables — Week 5 (Lecture 2 & 3)
### Exam Preparation Notes — Creation, Inspection, Modification, Lists & Arrays

> **Note:** Shell variable manipulations are **fast** — this is one of the reasons shell scripting relies so heavily on parameter expansion instead of external commands (like `sed`/`awk`) for simple string operations.

---

## 📑 Table of Contents

1. [Creating a Variable](#1-creating-a-variable)
   - [1.1 Setting Command Output to a Variable (Command Substitution)](#11-setting-command-output-to-a-variable-command-substitution)
2. [Accessing Variable Values](#2-accessing-variable-values)
3. [Exporting a Variable](#3-exporting-a-variable)
   - [3.1 Variable Availability to Shells and Subshells](#31-variable-availability-to-shells-and-subshells)
   - [3.2 Modifying an Exported Variable in a Child Shell](#32-modifying-an-exported-variable-in-a-child-shell)
4. [Removing a Variable / Its Value](#4-removing-a-variable--its-value)
   - [4.1 Removing a Variable Completely](#41-removing-a-variable-completely)
   - [4.2 Removing Only the Value](#42-removing-only-the-value)
5. [Checking Variable Status](#5-checking-variable-status)
   - [5.1 Test if a Variable is Set](#51-test-if-a-variable-is-set)
   - [5.2 Test if a Variable is Not Set](#52-test-if-a-variable-is-not-set)
6. [Default Value Handling](#6-default-value-handling)
   - [6.1 Substitute Default Value](#61-substitute-default-value-)
   - [6.2 Set Default Value](#62-set-default-value-)
   - [6.3 Reset Value if Variable is Set](#63-reset-value-if-variable-is-set-)
   - [6.4 Show Error Message if Variable is Not Set](#64-show-error-message-if-variable-is-not-set-varmessage)
7. [Listing Variable Names](#7-listing-variable-names)
8. [Length of String Value](#8-length-of-string-value)
9. [Slice of String Value](#9-slice-of-string-value)
   - [9.1 Negative Index (Slicing from the Right)](#91-negative-index-slicing-from-the-right)
10. [Pattern Removal](#10-pattern-removal)
    - [10.1 Remove Matching Pattern from the Beginning](#101-remove-matching-pattern-from-the-beginning-prefix)
    - [10.2 Remove Matching Pattern from the End](#102-remove-matching-pattern-from-the-end-suffix)
    - [10.3 Mixing Prefix and Suffix Removal](#103-mixing-prefix-and-suffix-removal)
11. [Pattern Replacement](#11-pattern-replacement)
    - [11.1 Replace Matching Pattern (Anywhere)](#111-replace-matching-pattern-anywhere)
    - [11.2 Replace Matching Pattern by Location](#112-replace-matching-pattern-by-location)
12. [Changing Case](#12-changing-case)
13. [Restricting Variable Value Types](#13-restricting-variable-value-types)
    - [13.1 Applying Restrictions](#131-applying-restrictions)
    - [13.2 Removing Restrictions](#132-removing-restrictions)
14. [Arrays](#14-arrays)
    - [14.1 Indexed Arrays](#141-indexed-arrays)
      - [14.1.1 Populating an Array in One Go](#1411-populating-an-array-in-one-go)
      - [14.1.2 Passing Command Output to an Array](#1412-passing-command-output-to-an-array)
    - [14.2 Associative Arrays](#142-associative-arrays)
15. [Summary](#15-summary)

---

## 1. Creating a Variable

### Concept
A shell variable is created simply by assigning a value to a name. Shell variables are **untyped** by default (they hold strings, but can store numbers, strings, or command output).

### Syntax
```bash
myvar="value string"
```

### Rules (Important — Exam Point)
| Rule | Explanation |
|---|---|
| No spaces around `=` | `myvar = "value"` is **invalid**; must be `myvar="value"` |
| Cannot start with a number | `1var=10` → Error. Must start with a letter or underscore |
| Alphanumeric + underscore allowed | `my_var1=10` → Valid |
| Value type | Can be a **number**, a **string**, or the output of a **command substitution** (`` `command` `` or `$(command)`) |

### Examples
```bash
myvar="Hello World"        # string
count=25                   # number
today=`date +%Y-%m-%d`     # command substitution (backticks)
today2=$(date +%Y-%m-%d)   # command substitution (modern preferred syntax)
```

### 1.1 Setting Command Output to a Variable (Command Substitution)

#### Concept
A variable can store the **output** of a command instead of a literal string. The shell runs the command first, captures its standard output, and assigns that output as the variable's value. This is called **command substitution**.

#### Syntax
```bash
myvar=`command`        # backtick syntax (older, harder to nest)
myvar=$(command)       # modern syntax (preferred — supports nesting easily)
```

#### Example — Using the `date` Command
The `date` command is commonly used to demonstrate command substitution because it accepts format-specifier options.

| `date` Option | Meaning | Example Output |
|---|---|---|
| `+%Y` | 4-digit year | `2026` |
| `+%m` | 2-digit month | `08` |
| `+%d` | 2-digit day | `22` |
| `+%Y-%m-%d` | Full date (ISO format) | `2026-08-22` |
| `+%H:%M:%S` | Current time (24-hr) | `14:05:32` |
| `+%A` | Full weekday name | `Saturday` |

```bash
today=$(date +%Y-%m-%d)
echo $today                 # Output: 2026-08-22

now=$(date +%H:%M:%S)
echo "Current time: $now"   # Output: Current time: 14:05:32

day=$(date +%A)
echo "Today is $day"        # Output: Today is Saturday
```

#### Nested Command Substitution Example
```bash
filecount=$(ls $(pwd) | wc -l)
echo "Files in current directory: $filecount"
```

> ⚠️ **Exam Tip:** Prefer `$()` over backticks — it is easier to read, and nesting backticks requires escaping (`` \` ``), while `$()` can be nested directly.

---

## 2. Accessing Variable Values

### Concept
To read/use the value stored in a variable, prefix its name with `$`. Using curly braces `{}` is recommended, especially when concatenating with other text, to avoid ambiguity.

### Syntax
```bash
echo $myvar
echo ${myvar}
echo "${myvar}_something"
```

### Examples
```bash
myvar="Linux"
echo $myvar              # Output: Linux
echo ${myvar}            # Output: Linux
echo "${myvar}_something"  # Output: Linux_something
echo "$myvar_something"    # Output: (empty) — shell looks for variable "myvar_something" instead!
```

> ⚠️ **Exam Tip:** Always use `${myvar}` when concatenating a variable directly with text (no space/underscore separator issue), since `$myvar_something` is parsed as one variable name `myvar_something`.

#### More Examples — Why `{ }` Matters
```bash
file="report"
echo "$file.txt"        # Output: report.txt   (works fine — "." is not a valid identifier char)
echo "${file}.txt"      # Output: report.txt   (same result, but safer habit)

ext="doc"
echo "$file_$ext"       # Output: (empty) — shell reads "file_" as one variable name (unset)
echo "${file}_${ext}"   # Output: report_doc   (correct — braces isolate each variable)

count=5
echo "There are $count items"    # Output: There are 5 items (space naturally ends the name)
echo "There are ${count}items"   # Output: There are 5items  (braces needed since "s" would merge)
```

> 📌 **Rule of Thumb:** Use `${ }` whenever a variable is immediately followed by a letter, digit, or underscore that could be mistaken as part of the variable's name.

---

## 3. Exporting a Variable

### Concept
By default, a shell variable is **local** to the current shell session and is **not** passed to child processes/subshells. `export` marks a variable as an **environment variable**, making it available to any child process spawned from that shell.

### Syntax
```bash
export myvar="value string"
```
**OR** (declare first, export later)
```bash
myvar="value string"
export myvar
```

### Example
```bash
export PATH="/usr/local/bin:$PATH"   # PATH now updated for this shell and all its children

myname="Aman"
export myname
bash -c 'echo $myname'      # Output: Aman (child shell can access it)
```

### 3.1 Variable Availability to Shells and Subshells

#### Concept
- A **non-exported (local) variable** exists only in the current shell — it is **not visible** inside a subshell or any child process.
- An **exported variable** is copied into the environment of every child process (subshell) created from that point onward.

#### Example
```bash
# Non-exported variable
localvar="I am local"
bash -c 'echo "Inside subshell: $localvar"'    # Output: Inside subshell:  (empty — not visible)

# Exported variable
export globalvar="I am global"
bash -c 'echo "Inside subshell: $globalvar"'   # Output: Inside subshell: I am global
```

#### Summary Table
| Variable Type | Visible in Current Shell? | Visible in Subshell/Child Process? |
|---|---|---|
| Local (not exported) | ✅ Yes | ❌ No |
| Exported | ✅ Yes | ✅ Yes |

### 3.2 Modifying an Exported Variable in a Child Shell

#### Concept
When a variable is exported, the **child process receives a copy** of it — not a reference/link. If the child modifies its copy, the change **stays local to the child** and is **never reflected back** in the parent shell once the child process exits.

#### Example
```bash
export counter="10"

bash -c '
    echo "Child sees: $counter"     # Output: Child sees: 10
    counter="99"
    echo "Child changed to: $counter"   # Output: Child changed to: 99
'

echo "Parent still sees: $counter"   # Output: Parent still sees: 10 (unchanged!)
```

> 📌 **Key Exam Point:** Environment variable inheritance flows **one-way** — from parent → child only. A child process can never modify a variable in its parent's shell environment.

---

## 4. Removing a Variable / Its Value

### 4.1 Removing a Variable Completely

#### Concept
`unset` completely deletes the variable — it no longer exists in the shell.

#### Syntax
```bash
unset myvar
```

### 4.2 Removing Only the Value

#### Concept
Assigning nothing after `=` keeps the variable **defined** but sets its value to an **empty string** (this is different from `unset`, which removes the variable entirely).

#### Syntax
```bash
myvar=
```

### Comparison Table
| Action | Command | Variable still exists? | `${myvar}` value | `[[ -v myvar ]]` result |
|---|---|---|---|---|
| Remove completely | `unset myvar` | ❌ No | (undefined) | Failure (exit 1) |
| Remove value only | `myvar=` | ✅ Yes | `""` (empty) | Success (exit 0) |

---

## 5. Checking Variable Status

### 5.1 Test if a Variable is Set

#### Concept
`-v` flag checks whether a variable name exists (has been declared), regardless of whether its value is empty.

#### Syntax
```bash
[[ -v myvar ]]; echo $?
```

#### Return Codes
| Code | Meaning |
|---|---|
| `0` | Success — variable `myvar` **is set** |
| `1` | Failure — variable `myvar` **is not set** |

#### Example
```bash
myvar="test"
[[ -v myvar ]]; echo $?     # Output: 0

unset myvar
[[ -v myvar ]]; echo $?     # Output: 1
```

### 5.2 Test if a Variable is Not Set

#### Concept
Uses the `${myvar+x}` expansion trick: if `myvar` is set, this expands to the literal string `x` (any placeholder string works); if unset, it expands to nothing (empty). `-z` then tests if the resulting string is empty.

#### Syntax
```bash
[[ -z ${myvar+x} ]]; echo $?
```
> `x` can be **any placeholder string** — it is never actually assigned to `myvar`.

#### Return Codes
| Code | Meaning |
|---|---|
| `0` | Success — variable `myvar` **is not set** |
| `1` | Failure — variable `myvar` **is set** |

#### Example
```bash
unset myvar
[[ -z ${myvar+x} ]]; echo $?    # Output: 0 (not set)

myvar="test"
[[ -z ${myvar+x} ]]; echo $?    # Output: 1 (set)
```

---

## 6. Default Value Handling

These three operators only ever **display or set** default values — they do **not** modify the *original* logic unless explicitly stated.

### 6.1 Substitute Default Value `:-`

#### Concept
If `myvar` is **not set** (or empty), the given default is only **displayed** — the variable itself remains unset/unchanged.

#### Syntax
```bash
echo ${myvar:-"default"}
```
*(no spaces around `:-`)*

#### Logic
| Condition | Behavior |
|---|---|
| `myvar` is set | Display its current value |
| `myvar` is not set | Display `"default"` (variable itself stays unset) |

#### Example
```bash
unset myvar
echo ${myvar:-"Guest"}    # Output: Guest
echo $myvar                # Output: (still empty — myvar unchanged)
```

### 6.2 Set Default Value `:=`

#### Concept
If `myvar` is **not set**, the default value is **assigned to the variable itself** (permanent change) and then displayed.

#### Syntax
```bash
echo ${myvar:="default"}
```

#### Logic
| Condition | Behavior |
|---|---|
| `myvar` is set | Display its current value |
| `myvar` is not set | **Set** `myvar="default"`, then display the new value |

#### Example
```bash
unset myvar
echo ${myvar:="Guest"}    # Output: Guest
echo $myvar                # Output: Guest (myvar is now permanently set)
```

### 6.3 Reset Value if Variable is Set `:+`

#### Concept
The **opposite** logic — if `myvar` **is** already set, replace/reset its value with the given default. If it is unset, display nothing.

#### Syntax
```bash
echo ${myvar:+"default"}
```

#### Logic
| Condition | Behavior |
|---|---|
| `myvar` is set | **Set** `myvar="default"`, then display new value |
| `myvar` is not set | Display `null` (nothing) |

#### Example
```bash
myvar="Original"
echo ${myvar:+"Replaced"}   # Output: Replaced
echo $myvar                  # Output: Replaced (value overwritten)

unset myvar
echo ${myvar:+"Replaced"}   # Output: (empty — nothing displayed)
```

### Quick Comparison Table
| Operator | If SET | If NOT SET | Modifies variable? |
|---|---|---|---|
| `${myvar:-"default"}` | Shows current value | Shows `"default"` only | ❌ No |
| `${myvar:="default"}` | Shows current value | Sets **and** shows `"default"` | ✅ Yes (only when unset) |
| `${myvar:+"default"}` | Sets **and** shows `"default"` | Shows nothing | ✅ Yes (only when set) |

### 6.4 Show Error Message if Variable is Not Set `${var?message}`

#### Concept
Unlike `:-`, `:=`, and `:+` (which substitute or set a default value), this operator **does not substitute anything**. If the variable is unset, it immediately prints a custom **error message** to standard error and, inside a script, **terminates execution** of that script with a non-zero exit status.

#### Syntax
```bash
echo ${myvar?"error message"}
```

#### Logic
| Condition | Behavior |
|---|---|
| `myvar` is set | Display its current value normally |
| `myvar` is not set | Print `bash: myvar: error message` to stderr, and **exit the script** (non-zero status) |

#### Example
```bash
unset dbhost
echo ${dbhost?"Error: dbhost is not set!"}
# Output (to stderr): bash: dbhost: Error: dbhost is not set!
# If run inside a script, the script stops here.

dbhost="localhost"
echo ${dbhost?"Error: dbhost is not set!"}
# Output: localhost   (variable was set, message never triggered)
```

#### Practical Script Example
```bash
#!/bin/bash
: ${API_KEY?"API_KEY must be set before running this script"}
echo "Continuing with API_KEY=$API_KEY"
```
> Here, `:` is the shell's "no-op" (do-nothing) builtin — it is used purely so the `${API_KEY?...}` expansion is evaluated without needing an `echo`.

> 📌 **Exam Point:** This is the **only** default-value-style operator that can **halt script execution** — the other three (`:-`, `:=`, `:+`) never do.

---

## 7. Listing Variable Names

### Concept
Lists the **names** (not values) of all currently defined shell variables that start with a given prefix.

### Syntax
```bash
echo ${!H*}
```
*(Lists names of shell variables starting with `H`)*

### Example
```bash
HOME=/home/user
HOSTNAME=myserver
echo ${!H*}     # Output: HOME HOSTNAME
```

---

## 8. Length of String Value

### Concept
Returns the number of characters stored in a variable's value. If the variable is unset, the length returned is `0`.

### Syntax
```bash
echo ${#myvar}
```

### Example
```bash
myvar="Hello"
echo ${#myvar}     # Output: 5

unset myvar
echo ${#myvar}     # Output: 0
```

---

## 9. Slice of String Value

### Concept
Extracts a **substring** from the variable's value using an offset (starting position) and length (number of characters).

### Syntax
```bash
echo ${myvar:offset:length}
```
Example given in source: `${myvar:5:4}` → skip first 5 characters, then display the next 4 characters.

### Example
```bash
myvar="ShellScripting"
echo ${myvar:5:4}     # Output: Scri  (skips "Shell", takes next 4 chars)
echo ${myvar:0:5}     # Output: Shell (first 5 characters)
echo ${myvar:5}       # Output: Scripting (from offset 5 to end, length omitted)
```

### 9.1 Negative Index (Slicing from the Right)

#### Concept
A **negative offset** counts from the **end** of the string instead of the beginning. This lets you slice relative to the right side of the value without knowing its total length.

#### Syntax
```bash
echo ${myvar: -offset:length}
```
> ⚠️ **Critical Syntax Rule:** A **space is required** before the `-` sign (`: -3`, not `:-3`). Without the space, bash interprets `:-` as the **"substitute default value"** operator (Section 6.1) instead of a negative offset — this is a very common mistake.

#### Example
```bash
myvar="ShellScripting"
echo ${myvar: -3:2}      # Output: in    (start 3 chars from the end, take 2 chars)
echo ${myvar: -6}        # Output: ipting (last 6 characters, to the end)

echo $USER                 # Output: student01 (example username)
echo ${USER: -3:2}         # Output: 0 1  →  e.g. "t0" (last 3 chars, first 2 of that slice)
```

#### Comparison Table
| Expression | Meaning |
|---|---|
| `${myvar:2:2}` | Start at index 2 (from left), take 2 characters |
| `${myvar: -2:2}` | Start 2 characters from the **right**, take 2 characters |
| `${myvar:-"x"}` | **Not slicing at all** — this is the default-value operator (no space before `-`) |

---

## 10. Pattern Removal

### 10.1 Remove Matching Pattern from the Beginning (Prefix)

#### Concept
Removes characters matching a pattern **from the start** of the string.

#### Syntax
```bash
echo ${myvar#pattern}     # removes SHORTEST match from the beginning
echo ${myvar##pattern}    # removes LONGEST possible match from the beginning
```

| Symbol | Match Type |
|---|---|
| `#` | Match once (shortest match) |
| `##` | Match maximum possible (greedy/longest match) |

#### Example
```bash
myvar="/home/user/documents/file.txt"
echo ${myvar#*/}      # Output: home/user/documents/file.txt   (removes up to first /)
echo ${myvar##*/}     # Output: file.txt                        (removes up to last /)
```

### 10.2 Remove Matching Pattern from the End (Suffix)

> **Typo Fix:** The original source material labeled this section *"Keep matching pattern"*, but the operators `%` and `%%` actually **remove** a matching suffix pattern from the end of the string — they do not "keep" anything. Corrected below.

#### Concept
Removes characters matching a pattern **from the end** of the string.

#### Syntax
```bash
echo ${myvar%pattern}     # removes SHORTEST match from the end
echo ${myvar%%pattern}    # removes LONGEST possible match from the end
```

| Symbol | Match Type |
|---|---|
| `%` | Match once (shortest match) |
| `%%` | Match maximum possible (greedy/longest match) |

#### Example
```bash
myvar="file.tar.gz"
echo ${myvar%.*}      # Output: file.tar     (removes shortest suffix from last dot)
echo ${myvar%%.*}     # Output: file          (removes longest suffix from first dot)
```

### Combined Reference Table
| Operator | Direction | Match Length | Example Input | Pattern | Output |
|---|---|---|---|---|---|
| `#` | From start | Shortest | `/a/b/c.txt` | `*/` | `a/b/c.txt` |
| `##` | From start | Longest | `/a/b/c.txt` | `*/` | `c.txt` |
| `%` | From end | Shortest | `file.tar.gz` | `.*` | `file.tar` |
| `%%` | From end | Longest | `file.tar.gz` | `.*` | `file` |

### 10.3 Mixing Prefix and Suffix Removal

#### Concept
`#`/`##` and `%`/`%%` can be **combined in sequence** (chained) on the same variable to strip both a prefix and a suffix in one logical operation — very useful for extracting a specific portion from a path or filename.

#### Example
```bash
path="/home/user/project/archive.tar.gz"

# Step 1: remove everything up to and including the last "/"  → filename only
# Step 2: remove the ".tar.gz" extension from that result
filename=${path##*/}              # Output: archive.tar.gz
name_only=${filename%%.*}          # Output: archive

echo $filename        # Output: archive.tar.gz
echo $name_only        # Output: archive

# Done directly in one line using nested expansion:
echo ${path##*/}                 # Output: archive.tar.gz
echo ${${path##*/}%%.*}            # ❌ Invalid in Bash (nested ${} not allowed like this)

# Correct way to chain — use an intermediate variable, or command substitution:
clean_name=$(basename "$path")     # Output: archive.tar.gz
echo ${clean_name%%.*}              # Output: archive
```

> 📌 **Exam Point:** Bash does **not** allow directly nesting `${ ${var...} ... }`. To apply two pattern operations in sequence, either use an **intermediate variable** (recommended, shown above) or chain through command substitution.

---

## 11. Pattern Replacement

### 11.1 Replace Matching Pattern (Anywhere)

#### Concept
Finds a pattern within the string and replaces it with a given replacement string.

#### Syntax
```bash
echo ${myvar/pattern/string}     # replaces FIRST match only
echo ${myvar//pattern/string}    # replaces ALL matches (global)
```

| Symbol | Behavior |
|---|---|
| `/` | Match once & replace with string |
| `//` | Match max possible & replace with string (global replace) |

#### Example
```bash
myvar="banana"
echo ${myvar/a/O}     # Output: bOnana  (only first "a" replaced)
echo ${myvar//a/O}    # Output: bOnOnO  (all "a" replaced)
```

### 11.2 Replace Matching Pattern by Location

#### Concept
Restricts the replacement to only occur if the pattern is found at the **beginning** or the **end** of the string.

#### Syntax
```bash
echo ${myvar/#pattern/string}     # replace only if match is at the BEGINNING
echo ${myvar/%pattern/string}     # replace only if match is at the END
```

| Symbol | Behavior |
|---|---|
| `/#` | Match at beginning & replace with string |
| `/%` | Match at the end & replace with string |

#### Example
```bash
myvar="report_final.doc"
echo ${myvar/#report/summary}     # Output: summary_final.doc  (matched at start)
echo ${myvar/%doc/pdf}            # Output: report_final.pdf   (matched at end)
```

---

## 12. Changing Case

> **Typo/Ordering Fix:** In the source material, the operator list and their descriptions were misaligned due to PDF layout extraction. The **correct** bash mapping (verified) is restored below.

### Concept
Bash parameter expansion can convert the case of a string's first character or all characters, without needing external tools like `tr`.

### Syntax & Correct Mapping
| Operator | Effect |
|---|---|
| `${myvar,}` | Change **first character** to lower case |
| `${myvar,,}` | Change **all characters** to lower case |
| `${myvar^}` | Change **first character** to upper case |
| `${myvar^^}` | Change **all characters** to upper case |

### Example
```bash
myvar="Hello World"

echo ${myvar,}     # Output: hello World   (first char → lower)
echo ${myvar,,}    # Output: hello world   (all chars → lower)
echo ${myvar^}     # Output: Hello World   (first char → upper, already upper here)
echo ${myvar^^}    # Output: HELLO WORLD   (all chars → upper)

myvar2="linux"
echo ${myvar2^}    # Output: Linux (first char capitalized)
```

---

## 13. Restricting Variable Value Types

### 13.1 Applying Restrictions

#### Concept
The `declare` builtin can enforce specific constraints on what kind of value a variable is allowed to hold.

#### Syntax
```bash
declare -i myvar     # Only integers can be assigned
declare -l myvar     # Only lower case characters assigned (auto-converts)
declare -u myvar      # Only upper case characters assigned (auto-converts)
declare -r myvar      # Variable becomes read-only (cannot be changed or unset)
```

#### Example
```bash
declare -i num
num="10"
echo $num          # Output: 10
num="abc"           # Non-numeric → treated as 0 in arithmetic context
echo $num          # Output: 0

declare -u upname
upname="hello"
echo $upname        # Output: HELLO (auto-converted)

declare -r PI=3.14
PI=3.14159          # Output: Error — readonly variable
```

### 13.2 Removing Restrictions

#### Concept
Restrictions applied via `-i`, `-l`, `-u` can be removed using the `+` prefix instead of `-`. **Exception:** `-r` (read-only) **cannot** be removed once set — this is a one-way restriction.

#### Syntax
```bash
declare +i myvar     # integer restriction removed
declare +l myvar     # lower case restriction removed
declare +u myvar     # upper case restriction removed
declare +r myvar     # ❌ Not possible — read-only cannot be undone
```

#### Example — `declare +u` in Action
```bash
declare -u username
username="admin"
echo $username          # Output: ADMIN (auto-uppercased due to -u)

declare +u username     # restriction removed
username="admin"
echo $username          # Output: admin (no longer force-uppercased)
```

### Restriction Summary Table
| Flag | Applies Restriction | Can be Removed with `+`? |
|---|---|---|
| `-i` | Integer only | ✅ Yes (`declare +i`) |
| `-l` | Lower case only | ✅ Yes (`declare +l`) |
| `-u` | Upper case only | ✅ Yes (`declare +u`) |
| `-r` | Read-only | ❌ **No** — permanent for the life of the variable |

---

## 14. Arrays

### 14.1 Indexed Arrays

#### Concept
An indexed array stores multiple values under a single variable name, accessed using **numeric indices** (starting from `0`).

#### Syntax & Operations
```bash
declare -a arr              # Declare arr as an indexed array
arr[0]="value"               # Set value of element with index 0
echo ${arr[0]}                # Value of element with index 0
echo ${#arr[@]}               # Number of elements in the array
echo ${!arr[@]}               # Display all indices used
echo ${arr[@]}                 # Display values of all elements
unset 'arr[2]'                  # Delete element with index 2
arr+=("value")                   # Append an element to the end of the array
```
> ⚠️ **Typo Fix:** The source PDF showed the assignment as `$arr[0]="value"` — this is incorrect. The `$` sign is **only** used when *reading* a value, never when *assigning* one. Correct assignment syntax is `arr[0]="value"` (no `$`).

#### Operation Reference Table
| Operation | Syntax | Description |
|---|---|---|
| Declare | `declare -a arr` | Declares `arr` as an indexed array |
| Set element | `arr[0]="value"` | Sets value at index 0 |
| Get element | `${arr[0]}` | Gets value at index 0 |
| Count elements | `${#arr[@]}` | Number of elements in array |
| List indices | `${!arr[@]}` | All indices currently used |
| List all values | `${arr[@]}` | All values in the array |
| Delete element | `unset 'arr[2]'` | Removes element at index 2 |
| Append element | `arr+=("value")` | Adds a new value to the end |

#### Example
```bash
declare -a fruits
fruits[0]="Apple"
fruits[1]="Banana"
fruits[2]="Cherry"

echo ${fruits[1]}       # Output: Banana
echo ${#fruits[@]}      # Output: 3
echo ${!fruits[@]}      # Output: 0 1 2
echo ${fruits[@]}       # Output: Apple Banana Cherry

unset 'fruits[1]'
echo ${fruits[@]}       # Output: Apple Cherry
echo ${!fruits[@]}      # Output: 0 2   (index 1 gap remains)

fruits+=("Mango")
echo ${fruits[@]}       # Output: Apple Cherry Mango
```

### 14.1.1 Populating an Array in One Go

#### Concept
Instead of assigning each element one index at a time, an entire indexed array can be initialized in a **single statement** using parentheses, with elements separated by spaces.

#### Syntax
```bash
arr=(element1 element2 element3 ...)
```

#### Example
```bash
colors=("Red" "Green" "Blue" "Yellow")
echo ${colors[@]}          # Output: Red Green Blue Yellow
echo ${#colors[@]}         # Output: 4
echo ${colors[2]}          # Output: Blue

numbers=(10 20 30 40 50)
echo ${numbers[@]}         # Output: 10 20 30 40 50
```

> 📌 **Note:** You can still declare it explicitly first with `declare -a arr=(...)`, though it isn't required — Bash auto-detects an array when parentheses are used.

### 14.1.2 Passing Command Output to an Array

#### Concept
The output of a command (which usually spans multiple lines or words) can be captured directly into an array, with each **word** (default) or each **line** becoming a separate array element.

#### Syntax
```bash
arr=($(command))                     # splits command output by whitespace into elements
mapfile -t arr < <(command)          # splits command output by LINE (preferred, safer)
readarray -t arr < <(command)        # same as mapfile — alternate name
```

#### Example
```bash
# Splitting by whitespace (word-based)
files=($(ls))
echo ${files[@]}              # Output: file1.txt file2.txt script.sh ...
echo ${#files[@]}             # Output: total count of items

# Splitting by line (safer for filenames with spaces)
mapfile -t lines < <(ls -1)
echo ${lines[0]}              # Output: first filename/line
echo ${#lines[@]}             # Output: total number of lines/files

# Practical example: capture users from /etc/passwd
mapfile -t users < <(cut -d: -f1 /etc/passwd)
echo ${users[@]}
```

> ⚠️ **Exam Tip:** `arr=($(command))` splits on **any whitespace** (spaces, tabs, newlines) — this can break filenames containing spaces. `mapfile -t` (or `readarray -t`) splits strictly by **newline**, which is safer for real-world use with `ls`, `cat`, or file lists.

### 14.2 Associative Arrays

#### Concept
An associative array (like a dictionary/hash map) stores values against **string keys** instead of numeric indices. Requires Bash 4.0+.

#### Syntax & Operations
```bash
declare -A hash               # Declare hash as an associative array
hash["a"]="value"              # Set value of element with key "a"
echo ${hash["a"]}               # Value of element with key "a"
echo ${#hash[@]}                 # Number of elements in the array
echo ${!hash[@]}                 # Display all keys used
echo ${hash[@]}                   # Display values of all elements
unset 'hash["a"]'                  # Delete element with key "a"
```
> ⚠️ **Typo Fix:** Same correction as above — source showed `$hash["a"]="value"`; correct assignment is `hash["a"]="value"` (no leading `$`).

#### Operation Reference Table
| Operation | Syntax | Description |
|---|---|---|
| Declare | `declare -A hash` | Declares `hash` as an associative array |
| Set element | `hash["a"]="value"` | Sets value at key `"a"` |
| Get element | `${hash["a"]}` | Gets value at key `"a"` |
| Count elements | `${#hash[@]}` | Number of elements in the array |
| List keys | `${!hash[@]}` | All keys currently used |
| List all values | `${hash[@]}` | All values in the array |
| Delete element | `unset 'hash["a"]'` | Removes element with key `"a"` |

#### Example
```bash
declare -A capitals
capitals["India"]="New Delhi"
capitals["Japan"]="Tokyo"
capitals["France"]="Paris"

echo ${capitals["Japan"]}    # Output: Tokyo
echo ${#capitals[@]}          # Output: 3
echo ${!capitals[@]}          # Output: India Japan France (order not guaranteed)
echo ${capitals[@]}            # Output: New Delhi Tokyo Paris (order not guaranteed)

unset 'capitals["France"]'
echo ${!capitals[@]}          # Output: India Japan
```

### Indexed vs Associative — Quick Comparison
| Feature | Indexed Array | Associative Array |
|---|---|---|
| Declaration | `declare -a arr` | `declare -A hash` |
| Key type | Numeric (0,1,2...) | String |
| Set value | `arr[0]="val"` | `hash["key"]="val"` |
| Auto-append | `arr+=("val")` supported | Must specify key explicitly |
| Bash version needed | Any | Bash 4.0+ |

---

## 15. Summary

- **Variable Basics:** Create with `name=value` (no spaces around `=`); names cannot start with a digit. Use `export` to make a variable available to child processes.
- **Command Substitution:** `myvar=$(command)` (preferred) or `` myvar=`command` `` (legacy) captures a command's output into a variable — commonly demonstrated with `date +FORMAT`.
- **Accessing Values:** Always use `${var}` (curly braces) especially when concatenating text, to avoid ambiguous variable names — the shell will otherwise try to match a longer, unintended variable name.
- **Export & Subshells:** Only **exported** variables are visible inside subshells/child processes; non-exported ones are not. Inheritance is **one-way** — a child modifying its copy of an exported variable never affects the parent shell.
- **Removing Variables:** `unset var` deletes it entirely; `var=` only empties its value while keeping it declared.
- **Status Checks:** `[[ -v var ]]` tests if a variable is set; `[[ -z ${var+x} ]]` tests if it is **not** set — both return `0` for success/true.
- **Default Values:** Four related but distinct operators:
  - `:-` → shows a default without changing the variable.
  - `:=` → sets the variable to the default if unset.
  - `:+` → overwrites the variable's value if it **is** set.
  - `?` → prints a custom error message and halts script execution if the variable is unset.
- **Introspection:** `${!H*}` lists variable names by prefix; `${#var}` gives string length.
- **Substrings:** `${var:offset:length}` extracts a slice of the string; a **negative offset** (`${var: -offset:length}`, note the required space) slices from the right end instead of the left.
- **Pattern Removal:** `#`/`##` strip from the beginning (shortest/longest match); `%`/`%%` strip from the end (shortest/longest match). These can be chained (via an intermediate variable) to strip both a prefix and a suffix.
- **Pattern Replacement:** `/` replaces the first match, `//` replaces all matches; `/#` and `/%` restrict replacement to the beginning or end of the string respectively.
- **Case Conversion:** `,` and `,,` lower the first character / all characters; `^` and `^^` upper the first character / all characters.
- **Type Restrictions:** `declare -i/-l/-u/-r` restrict a variable to integer, lower-case, upper-case, or read-only respectively; all except `-r` can be reversed using `+` instead of `-`.
- **Arrays:** `declare -a` creates numeric-indexed arrays; `declare -A` creates key-based associative arrays. Both support counting (`${#arr[@]}`), listing indices/keys (`${!arr[@]}`), listing values (`${arr[@]}`), and deleting elements (`unset`). Indexed arrays additionally support appending (`+=`), one-shot initialization (`arr=(a b c)`), and direct population from command output (`arr=($(command))` or the safer `mapfile -t arr < <(command)`).
- **Performance Note:** Shell variable and parameter-expansion operations are computationally fast, which is why they are preferred over invoking external utilities (`sed`, `awk`, `cut`) for simple in-shell string manipulation.

---
*End of Notes — Week 5, Lecture 2 & 3: Shell Variables*
