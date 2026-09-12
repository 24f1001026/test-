# Shell Scripting — Complete Exam Reference Notes

> A professional, exam-oriented reference covering shell scripting fundamentals through advanced topics, with syntax, working examples, and common pitfalls for every concept.

---

## Table of Contents

1. [Introduction to Shell Scripting](#1-introduction-to-shell-scripting)
2. [Shell Types and Script Basics](#2-shell-types-and-script-basics)
3. [Variables](#3-variables)
4. [Special Variables and Command-Line Arguments](#4-special-variables-and-command-line-arguments)
5. [Input and Output](#5-input-and-output)
6. [Quoting Rules](#6-quoting-rules)
7. [Operators](#7-operators)
8. [Conditional Statements](#8-conditional-statements)
9. [File Test Operators](#9-file-test-operators)
10. [Loops](#10-loops)
11. [Functions](#11-functions)
12. [Arrays](#12-arrays)
13. [String Manipulation](#13-string-manipulation)
14. [Redirection and Pipes](#14-redirection-and-pipes)
15. [Command Substitution and Arithmetic Expansion](#15-command-substitution-and-arithmetic-expansion)
16. [Exit Status and Error Handling](#16-exit-status-and-error-handling)
17. [Essential Linux Commands with Examples](#17-essential-linux-commands-with-examples)
18. [Regular Expressions](#18-regular-expressions)
19. [Process Management and Signals](#19-process-management-and-signals)
20. [Debugging Shell Scripts](#20-debugging-shell-scripts)
21. [Best Practices](#21-best-practices)
22. [Common Pitfalls Summary](#22-common-pitfalls-summary)
23. [Quick Exam Cheat Sheet](#23-quick-exam-cheat-sheet)

---

## 1. Introduction to Shell Scripting

A **shell** is a command-line interpreter that provides an interface between the user and the operating system kernel. A **shell script** is a text file containing a sequence of shell commands that is executed by the shell interpreter, allowing automation of repetitive tasks.

### Why Shell Scripting?
- Automates repetitive administrative tasks (backups, log rotation, deployments)
- Combines multiple commands into a single reusable unit
- Enables system administration, batch processing, and CI/CD pipelines
- Lightweight — no compilation needed, runs directly through an interpreter

### How a Script Executes
1. The kernel reads the first line of the script (the **shebang**)
2. It invokes the specified interpreter
3. The interpreter reads and executes commands line-by-line

**Pitfall:** Many beginners think shell scripts are "compiled." They are **interpreted line-by-line**, which means a syntax error later in the script won't be caught until execution reaches that line — partial execution can occur before the script fails.

---

## 2. Shell Types and Script Basics

### Common Shells
| Shell | Path | Notes |
|---|---|---|
| Bourne Shell | `/bin/sh` | Original, minimal features |
| Bash (Bourne Again Shell) | `/bin/bash` | Most common, default on Linux |
| Korn Shell | `/bin/ksh` | Bourne-compatible, has arrays |
| C Shell | `/bin/csh` | C-like syntax |
| Z Shell | `/bin/zsh` | Feature-rich, default on macOS |

Check current shell:
```bash
echo $SHELL
echo $0
```

### The Shebang (`#!`)
The **first line** of a script tells the OS which interpreter to use.

```bash
#!/bin/bash
echo "Hello, World!"
```

**Pitfall:** The shebang **must be the very first line**, with no blank line or space before `#!`. If omitted, the script runs using the *current* shell (unpredictable behavior across systems), or fails if executed directly.

**Pitfall:** Using `#!/bin/sh` when you use Bash-only features (like arrays, `[[ ]]`) can break the script on systems where `/bin/sh` is linked to `dash` (not bash), since `dash` doesn't support many bashisms.

### Making a Script Executable
```bash
chmod +x script.sh      # Add execute permission
./script.sh             # Run it (needs ./ if not in PATH)
bash script.sh          # Run without chmod, explicitly invoking bash
sh script.sh            # Run using sh interpreter
```

**Pitfall:** Forgetting `./` before the script name results in `command not found` because the current directory (`.`) is usually **not** in `$PATH` for security reasons.

### Comments
```bash
# This is a single-line comment
: '
This is a
multi-line comment (uses the null command : and a quoted string)
'
```

---

## 3. Variables

### Declaring and Using Variables
```bash
name="Alice"
age=25
echo "$name is $age years old"
```

**Rules:**
- No spaces around `=` (`name = "Alice"` is **WRONG** — treated as a command `name` with arguments `=` and `"Alice"`)
- Variable names: letters, digits, underscores; cannot start with a digit
- Case-sensitive (`Name` ≠ `name`)
- Access value using `$variable` or `${variable}`

**Pitfall #1 (extremely common):**
```bash
name = "Alice"     # ERROR: "name: command not found"
name="Alice"       # CORRECT
```

**Pitfall #2:** Unquoted variables undergo **word splitting** and **globbing**.
```bash
file="my file.txt"
rm $file      # WRONG: tries to remove "my" and "file.txt" separately
rm "$file"    # CORRECT
```

### Variable Scope
```bash
x=10          # Global (in shell context)
function foo {
    local y=20    # Local to function only
    echo "$x $y"
}
```
**Pitfall:** Without `local`, variables inside functions are **global by default** in Bash — unlike most programming languages. This can silently overwrite outer variables.

### Read-Only Variables
```bash
readonly PI=3.14
PI=3.15        # Error: PI: readonly variable
```

### Unsetting Variables
```bash
unset name
```

### Environment Variables vs Shell Variables
```bash
export MY_VAR="value"   # Makes it available to child processes
```
**Pitfall:** A variable NOT exported is only visible in the current shell — child scripts/processes called from it will NOT see it.

### Default/Substitution Values
| Syntax | Meaning |
|---|---|
| `${var:-default}` | Use `default` if `var` is unset or empty (doesn't change var) |
| `${var:=default}` | Assign `default` to `var` if unset or empty |
| `${var:+alt}` | Use `alt` if `var` IS set (otherwise empty) |
| `${var:?message}` | Print error `message` and exit if `var` is unset |

```bash
echo "${name:-Guest}"     # prints "Guest" if $name is unset
: ${count:=0}              # sets count=0 if unset
echo "${DEBUG:+enabled}"   # prints "enabled" only if DEBUG is set
```

---

## 4. Special Variables and Command-Line Arguments

| Variable | Meaning |
|---|---|
| `$0` | Script name |
| `$1, $2, ...` | Positional parameters (arguments) |
| `$#` | Number of arguments passed |
| `$@` | All arguments as **separate** quoted strings |
| `$*` | All arguments as a **single** string |
| `$?` | Exit status of the last command |
| `$$` | PID of current script |
| `$!` | PID of the last background process |
| `$_` | Last argument of the previous command |

### Example
```bash
#!/bin/bash
echo "Script name: $0"
echo "First arg: $1"
echo "Total args: $#"
echo "All args: $@"
```
Run: `./script.sh apple banana` →
```
Script name: ./script.sh
First arg: apple
Total args: 2
All args: apple banana
```

### `$@` vs `$*` — Critical Exam Topic

```bash
for arg in "$@"; do echo "$arg"; done   # Each argument treated separately (CORRECT for loops)
for arg in "$*"; do echo "$arg"; done   # All args as ONE single string
```

**Pitfall:** `"$@"` and `"$*"` behave identically when **unquoted**, but differ significantly when **quoted**:
- `"$@"` → expands to `"$1" "$2" "$3"` (preserves each argument, even with spaces)
- `"$*"` → expands to `"$1 $2 $3"` (single merged string using first char of `IFS`)

This is a **favorite exam trick question**.

### `shift` command
Shifts positional parameters to the left.
```bash
#!/bin/bash
while [ $# -gt 0 ]; do
    echo "Processing: $1"
    shift
done
```

---

## 5. Input and Output

### `echo`
```bash
echo "Hello"                 # Basic output
echo -n "No newline"         # -n: suppress trailing newline
echo -e "Tab:\tNewline:\n"   # -e: enable interpretation of backslash escapes
```
**Pitfall:** `echo` behavior for `-e`/escape sequences is **not portable** across shells (works differently in `sh` vs `bash`). Use `printf` for reliable formatting.

### `printf` (preferred for formatting)
```bash
printf "Name: %s, Age: %d\n" "Alice" 25
printf "%-10s|%5d\n" "Item" 42     # left-align string (10 wide), right-align int (5 wide)
```

### `read` — taking user input
```bash
read -p "Enter your name: " name    # -p: show prompt
echo "Hello, $name"

read -s -p "Password: " pass        # -s: silent (hide input)
echo

read -t 5 -p "Enter within 5 sec: " ans   # -t: timeout in seconds

read -a arr -p "Enter values: "     # -a: read into an array
```

**Pitfall:** `read` without `-r` will **interpret backslashes** as escape characters, mangling input like file paths (`C:\new` becomes `C:new`). Always prefer `read -r`.

```bash
read -r line     # CORRECT — treats backslashes literally
```

**Pitfall:** Piping into a `while read` loop runs the loop in a **subshell**, so variables set inside won't persist outside:
```bash
count=0
cat file.txt | while read -r line; do
    count=$((count+1))
done
echo "$count"   # Prints 0! Because pipe created a subshell

# FIX: use process substitution
while read -r line; do
    count=$((count+1))
done < file.txt
echo "$count"   # Correct count
```

---

## 6. Quoting Rules

| Quote Type | Behavior |
|---|---|
| `'single quotes'` | **Literal** — no variable expansion, no command substitution |
| `"double quotes"` | Allows variable expansion (`$var`) and command substitution (`` `cmd` `` or `$(cmd)`) |
| `` `backticks` `` or `` \`escape\` `` | Escape a single special character |

```bash
name="Alice"
echo 'Hello $name'     # Output: Hello $name  (literal)
echo "Hello $name"     # Output: Hello Alice  (expanded)
echo Hello\ $name      # Output: Hello Alice (backslash escapes the space)
```

**Pitfall:** Forgetting to quote variables containing spaces or wildcards causes **word splitting** and **filename expansion (globbing)**:
```bash
var="*.txt"
echo $var      # Expands to list of matching files in cwd!
echo "$var"    # Prints literally: *.txt
```

**Rule of thumb for exams:** *Always double-quote variable expansions* (`"$var"`) unless you specifically want word-splitting/globbing.

---

## 7. Operators

### Arithmetic Operators
Used inside `$(( ))`, `let`, or with `-lt`, `-gt` etc. in `[ ]`.

| Operator | Meaning |
|---|---|
| `+ - * / %` | add, subtract, multiply, divide, modulus |
| `**` | exponentiation (bash only) |
| `++ --` | increment/decrement (inside `(( ))`) |

```bash
a=10; b=3
echo $((a + b))     # 13
echo $((a % b))     # 1
echo $((a ** 2))    # 100
((a++))             # increment a
let "c = a + b"     # let command for arithmetic
```

**Pitfall:** `/` performs **integer division** in bash — `echo $((7/2))` gives `3`, NOT `3.5`. For floating point, use `bc` or `awk`:
```bash
echo "7 / 2" | bc -l      # 3.50000000000000000000
awk "BEGIN {print 7/2}"   # 3.5
```

### Relational (Numeric Comparison) Operators — used inside `[ ]` or `test`
| Operator | Meaning |
|---|---|
| `-eq` | equal to |
| `-ne` | not equal to |
| `-gt` | greater than |
| `-lt` | less than |
| `-ge` | greater than or equal |
| `-le` | less than or equal |

```bash
if [ "$a" -eq "$b" ]; then echo "equal"; fi
```

**Pitfall:** Using `==` or `=` (string operators) for numbers, or `-eq` for strings, causes wrong or erroring comparisons. `-eq` etc. are **only for integers**.

### String Comparison Operators
| Operator | Meaning |
|---|---|
| `=` or `==` | equal (== is bash-specific, non-POSIX) |
| `!=` | not equal |
| `-z` | string is empty (zero length) |
| `-n` | string is not empty |
| `<` `>` | lexicographic comparison (inside `[[ ]]`, must be escaped in `[ ]`) |

```bash
if [ "$str1" = "$str2" ]; then echo "same"; fi
if [ -z "$str" ]; then echo "empty string"; fi
```

**Pitfall:** `>` and `<` inside single `[ ]` are interpreted as **redirection operators**, not comparisons! You must use `[[ ]]` or escape them: `[ "$a" \> "$b" ]`.

### Logical Operators
| Operator | Meaning |
|---|---|
| `&&` | AND |
| `\|\|` | OR |
| `!` | NOT |
| `-a` | AND (inside old `[ ]` test — deprecated) |
| `-o` | OR (inside old `[ ]` test — deprecated) |

```bash
if [ "$a" -gt 5 ] && [ "$a" -lt 10 ]; then echo "in range"; fi
if [[ $a -gt 5 && $a -lt 10 ]]; then echo "in range"; fi
```

**Pitfall:** `-a`/`-o` inside `[ ]` are officially deprecated and can behave unpredictably with complex expressions — prefer separate `[ ]` tests joined by `&&`/`||`, or use `[[ ]]`.

### Assignment / Compound Operators
```bash
a=5
a+=3       # a = a + 3 -> 8 (arithmetic context) but string concatenation in non-numeric context
```

---

## 8. Conditional Statements

### `if / elif / else`
```bash
#!/bin/bash
num=10
if [ "$num" -gt 0 ]; then
    echo "Positive"
elif [ "$num" -lt 0 ]; then
    echo "Negative"
else
    echo "Zero"
fi
```

**Syntax rules (exam-critical):**
- Space required after `[` and before `]`: `[ "$num" -gt 0 ]` NOT `["$num" -gt 0]`
- `then` can be on the same line only if preceded by `;`
- Must close with `fi`

**Pitfall:** `if [ $num -gt 0 ]` (unquoted) fails/errors when `$num` is empty or unset because it becomes `[ -gt 0 ]` — a syntax error. Always quote: `[ "$num" -gt 0 ]`.

### `[ ]` vs `[[ ]]` vs `(( ))`
| Construct | Type | Notes |
|---|---|---|
| `[ ]` | POSIX `test` command | Portable, but word-splitting/globbing risk if unquoted |
| `[[ ]]` | Bash keyword | No word splitting, supports `&&`, `\|\|`, pattern matching, `=~` regex |
| `(( ))` | Arithmetic evaluation | For numeric conditions/expressions only |

```bash
[[ $str == a* ]]         # pattern matching (glob-style) works only in [[ ]]
[[ $str =~ ^[0-9]+$ ]]    # regex matching works only in [[ ]]
(( a > b && b > c ))      # arithmetic — no $ needed inside (( ))
```

**Pitfall:** Exam favorite — `[[ ]]` is a **Bash-only** feature. Scripts using `#!/bin/sh` may fail on systems where `sh` ≠ `bash` (e.g., Debian/Ubuntu's `dash`).

### `case` Statement
```bash
#!/bin/bash
read -p "Enter a fruit: " fruit
case $fruit in
    apple)
        echo "Apple selected"
        ;;
    banana|mango)
        echo "Banana or Mango selected"
        ;;
    a*)
        echo "Starts with a"
        ;;
    *)
        echo "Unknown fruit"
        ;;
esac
```
**Notes:**
- Each pattern block ends with `;;`
- `|` separates multiple patterns for one action
- `*` is the default/catch-all case
- Supports glob patterns (`*`, `?`, `[abc]`)

**Pitfall:** Forgetting `;;` after a block causes fall-through into the next pattern's commands (unlike C's switch, bash requires explicit `;;` per case — but note bash 4+ also supports `;&` for intentional fallthrough and `;;&` to continue testing).

---

## 9. File Test Operators

Used inside `[ ]` or `[[ ]]` to check file properties.

| Operator | Meaning |
|---|---|
| `-e file` | Exists |
| `-f file` | Regular file |
| `-d file` | Directory |
| `-L file` | Symbolic link |
| `-r file` | Readable |
| `-w file` | Writable |
| `-x file` | Executable |
| `-s file` | Size > 0 (not empty) |
| `-z string` | String length is zero |
| `file1 -nt file2` | file1 newer than file2 |
| `file1 -ot file2` | file1 older than file2 |

```bash
if [ -f "$file" ]; then
    echo "Regular file exists"
elif [ -d "$file" ]; then
    echo "It's a directory"
else
    echo "Does not exist"
fi
```

**Pitfall:** `-e` checks existence of **any** file type (file, dir, symlink target); if you specifically need a regular file, use `-f`, otherwise directories will also pass the check silently.

---

## 10. Loops

### `for` loop (list form)
```bash
for i in 1 2 3 4 5; do
    echo "Number: $i"
done

for fruit in apple banana mango; do
    echo "$fruit"
done

for file in *.txt; do        # glob expansion
    echo "$file"
done
```

### `for` loop (C-style)
```bash
for (( i=0; i<5; i++ )); do
    echo "i = $i"
done
```

### `for` with range
```bash
for i in {1..10}; do echo "$i"; done
for i in {1..10..2}; do echo "$i"; done   # step of 2
for i in $(seq 1 2 10); do echo "$i"; done  # portable alternative
```
**Pitfall:** Brace expansion `{1..10}` does **not** work with variables directly: `for i in {1..$n}` will NOT expand as expected (prints literal `{1..5}` if $n=5). Use `seq` or C-style loop instead when bounds are variables.

### `while` loop
```bash
count=1
while [ "$count" -le 5 ]; do
    echo "Count: $count"
    ((count++))
done
```

### `until` loop (runs until condition becomes TRUE — opposite of while)
```bash
count=1
until [ "$count" -gt 5 ]; do
    echo "Count: $count"
    ((count++))
done
```

### Loop control: `break` and `continue`
```bash
for i in {1..10}; do
    if [ "$i" -eq 5 ]; then break; fi
    if [ $((i % 2)) -eq 0 ]; then continue; fi
    echo "$i"
done
```
**Pitfall:** `break 2` / `continue 2` breaks/continues out of **nested loops** by N levels — commonly missed in exams.

### `select` loop (creates simple menus)
```bash
select option in "Start" "Stop" "Exit"; do
    case $option in
        "Start") echo "Starting...";;
        "Stop") echo "Stopping...";;
        "Exit") break;;
        *) echo "Invalid option";;
    esac
done
```

---

## 11. Functions

### Defining and Calling
```bash
# Method 1
greet() {
    echo "Hello, $1!"
}

# Method 2
function greet2 {
    echo "Hi, $1!"
}

greet "Alice"     # Call with argument
```

### Return Values
```bash
add() {
    local sum=$(( $1 + $2 ))
    echo "$sum"          # "return" data via echo + command substitution
    return 0              # return is for EXIT STATUS only (0-255), NOT data
}

result=$(add 5 10)
echo "Sum is: $result"
```

**Pitfall (major):** `return` in bash functions can ONLY return an **integer exit status (0–255)**, not arbitrary data like strings or large numbers. To "return" a value, either:
1. `echo` the value and capture with `$(function_name)`, or
2. Assign to a global variable inside the function

```bash
# WRONG assumption:
myfunc() { return "hello"; }   # ERROR: numeric argument required

# WRONG:
myfunc() { return 300; }       # 300 wraps around modulo 256 -> exit status becomes 44
```

### Passing Arguments and Local Variables
```bash
myfunc() {
    local x=$1
    local y=$2
    echo "$((x + y))"
}
```
**Pitfall:** Omitting `local` means variables leak into global scope and can silently clobber same-named variables elsewhere in the script.

### Recursive Functions
```bash
factorial() {
    if [ "$1" -le 1 ]; then
        echo 1
    else
        local prev=$(factorial $(( $1 - 1 )))
        echo $(( $1 * prev ))
    fi
}
echo "$(factorial 5)"   # 120
```

---

## 12. Arrays

### Indexed Arrays
```bash
arr=(apple banana mango)
arr[3]="grape"

echo "${arr[0]}"        # First element: apple
echo "${arr[@]}"        # All elements
echo "${#arr[@]}"       # Length of array: 4
echo "${!arr[@]}"       # List of indices: 0 1 2 3
```

### Modifying Arrays
```bash
arr+=("orange")          # Append element
unset arr[1]             # Remove element at index 1 (leaves a gap!)
arr=("${arr[@]}")        # Re-index array (removes gaps)
```

### Slicing
```bash
echo "${arr[@]:1:2}"     # 2 elements starting from index 1
```

### Associative Arrays (Bash 4+, requires declaration)
```bash
declare -A capitals
capitals[India]="New Delhi"
capitals[USA]="Washington DC"

echo "${capitals[India]}"
for key in "${!capitals[@]}"; do
    echo "$key -> ${capitals[$key]}"
done
```

**Pitfall:** Associative arrays **require** `declare -A` before use — without it, bash treats it as a regular indexed array and keys like `capitals[India]` silently become `capitals[0]` (since "India" evaluates to 0 in arithmetic context).

**Pitfall:** `"${arr[@]}"` (quoted with @) preserves elements with spaces as separate items; `"${arr[*]}"` merges all into one string — same distinction as `$@` vs `$*`.

---

## 13. String Manipulation

### Length
```bash
str="Hello World"
echo "${#str}"           # 11
```

### Substring Extraction
```bash
echo "${str:0:5}"        # "Hello" (offset:length)
echo "${str:6}"          # "World" (from offset to end)
echo "${str: -5}"        # "World" (negative offset — note the required space before -5)
```
**Pitfall:** `${str:-5}` (no space) is interpreted as the **default-value** operator (Section 3), NOT substring extraction! You MUST write `${str: -5}` with a space to get the last 5 characters.

### Replace
```bash
echo "${str/World/Bash}"    # Replace first match: "Hello Bash"
echo "${str//o/0}"          # Replace ALL matches: "Hell0 W0rld"
echo "${str/#Hello/Hi}"     # Replace only if match is at the START
echo "${str/%World/Bash}"   # Replace only if match is at the END
```

### Case Conversion (Bash 4+)
```bash
echo "${str,,}"     # lowercase all: "hello world"
echo "${str^^}"     # UPPERCASE all: "HELLO WORLD"
echo "${str,}"      # lowercase first char
echo "${str^}"      # uppercase first char
```

### Delete Pattern (trim)
```bash
str="  Hello  "
echo "${str#"${str%%[![:space:]]*}"}"   # trim leading whitespace (advanced)
```

### Prefix/Suffix Removal
| Syntax | Meaning |
|---|---|
| `${var#pattern}` | Remove shortest match from **front** |
| `${var##pattern}` | Remove longest match from **front** |
| `${var%pattern}` | Remove shortest match from **end** |
| `${var%%pattern}` | Remove longest match from **end** |

```bash
path="/usr/local/bin/script.sh"
echo "${path##*/}"     # "script.sh" -> basename
echo "${path%/*}"      # "/usr/local/bin" -> dirname
filename="archive.tar.gz"
echo "${filename%.*}"  # "archive.tar" (removes shortest suffix match)
echo "${filename%%.*}" # "archive" (removes longest suffix match)
```
**Pitfall:** `#` vs `##` and `%` vs `%%` (short vs long/greedy match) is a **very common exam trick** — mixing them up gives wrong results with multi-dot filenames.

### Splitting Strings into Arrays
```bash
IFS=',' read -ra parts <<< "a,b,c"
echo "${parts[1]}"    # b
```

---

## 14. Redirection and Pipes

| Symbol | Meaning |
|---|---|
| `>` | Redirect stdout, **overwrite** file |
| `>>` | Redirect stdout, **append** to file |
| `<` | Redirect stdin from file |
| `2>` | Redirect stderr |
| `2>>` | Append stderr |
| `&>` or `> file 2>&1` | Redirect BOTH stdout and stderr |
| `2>&1` | Redirect stderr to same place as stdout |
| `<<` | Here-document (multi-line input) |
| `<<<` | Here-string (single-line input) |
| `\|` | Pipe output of one command into another |
| `/dev/null` | Discard output (the "black hole" device) |

```bash
echo "log entry" >> logfile.txt        # append
command > output.txt 2> error.txt      # separate stdout & stderr
command > all_output.txt 2>&1          # combine into one file (order matters!)
command &> all_output.txt              # bash shortcut for the above

grep "error" logfile.txt | sort | uniq -c   # pipe chain

cat <<EOF
Multi-line
text block
EOF

grep "pattern" <<< "$variable"          # here-string
```

**Pitfall (order matters!):**
```bash
command > file 2>&1    # CORRECT: stdout->file, then stderr follows stdout (to file)
command 2>&1 > file    # WRONG: stderr goes to terminal (current stdout), THEN stdout redirected to file
```

**Pitfall:** `>` truncates/overwrites a file **immediately** when the command starts, even if the command fails afterward — you can lose original data. Use `>>` if you want to preserve existing content.

---

## 15. Command Substitution and Arithmetic Expansion

### Command Substitution
```bash
current_date=$(date)          # Modern, preferred syntax
current_date2=`date`          # Old backtick syntax (avoid — hard to nest, less readable)
files=$(ls *.txt)
echo "Today is $current_date"
```
**Pitfall:** Backticks cannot be nested easily; `$(...)` can be nested cleanly: `$(echo $(date))`. Prefer `$()` always.

### Arithmetic Expansion
```bash
result=$((5 + 3))
echo "$result"                # 8
echo $(( (a + b) * 2 ))
```

### `let` and `expr` (older alternatives)
```bash
let result=5+3
expr 5 + 3                    # NOTE: spaces required around operators with expr
```
**Pitfall:** `expr` requires spaces between every token and operator (`expr 5+3` fails, `expr 5 + 3` works); also `*` must be escaped in `expr` (`expr 5 \* 3`) since it's a glob character to the shell.

---

## 16. Exit Status and Error Handling

Every command returns an **exit status** (0–255) stored in `$?`. `0` = success, non-zero = failure/error.

```bash
grep "pattern" file.txt
echo "$?"       # 0 if found, 1 if not found, 2 if error (e.g., file missing)
```

### `exit` command
```bash
exit 0     # success
exit 1     # generic error
```
**Pitfall:** `exit` values greater than 255 wrap around using modulo 256 (`exit 256` becomes exit status `0`).

### Conditional execution with `&&` and `||`
```bash
mkdir newdir && cd newdir        # cd runs only if mkdir succeeds
command || echo "Command failed"  # echo runs only if command fails
```

### `set` options for robust scripts (very important for exams)
```bash
set -e     # Exit immediately if any command fails (non-zero exit)
set -u     # Treat unset variables as an error
set -x     # Print each command before executing (debug trace)
set -o pipefail   # Make a pipeline fail if ANY command in it fails, not just the last
```
Combined (common professional idiom):
```bash
set -euo pipefail
```

**Pitfall:** Without `pipefail`, `cmd1 | cmd2` only reports the exit status of `cmd2` — even if `cmd1` failed, the pipeline may report success.

**Pitfall:** `set -e` does NOT trigger on failures inside `if` conditions, `&&`/`||` chains, or commands whose exit code is checked — this surprises many exam-takers who expect it to catch everything.

### `trap` for error handling (see also Section 19)
```bash
trap 'echo "Error occurred at line $LINENO"; exit 1' ERR
```

---

## 17. Essential Linux Commands with Examples

### `grep` — search text using patterns
```bash
grep "error" file.txt              # Basic search
grep -i "error" file.txt           # Case-insensitive
grep -v "error" file.txt           # Invert match (lines NOT containing pattern)
grep -c "error" file.txt           # Count matching lines
grep -r "TODO" ./src               # Recursive search in directory
grep -n "error" file.txt           # Show line numbers
grep -E "err(or|ors)" file.txt     # Extended regex (egrep)
grep -w "cat" file.txt             # Match whole word only
```
**Pitfall:** By default `grep` uses **Basic Regular Expressions (BRE)** where `+`, `?`, `|`, `()` are literal unless escaped. Use `-E` (or `egrep`) for Extended Regex to use them unescaped.

### `sed` — stream editor (find & replace, line editing)
```bash
sed 's/foo/bar/' file.txt          # Replace FIRST occurrence per line
sed 's/foo/bar/g' file.txt         # Replace ALL occurrences (global)
sed -i 's/foo/bar/g' file.txt      # Edit file IN PLACE
sed -n '2,4p' file.txt             # Print only lines 2-4
sed '/pattern/d' file.txt          # Delete lines matching pattern
sed '3d' file.txt                  # Delete line 3
```
**Pitfall:** `sed -i` (in-place edit) on Linux (GNU sed) vs macOS (BSD sed) differ: BSD requires an explicit backup extension argument `sed -i '' 's/a/b/' file` (empty string), while GNU allows `sed -i 's/a/b/' file` directly. Script portability breaks here often.

### `awk` — pattern scanning and processing (field-based)
```bash
awk '{print $1}' file.txt              # Print first column/field
awk -F',' '{print $2}' file.csv        # Use comma as field separator
awk '{print NF}' file.txt              # Number of fields per line
awk '{print NR, $0}' file.txt          # Print line number + full line
awk '$3 > 50 {print $1}' file.txt      # Conditional filtering
awk 'BEGIN{sum=0} {sum+=$1} END{print sum}' file.txt   # Sum a column
```
**Pitfall:** Default field separator is whitespace (spaces/tabs) — for CSVs with commas you MUST specify `-F','`, otherwise entire lines are treated as one field.

### `cut` — extract columns
```bash
cut -d',' -f1,3 file.csv     # Extract fields 1 and 3, comma-delimited
cut -c1-5 file.txt            # Extract characters 1 to 5
```

### `sort`
```bash
sort file.txt                 # Alphabetical sort
sort -n file.txt               # Numeric sort
sort -r file.txt               # Reverse order
sort -k2 file.txt               # Sort by 2nd column/field
sort -u file.txt                # Sort and remove duplicates
```
**Pitfall:** Without `-n`, sorting numbers gives **lexicographic** order (e.g., "10" comes before "2") because sort treats input as text by default.

### `uniq` — remove/report duplicate lines (requires sorted input!)
```bash
sort file.txt | uniq            # Remove adjacent duplicates
sort file.txt | uniq -c         # Count occurrences
sort file.txt | uniq -d         # Show only duplicated lines
```
**Pitfall:** `uniq` only removes **consecutive/adjacent** duplicate lines. Input must be sorted first, or non-adjacent duplicates will remain.

### `tr` — translate/delete characters
```bash
echo "hello" | tr 'a-z' 'A-Z'      # Convert to uppercase
echo "hello world" | tr -d 'l'      # Delete all 'l' characters
echo "a  b   c" | tr -s ' '          # Squeeze repeated spaces into one
```

### `wc` — word/line/character count
```bash
wc -l file.txt      # Line count
wc -w file.txt      # Word count
wc -c file.txt      # Byte count
```

### `find` — search files/directories
```bash
find . -name "*.txt"                  # Find by name pattern
find . -type f -size +1M              # Files larger than 1MB
find . -type d -name "temp"           # Find directories named temp
find . -mtime -7                      # Modified in last 7 days
find . -name "*.log" -exec rm {} \;   # Find and execute a command on each result
find . -name "*.tmp" -delete          # Find and delete directly
```
**Pitfall:** `-exec cmd {} \;` runs the command **once per file** (slow for large sets); `-exec cmd {} +` batches multiple files into fewer command invocations (much faster, like `xargs`).

### `xargs` — build/execute commands from input
```bash
find . -name "*.log" | xargs rm             # Pass find results as arguments to rm
echo "file1 file2" | xargs touch
cat urls.txt | xargs -n1 curl -O            # Run curl once per line (-n1: 1 arg per invocation)
find . -name "*.txt" -print0 | xargs -0 rm   # Handle filenames with spaces safely
```
**Pitfall:** Without `-print0`/`-0` pairing, filenames containing spaces or newlines get split incorrectly by `xargs`. Always use `find ... -print0 | xargs -0 ...` for safety with unusual filenames.

### `ps`, `top` — process monitoring
```bash
ps aux                  # List all running processes
ps aux | grep firefox   # Find specific process
top                      # Real-time process monitor
```

### `chmod` / `chown` — permissions
```bash
chmod 755 script.sh        # rwxr-xr-x (owner: rwx, group: r-x, others: r-x)
chmod u+x script.sh        # Add execute permission for user/owner only
chown user:group file.txt   # Change ownership
```
Permission digits: `4=read, 2=write, 1=execute` (sum them: `7=rwx, 5=r-x, 6=rw-`).

### `tar` — archiving
```bash
tar -cvf archive.tar folder/       # Create archive
tar -xvf archive.tar               # Extract archive
tar -czvf archive.tar.gz folder/   # Create gzip-compressed archive
tar -xzvf archive.tar.gz           # Extract gzip-compressed archive
```

### `curl` / `wget` — network requests
```bash
curl -O https://example.com/file.zip     # Download, keep filename
curl -s https://api.example.com/data      # Silent mode (for scripting)
wget https://example.com/file.zip         # Alternative downloader
```

### `crontab` — task scheduling
```bash
crontab -e                  # Edit cron jobs
crontab -l                  # List cron jobs
# Format: minute hour day month weekday command
0 2 * * * /home/user/backup.sh   # Run daily at 2:00 AM
```

---

## 18. Regular Expressions

### Basic Metacharacters
| Symbol | Meaning |
|---|---|
| `.` | Any single character |
| `*` | Zero or more of the preceding character |
| `+` | One or more (needs `-E` in grep/needs escaping in BRE) |
| `?` | Zero or one (needs `-E`/escaping in BRE) |
| `^` | Start of line |
| `$` | End of line |
| `[abc]` | Any one of a, b, c |
| `[^abc]` | Any character EXCEPT a, b, c |
| `[0-9]` | Any digit |
| `\|` | OR (alternation, needs `-E`/escaping in BRE) |
| `()` | Grouping (needs `-E`/escaping in BRE) |
| `{n,m}` | Between n and m repetitions |
| `\d`, `\w`, `\s` | Digit, word char, whitespace (PCRE only — use `grep -P`) |

### Examples
```bash
grep "^[A-Z]" file.txt          # Lines starting with uppercase letter
grep "[0-9]\{3\}" file.txt      # BRE: exactly 3 digits (braces escaped)
grep -E "[0-9]{3}" file.txt     # ERE: same, no escaping needed
grep -E "^(cat|dog)$" file.txt  # Exact match "cat" or "dog"
grep -P "\d+" file.txt          # PCRE: Perl-compatible regex (GNU grep only)
```

### Bash's Built-in Regex Match `=~` (inside `[[ ]]`)
```bash
str="hello123"
if [[ $str =~ ^[a-z]+[0-9]+$ ]]; then
    echo "Matched"
fi
```
**Pitfall:** Inside `[[ $str =~ pattern ]]`, do **NOT** quote the pattern if it contains regex metacharacters intended to be interpreted as regex — quoting forces literal string matching instead of regex.

---

## 19. Process Management and Signals

### Foreground / Background Jobs
```bash
sleep 100 &         # Run in background, prints job number & PID
jobs                # List background jobs
fg %1                # Bring job 1 to foreground
bg %1                # Resume job 1 in background
wait                 # Wait for all background jobs to finish
wait $!              # Wait for the last background process specifically
```

### Killing Processes
```bash
kill PID              # Send SIGTERM (graceful termination request)
kill -9 PID            # Send SIGKILL (force kill, cannot be caught/ignored)
kill -l                # List all available signals
killall firefox        # Kill by process name
```

**Pitfall:** `kill -9` (SIGKILL) does NOT allow the process to clean up (close files, release locks) — data corruption is possible. Prefer `kill` (SIGTERM) first, and only use `-9` as a last resort.

### `trap` — handling signals inside scripts
```bash
trap 'echo "Interrupted! Cleaning up..."; exit 1' SIGINT SIGTERM

trap 'rm -f "$tempfile"' EXIT    # Cleanup on any script exit (success or failure)
```
Common signals:
| Signal | Number | Meaning |
|---|---|---|
| SIGHUP | 1 | Hangup (terminal closed) |
| SIGINT | 2 | Interrupt (Ctrl+C) |
| SIGKILL | 9 | Force kill (cannot be trapped) |
| SIGTERM | 15 | Graceful terminate request (default of `kill`) |
| EXIT | — | Bash pseudo-signal: fires on script exit, useful for cleanup |

**Pitfall:** `SIGKILL` (9) and `SIGSTOP` (19) **cannot be trapped or ignored** by a script — they are handled directly by the kernel.

---

## 20. Debugging Shell Scripts

```bash
bash -x script.sh          # Trace execution: print each command before running it
bash -n script.sh          # Syntax check only (no execution)
bash -v script.sh          # Print each line as it's read (verbose)
```

Inline debugging:
```bash
set -x     # Turn ON tracing from this point
# ... commands ...
set +x     # Turn OFF tracing
```

Adding debug print statements:
```bash
echo "DEBUG: variable value is '$var'" >&2   # Send to stderr, not mixed with normal output
```

**Pitfall:** Debug output sent to stdout can interfere with a script's actual output (especially if that output is piped/captured by another program). Always send debug/log messages to **stderr** (`>&2`).

---

## 21. Best Practices

1. **Always start scripts with a shebang**: `#!/bin/bash`
2. **Use `set -euo pipefail`** at the top for safer, fail-fast scripts
3. **Always double-quote variables**: `"$var"` not `$var`
4. **Use `[[ ]]` over `[ ]`** in Bash scripts for safety and extra features
5. **Use `local`** for all function-internal variables
6. **Use `$( )`** instead of backticks for command substitution
7. **Check exit statuses** of critical commands (`if ! command; then ...`)
8. **Use meaningful variable names** and add comments for complex logic
9. **Validate input arguments** at the start of scripts (`$#` check)
10. **Use `mktemp`** for temporary files rather than hardcoded paths
11. **Use `trap ... EXIT`** to guarantee cleanup of temp files/resources
12. **Test scripts with `shellcheck`** (a static analysis tool) before submission/deployment
13. **Prefer `printf` over `echo`** for predictable, portable formatting

---

## 22. Common Pitfalls Summary

| # | Pitfall | Fix |
|---|---|---|
| 1 | Spaces around `=` in assignment | `var="value"` (no spaces) |
| 2 | Unquoted variables causing word-splitting/globbing | Always use `"$var"` |
| 3 | Using `==`/`-eq` for wrong data type | `-eq` for numbers, `=`/`==` for strings |
| 4 | `[ ]` vs `[[ ]]` bash-only features | Use `[[ ]]` for regex/pattern matching, `&&`/`\|\|` |
| 5 | `>` vs `<` misinterpreted as redirection in `[ ]` | Escape or use `[[ ]]` |
| 6 | Forgetting `local` in functions | Global leakage of variables |
| 7 | `return` used for data, not just exit status | Use `echo` + command substitution instead |
| 8 | `$@` vs `$*` confusion | `"$@"` preserves individual args |
| 9 | Piping into `while read` loses variables (subshell) | Use input redirection `< file` instead |
| 10 | `read` without `-r` mangles backslashes | Always use `read -r` |
| 11 | Integer-only division in bash arithmetic | Use `bc` or `awk` for floats |
| 12 | `uniq` needs sorted input | Pipe through `sort` first |
| 13 | `find -exec` with `\;` is slow | Use `+` or pipe to `xargs` |
| 14 | `${str:-5}` vs `${str: -5}` (default val vs substring) | Add a space before negative offset |
| 15 | `#`/`##` and `%`/`%%` mixed up | `#`/`%` = shortest match, `##`/`%%` = longest/greedy |
| 16 | Associative array without `declare -A` | Always declare explicitly |
| 17 | `set -e` doesn't catch all failures | Doesn't trigger inside conditionals/`&&`/`||` chains |
| 18 | Pipeline exit status hides earlier failures | Use `set -o pipefail` |
| 19 | Missing `./` to run local script | `./script.sh`, not just `script.sh` |
| 20 | `#!/bin/sh` but using bashisms | Match shebang to actual syntax used |
| 21 | `kill -9` prevents cleanup | Prefer plain `kill` (SIGTERM) first |
| 22 | Debug/log output mixed into stdout | Redirect debug messages to stderr (`>&2`) |

---

## 23. Quick Exam Cheat Sheet

```bash
#!/bin/bash
set -euo pipefail

# Variables
name="Alice"; age=25

# Conditionals
if [[ "$age" -ge 18 ]]; then echo "Adult"; fi

# Loops
for i in {1..5}; do echo "$i"; done
while [ "$count" -le 5 ]; do ((count++)); done

# Functions
greet() { local n="$1"; echo "Hi, $n"; }

# Arrays
arr=(a b c); echo "${arr[@]}"; echo "${#arr[@]}"

# String ops
echo "${name^^}"          # uppercase
echo "${name:0:2}"        # substring

# Command substitution
today=$(date +%F)

# Redirection
cmd > out.txt 2>&1

# Exit status
command && echo ok || echo fail
```

### Key Numeric Test Flags
`-eq -ne -gt -lt -ge -le` (numbers) vs `= != -z -n` (strings)

### Key File Test Flags
`-e -f -d -r -w -x -s`

### Special Vars
`$0 $1 $# $@ $* $? $$ $!`

---

*End of notes. Recommended companion practice: write and run every example above in a terminal, then intentionally break each one (remove quotes, spaces, flags) to observe the pitfalls firsthand — this is the fastest way to internalize shell scripting behavior for exams.*
