# 📘 Complete Revision Guide: Bash Scripting

> A beginner-friendly, detailed guide covering everything about Bash Scripting — with examples, explanations, practice problems, and solutions.

---

## Table of Contents

1. [What is Bash Scripting?](#1-what-is-bash-scripting)
2. [Writing and Running Your First Script](#2-writing-and-running-your-first-script)
3. [The Shebang Line](#3-the-shebang-line)
4. [Comments](#4-comments)
5. [Variables](#5-variables)
6. [Taking User Input](#6-taking-user-input)
7. [Command Line Arguments](#7-command-line-arguments)
8. [Special Variables](#8-special-variables)
9. [Quoting Rules (very important!)](#9-quoting-rules-very-important)
10. [Arithmetic Operations](#10-arithmetic-operations)
11. [String Operations](#11-string-operations)
12. [Arrays](#12-arrays)
13. [Conditional Statements (if / else / elif)](#13-conditional-statements-if--else--elif)
14. [Test Operators (comparison conditions)](#14-test-operators-comparison-conditions)
15. [case Statement](#15-case-statement)
16. [Loops](#16-loops)
17. [Functions](#17-functions)
18. [Exit Status and Exit Code](#18-exit-status-and-exit-code)
19. [File Test Operators](#19-file-test-operators)
20. [Reading Files Line by Line](#20-reading-files-line-by-line)
21. [Here Document (`<<`) and Here String (`<<<`)](#21-here-document--and-here-string-)
22. [Command Substitution](#22-command-substitution)
23. [Redirection and Pipes (quick recap)](#23-redirection-and-pipes-quick-recap)
24. [Debugging a Script](#24-debugging-a-script)
25. [Practice Problems with Solutions](#25-practice-problems-with-solutions)
26. [Common Mistakes Beginners Make](#26-common-mistakes-beginners-make)
27. [Quick Cheat Sheet](#27-quick-cheat-sheet)
28. [Summary](#28-summary)

---

## 1. What is Bash Scripting?

**Bash** (**B**ourne **A**gain **SH**ell) is the default command-line shell/interpreter on most Linux systems.

A **Bash script** is simply a **text file containing a series of Linux commands**, written in order, that get executed automatically — just like you'd type them one by one in the terminal, but saved and run together.

### Why do we write Bash scripts?
- To **automate repetitive tasks** (backups, file renaming, log cleaning).
- To **combine multiple commands** into a single reusable program.
- To write **system administration** tools (server setup, monitoring, deployment).
- To make decisions and loop through data using logic (if/else, loops).
- To glue together tools like `grep`, `sed`, `awk` into a bigger workflow.

> 🧠 Think of a bash script as a **recipe** — a list of steps the computer follows exactly, in order, every single time.

---

## 2. Writing and Running Your First Script

### Step 1: Create a file
```bash
nano myscript.sh
```

### Step 2: Write your script
```bash
#!/bin/bash
echo "Hello, World!"
```

### Step 3: Give it execute permission
```bash
chmod +x myscript.sh
```

### Step 4: Run it
```bash
./myscript.sh
```
**Output:**
```
Hello, World!
```

### Alternative ways to run a script (without chmod)
```bash
bash myscript.sh
sh myscript.sh
```
👉 These directly tell the shell to interpret the file, so execute permission isn't strictly required.

---

## 3. The Shebang Line

```bash
#!/bin/bash
```

This **must be the very first line** of your script. It tells the system **which interpreter** should run this script.

- `#!/bin/bash` → use Bash shell
- `#!/bin/sh` → use the basic POSIX shell
- `#!/usr/bin/env bash` → find bash automatically wherever it's installed (more portable across systems)

> ⚠️ Without a shebang, the script might still run (using your current shell), but it's a **best practice** to always include it, so behaviour stays predictable everywhere.

---

## 4. Comments

Comments are notes for humans — Bash ignores them completely.

```bash
# This is a single-line comment
echo "This line runs"   # comment can also be added after a command
```

There's no official "multi-line comment" symbol in bash, but this trick is commonly used:
```bash
: '
This is a
multi-line comment block
'
```

---

## 5. Variables

### 5.1 Declaring a Variable
```bash
name="Ravi"
age=25
```
> ⚠️ **No spaces** allowed around `=`. `name = "Ravi"` will **cause an error** — it must be `name="Ravi"`.

### 5.2 Using (Accessing) a Variable
Use `$` before the variable name:
```bash
echo $name
echo "My name is $name and I am $age years old"
```
**Output:**
```
My name is Ravi and I am 25 years old
```

### 5.3 Curly Braces `${}`
Used to clearly separate variable name from surrounding text.
```bash
fruit="apple"
echo "I have a ${fruit}pie"    # without {} bash would look for a variable called "fruitpie" (wrong!)
```

### 5.4 Read-only Variables
```bash
readonly PI=3.14
```
👉 Once set, this value **cannot be changed** later in the script.

### 5.5 Unset a Variable
```bash
unset name
```

### 5.6 Environment Variables (already exist in your system)
```bash
echo $HOME     # your home directory
echo $USER     # current username
echo $PATH     # search path for commands
```

---

## 6. Taking User Input

The `read` command lets your script accept input **from the user**.

```bash
#!/bin/bash
echo "Enter your name:"
read name
echo "Hello, $name!"
```

### Reading with a prompt in one line (`-p`)
```bash
read -p "Enter your age: " age
echo "You are $age years old"
```

### Reading multiple values at once
```bash
read -p "Enter first and last name: " first last
echo "First: $first, Last: $last"
```

### Silent input (for passwords — hides typed characters)
```bash
read -sp "Enter password: " pass
echo ""
echo "Password saved (hidden)."
```

---

## 7. Command Line Arguments

You can pass values to your script **when you run it**, instead of asking with `read`.

```bash
#!/bin/bash
echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "Total arguments: $#"
```

Run it:
```bash
./myscript.sh Ravi 25
```
**Output:**
```
Script name: ./myscript.sh
First argument: Ravi
Second argument: 25
Total arguments: 2
```

---

## 8. Special Variables

| Variable | Meaning |
|----------|---------|
| `$0` | Name of the script itself |
| `$1, $2, ...` | Positional arguments (1st, 2nd, ... passed to script) |
| `$#` | Number of arguments passed |
| `$@` | All arguments, as **separate** words |
| `$*` | All arguments, as **one single** string |
| `$?` | Exit status of the last executed command (0 = success) |
| `$$` | Process ID (PID) of the current script |
| `$!` | PID of the last background process |

### Example: `$@` vs `$*`
```bash
#!/bin/bash
for arg in "$@"; do
  echo "Arg: $arg"
done
```
👉 `"$@"` treats each argument **separately**, even if it has spaces — this is almost always what you want. `"$*"` would combine everything into one single string.

---

## 9. Quoting Rules (very important!)

Bash has 3 types of quoting, and beginners often mix them up.

| Quote Type | Behavior |
|------------|----------|
| `'single quotes'` | Everything inside is **literal** — no variable expansion |
| `"double quotes"` | Variables and command substitution **are** expanded |
| `` `backticks` `` or `$()` | Used for command substitution (explained in Section 22) |

### Example
```bash
name="Ravi"
echo 'Hello $name'     # Output: Hello $name   (literal, no expansion)
echo "Hello $name"     # Output: Hello Ravi    (expanded)
```

> 💡 Rule of thumb for beginners: **always use double quotes** around variables (`"$name"`) unless you specifically want literal text — this also prevents bugs caused by spaces in values.

---

## 10. Arithmetic Operations

Bash doesn't do math directly with `+`, `-` etc. like other languages — you need special syntax.

### 10.1 Using `$(( ))` (most common, recommended)
```bash
a=10
b=3
echo $((a + b))    # 13
echo $((a - b))    # 7
echo $((a * b))    # 30
echo $((a / b))    # 3   (integer division, decimals are dropped!)
echo $((a % b))    # 1   (remainder)
echo $((a ** b))   # 1000  (power)
```

### 10.2 Using `expr` (older method)
```bash
expr 10 + 3
```
> ⚠️ `expr` needs **spaces** around operators, and `*` must be escaped (`\*`) because it's a wildcard in bash.

### 10.3 Using `bc` for decimal/floating-point math
Bash's `$(( ))` only handles **integers**. For decimals, use `bc`:
```bash
echo "10 / 3" | bc -l
```
**Output:** `3.33333333333333333333`

### 10.4 Increment/Decrement
```bash
count=5
((count++))     # count becomes 6
((count--))     # count becomes 5 again
```

---

## 11. String Operations

### 11.1 String Concatenation
```bash
first="Ravi"
last="Kumar"
full="$first $last"
echo "$full"     # Ravi Kumar
```

### 11.2 String Length
```bash
str="Hello"
echo ${#str}      # 5
```

### 11.3 Substring Extraction
```bash
str="Hello World"
echo ${str:0:5}    # Hello   (start at index 0, length 5)
echo ${str:6}       # World   (from index 6 to end)
```

### 11.4 Replace Text in a String
```bash
str="I like cats"
echo ${str/cats/dogs}    # I like dogs   (replaces first match)
```

### 11.5 Convert Case (Bash 4+)
```bash
str="hello"
echo ${str^^}    # HELLO   (uppercase)
echo ${str,,}    # hello   (lowercase)
```

### 11.6 Compare Strings
```bash
str1="apple"
str2="banana"
if [ "$str1" == "$str2" ]; then
  echo "Same"
else
  echo "Different"
fi
```

---

## 12. Arrays

### 12.1 Declaring an Array
```bash
fruits=("apple" "banana" "mango")
```

### 12.2 Accessing Elements
```bash
echo ${fruits[0]}      # apple  (indexing starts at 0)
echo ${fruits[@]}      # apple banana mango  (all elements)
echo ${#fruits[@]}      # 3  (total number of elements)
```

### 12.3 Adding an Element
```bash
fruits+=("orange")
```

### 12.4 Looping Through an Array
```bash
for fruit in "${fruits[@]}"; do
  echo "$fruit"
done
```

### 12.5 Associative Arrays (like dictionaries, Bash 4+)
```bash
declare -A capital
capital["India"]="New Delhi"
capital["France"]="Paris"

echo "${capital[India]}"    # New Delhi

for country in "${!capital[@]}"; do
  echo "$country -> ${capital[$country]}"
done
```

---

## 13. Conditional Statements (if / else / elif)

### 13.1 Basic if
```bash
#!/bin/bash
age=20
if [ $age -ge 18 ]; then
  echo "You are an adult"
fi
```

### 13.2 if / else
```bash
if [ $age -ge 18 ]; then
  echo "Adult"
else
  echo "Minor"
fi
```

### 13.3 if / elif / else
```bash
marks=75
if [ $marks -ge 90 ]; then
  echo "Grade A"
elif [ $marks -ge 75 ]; then
  echo "Grade B"
elif [ $marks -ge 50 ]; then
  echo "Grade C"
else
  echo "Fail"
fi
```

> ⚠️ Note the **spaces** — `[ $age -ge 18 ]` needs a space after `[` and before `]`. Missing these spaces is one of the most common beginner errors.

---

## 14. Test Operators (comparison conditions)

### 14.1 Numeric Comparison
| Operator | Meaning |
|----------|---------|
| `-eq` | equal to |
| `-ne` | not equal to |
| `-gt` | greater than |
| `-lt` | less than |
| `-ge` | greater than or equal to |
| `-le` | less than or equal to |

```bash
if [ $a -eq $b ]; then echo "Equal"; fi
```

### 14.2 String Comparison
| Operator | Meaning |
|----------|---------|
| `==` | equal |
| `!=` | not equal |
| `-z` | string is empty |
| `-n` | string is NOT empty |

```bash
if [ -z "$name" ]; then echo "Name is empty"; fi
```

### 14.3 Logical Operators
| Operator | Meaning |
|----------|---------|
| `&&` | AND |
| `\|\|` | OR |
| `!` | NOT |

```bash
if [ $age -ge 18 ] && [ $age -le 60 ]; then
  echo "Working age"
fi
```

### 14.4 Modern Alternative: `[[ ]]` (recommended, more powerful)
```bash
if [[ $age -ge 18 && $age -le 60 ]]; then
  echo "Working age"
fi
```
👉 `[[ ]]` is a Bash-specific improvement over `[ ]` — supports regex matching (`=~`), doesn't require quoting variables as strictly, and avoids some tricky edge cases.

---

## 15. case Statement

Used when you have **many possible values** to check — cleaner than a long `if/elif` chain.

```bash
#!/bin/bash
read -p "Enter a fruit name: " fruit

case $fruit in
  apple)
    echo "It's red or green."
    ;;
  banana)
    echo "It's yellow."
    ;;
  mango)
    echo "It's the king of fruits!"
    ;;
  *)
    echo "Unknown fruit."
    ;;
esac
```
👉 `*)` acts as the **default case** (like `else`), and `;;` ends each case block.

---

## 16. Loops

### 16.1 for loop (over a list)
```bash
for i in 1 2 3 4 5; do
  echo "Number: $i"
done
```

### 16.2 for loop (numeric range, C-style)
```bash
for ((i=1; i<=5; i++)); do
  echo "i = $i"
done
```

### 16.3 for loop over files
```bash
for file in *.txt; do
  echo "Found: $file"
done
```

### 16.4 while loop
```bash
count=1
while [ $count -le 5 ]; do
  echo "Count: $count"
  ((count++))
done
```

### 16.5 until loop (opposite of while — runs UNTIL condition becomes true)
```bash
count=1
until [ $count -gt 5 ]; do
  echo "Count: $count"
  ((count++))
done
```

### 16.6 break and continue
```bash
for i in 1 2 3 4 5; do
  if [ $i -eq 3 ]; then
    continue    # skips number 3, continues loop
  fi
  if [ $i -eq 5 ]; then
    break       # stops loop entirely when i=5
  fi
  echo $i
done
```

---

## 17. Functions

Functions let you **group commands together** and reuse them, just like in any programming language.

### 17.1 Defining and Calling a Function
```bash
#!/bin/bash
greet() {
  echo "Hello, $1!"
}

greet "Ravi"      # Output: Hello, Ravi!
greet "Sonia"     # Output: Hello, Sonia!
```
👉 Inside a function, `$1`, `$2` etc. refer to the arguments **passed to the function**, not the script's own arguments.

### 17.2 Returning a Value
Functions in bash don't "return" values like other languages — they return an **exit status** (0-255) via `return`, or you print the result and capture it.

```bash
add() {
  local result=$(( $1 + $2 ))
  echo $result
}

sum=$(add 5 10)
echo "Sum is: $sum"     # Sum is: 15
```
👉 `local` keeps the variable scoped inside the function only (best practice, avoids clashing with variables outside).

---

## 18. Exit Status and Exit Code

Every command in Linux returns an **exit status** after running:
- `0` = success
- any non-zero number (1–255) = failure/error

```bash
ls /home
echo $?     # prints 0 if ls succeeded, non-zero if it failed
```

### Setting a custom exit code in your own script
```bash
#!/bin/bash
echo "Doing some task..."
exit 0     # success
```
```bash
#!/bin/bash
echo "Something went wrong"
exit 1     # failure
```
👉 Useful when your script is called by another script/program that needs to know if it succeeded or failed.

---

## 19. File Test Operators

Used to check properties of files/directories — very commonly used in real scripts.

| Operator | Meaning |
|----------|---------|
| `-e file` | file exists |
| `-f file` | file exists and is a regular file |
| `-d file` | file exists and is a directory |
| `-r file` | file is readable |
| `-w file` | file is writable |
| `-x file` | file is executable |
| `-s file` | file exists and is NOT empty |

### Example
```bash
#!/bin/bash
file="students.txt"

if [ -e "$file" ]; then
  echo "$file exists"
else
  echo "$file does not exist"
fi
```

```bash
if [ -d "/home/user" ]; then
  echo "It's a directory"
fi
```

---

## 20. Reading Files Line by Line

A very common real-world task — processing each line of a file inside a script.

```bash
#!/bin/bash
while IFS= read -r line; do
  echo "Line: $line"
done < "students.txt"
```

**Breakdown:**
- `IFS=` → prevents leading/trailing whitespace from being trimmed
- `read -r` → `-r` prevents backslashes (`\`) from being interpreted specially
- `< "students.txt"` → feeds the file into the while loop line by line

---

## 21. Here Document (`<<`) and Here String (`<<<`)

### 21.1 Here Document — feed multiple lines of input to a command
```bash
cat << EOF
This is line 1
This is line 2
Name: $name
EOF
```
👉 Everything between `<< EOF` and `EOF` is treated as input text (and variables **are** expanded, since it's like double-quoting).

### 21.2 Here String — feed a single line/string to a command
```bash
grep "Ravi" <<< "$data"
```
👉 Sends the value of `$data` directly as input to `grep`, without needing a separate file.

---

## 22. Command Substitution

Lets you **capture the output of a command** and store/use it as a value.

### Modern syntax (recommended)
```bash
today=$(date)
echo "Today is: $today"
```

### Old syntax (still works, but harder to read/nest)
```bash
today=`date`
```

### Practical Example
```bash
count=$(ls | wc -l)
echo "Number of files: $count"
```

---

## 23. Redirection and Pipes (quick recap)

Bash scripts commonly use these to control input/output:

| Symbol | Meaning |
|--------|---------|
| `>` | redirect output to a file (overwrite) |
| `>>` | redirect output to a file (append) |
| `<` | take input from a file |
| `\|` | pipe output of one command into another |
| `2>` | redirect error messages |
| `&>` | redirect both output and errors |

```bash
echo "Log entry" >> logfile.txt      # append to file
grep "error" logfile.txt | wc -l     # count lines with "error"
command 2> errors.txt                # save error messages separately
```

---

## 24. Debugging a Script

### 24.1 Run script with debug mode (`-x`)
```bash
bash -x myscript.sh
```
👉 Prints every command **before** it runs, along with variable values — great for finding bugs.

### 24.2 Add debug mode inside the script itself
```bash
#!/bin/bash
set -x     # turn debugging ON from here
# ... commands ...
set +x     # turn debugging OFF
```

### 24.3 Stop script immediately on any error
```bash
set -e
```
👉 If any command fails (non-zero exit status), the entire script stops immediately — very useful to prevent a script from continuing after something breaks.

---

## 25. Practice Problems with Solutions

> 💪 Try solving these yourself first before checking the solution!

---

**Problem 1:** Write a script that asks for the user's name and greets them.

**Solution:**
```bash
#!/bin/bash
read -p "Enter your name: " name
echo "Hello, $name! Welcome."
```

---

**Problem 2:** Write a script that takes two numbers as command-line arguments and prints their sum.

**Solution:**
```bash
#!/bin/bash
a=$1
b=$2
echo "Sum: $((a + b))"
```
Run: `./script.sh 10 20` → Output: `Sum: 30`

---

**Problem 3:** Write a script to check if a number is even or odd.

**Solution:**
```bash
#!/bin/bash
read -p "Enter a number: " num
if [ $((num % 2)) -eq 0 ]; then
  echo "Even"
else
  echo "Odd"
fi
```

---

**Problem 4:** Write a script to check if a file exists, and if not, create it.

**Solution:**
```bash
#!/bin/bash
file="data.txt"
if [ -e "$file" ]; then
  echo "File already exists."
else
  touch "$file"
  echo "File created."
fi
```

---

**Problem 5:** Write a script to print numbers from 1 to 10 using a `for` loop.

**Solution:**
```bash
#!/bin/bash
for i in {1..10}; do
  echo $i
done
```

---

**Problem 6:** Write a script to print the multiplication table of a number entered by the user.

**Solution:**
```bash
#!/bin/bash
read -p "Enter a number: " num
for ((i=1; i<=10; i++)); do
  echo "$num x $i = $((num * i))"
done
```

---

**Problem 7:** Write a function that checks whether a given number is prime.

**Solution:**
```bash
#!/bin/bash
is_prime() {
  num=$1
  if [ $num -lt 2 ]; then
    echo "Not prime"
    return
  fi
  for ((i=2; i*i<=num; i++)); do
    if [ $((num % i)) -eq 0 ]; then
      echo "Not prime"
      return
    fi
  done
  echo "Prime"
}

read -p "Enter a number: " n
is_prime $n
```

---

**Problem 8:** Write a script to count how many `.txt` files exist in the current directory.

**Solution:**
```bash
#!/bin/bash
count=$(ls *.txt 2>/dev/null | wc -l)
echo "Total .txt files: $count"
```

---

**Problem 9:** Write a script that reads a file line by line and prints only lines containing the word "error".

**Solution:**
```bash
#!/bin/bash
while IFS= read -r line; do
  if [[ $line == *"error"* ]]; then
    echo "$line"
  fi
done < "logfile.txt"
```

---

**Problem 10:** Write a script to reverse a string entered by the user.

**Solution:**
```bash
#!/bin/bash
read -p "Enter a string: " str
echo "$str" | rev
```

---

**Problem 11:** Write a script using a `case` statement that tells the day type (Weekday/Weekend) based on user input (Mon–Sun).

**Solution:**
```bash
#!/bin/bash
read -p "Enter day (Mon/Tue/.../Sun): " day
case $day in
  Sat|Sun)
    echo "Weekend"
    ;;
  Mon|Tue|Wed|Thu|Fri)
    echo "Weekday"
    ;;
  *)
    echo "Invalid day"
    ;;
esac
```

---

**Problem 12:** Write a script to find the largest of three numbers passed as arguments.

**Solution:**
```bash
#!/bin/bash
a=$1
b=$2
c=$3

if [ $a -ge $b ] && [ $a -ge $c ]; then
  echo "Largest: $a"
elif [ $b -ge $a ] && [ $b -ge $c ]; then
  echo "Largest: $b"
else
  echo "Largest: $c"
fi
```

---

**Problem 13:** Write a script to create a backup copy of a file with today's date in the filename.

**Solution:**
```bash
#!/bin/bash
file="students.txt"
date_today=$(date +%Y-%m-%d)
cp "$file" "${file%.txt}_$date_today.txt"
echo "Backup created: ${file%.txt}_$date_today.txt"
```

---

**Problem 14:** Write a script using an array to store 5 city names and print them all with numbering.

**Solution:**
```bash
#!/bin/bash
cities=("Delhi" "Mumbai" "Pune" "Chennai" "Kolkata")
for i in "${!cities[@]}"; do
  echo "$((i+1)). ${cities[$i]}"
done
```

---

**Problem 15:** Write a script that keeps asking the user to guess a number (fixed as 7) until they guess correctly.

**Solution:**
```bash
#!/bin/bash
secret=7
guess=0
while [ $guess -ne $secret ]; do
  read -p "Guess the number (1-10): " guess
  if [ $guess -ne $secret ]; then
    echo "Wrong! Try again."
  fi
done
echo "Correct! You guessed it."
```

---

## 26. Common Mistakes Beginners Make

1. ❌ Missing spaces inside `[ ]`:
   ```bash
   if [$age -ge 18]     # ❌ WRONG - no spaces
   if [ $age -ge 18 ]   # ✅ CORRECT
   ```

2. ❌ Spaces around `=` when assigning variables:
   ```bash
   name = "Ravi"    # ❌ WRONG
   name="Ravi"      # ✅ CORRECT
   ```

3. ❌ Forgetting to quote variables — causes errors when values contain spaces:
   ```bash
   if [ $name == "Ravi Kumar" ]     # ❌ risky if $name has spaces
   if [ "$name" == "Ravi Kumar" ]   # ✅ safer
   ```

4. ❌ Forgetting to make the script executable before running with `./`:
   ```bash
   ./script.sh     # ❌ Permission denied (if chmod +x wasn't run)
   chmod +x script.sh
   ./script.sh     # ✅ works now
   ```

5. ❌ Using `==` for numeric comparison instead of `-eq`:
   ```bash
   if [ $a == $b ]     # works but not ideal for numbers
   if [ $a -eq $b ]     # ✅ correct/clear for numbers
   ```
   (`==` is meant for strings; `-eq` etc. are for numbers — mixing them up causes confusion, though bash is somewhat forgiving here.)

6. ❌ Forgetting `$` when using a variable (only needed when **reading** its value, not when **assigning**):
   ```bash
   echo name    # ❌ prints the literal word "name"
   echo $name   # ✅ prints the value stored in the variable
   ```

7. ❌ Forgetting `fi`, `done`, `esac` to close `if`, loops, and `case` blocks — bash will throw a syntax error if these are missing.

8. ❌ Using single quotes when you actually need variable expansion:
   ```bash
   echo 'Value is $x'    # ❌ prints literally: Value is $x
   echo "Value is $x"    # ✅ prints the actual value
   ```

---

## 27. Quick Cheat Sheet

### Variables
```bash
name="Ravi"          # assign (no spaces around =)
echo $name            # use
echo "${name}"         # use safely with braces
readonly PI=3.14       # read-only
unset name              # remove
```

### Special Variables
| Var | Meaning |
|-----|---------|
| `$0` | script name |
| `$1...$9` | positional arguments |
| `$#` | number of arguments |
| `$@` | all arguments (separate) |
| `$*` | all arguments (as one string) |
| `$?` | exit status of last command |
| `$$` | current process ID |

### Arithmetic
```bash
echo $((a + b))
((count++))
```

### Conditionals
```bash
if [ condition ]; then
  ...
elif [ condition ]; then
  ...
else
  ...
fi
```

### Test Operators
| Numeric | String | File |
|---------|--------|------|
| `-eq -ne -gt -lt -ge -le` | `== != -z -n` | `-e -f -d -r -w -x -s` |

### Loops
```bash
for i in 1 2 3; do ...; done
for ((i=0;i<5;i++)); do ...; done
while [ cond ]; do ...; done
until [ cond ]; do ...; done
```

### Functions
```bash
myfunc() {
  local x=$1
  echo "$x"
}
result=$(myfunc "value")
```

### case
```bash
case $var in
  pattern1) commands ;;
  pattern2) commands ;;
  *) default commands ;;
esac
```

### File Handling
```bash
while IFS= read -r line; do echo "$line"; done < file.txt
```

### Debugging
```bash
bash -x script.sh    # debug mode
set -e                 # stop on error
```

---

## 28. Summary

- A **Bash script** is a text file of Linux commands executed in sequence, starting with a **shebang** (`#!/bin/bash`).
- **Variables** are created without `$` (`name="Ravi"`) but accessed **with** `$` (`echo $name`) — no spaces around `=`.
- **User input** is captured with `read`; **command-line arguments** are accessed via `$1, $2, ... $#, $@`.
- Always **double-quote variables** (`"$var"`) to avoid bugs from spaces/empty values.
- **Arithmetic** uses `$(( ))`; string/decimal math needs `bc`.
- **Conditionals** (`if/elif/else`) use test operators — numeric (`-eq, -gt...`), string (`==, -z...`), and file (`-e, -f, -d...`).
- **Loops** (`for`, `while`, `until`) automate repetition; `break`/`continue` control flow inside them.
- **Functions** group reusable code; use `local` for variables scoped only inside the function.
- Every command returns an **exit status** (`$?`) — `0` means success, non-zero means failure; scripts can set their own with `exit`.
- **Arrays** (indexed and associative) let you store and loop through multiple related values.
- Real-world scripts commonly combine: reading files line-by-line, command substitution (`$(...)`), redirection (`>`, `>>`, `<`, `|`), and file test operators.
- Use `bash -x script.sh` or `set -x` to **debug**, and `set -e` to stop execution immediately on errors.

---

### 🎯 Final Tip for a Beginner
Start by writing very small scripts — just `echo` statements and variables. Once comfortable, add `if` conditions, then loops, then functions. Bash scripting is best learned by **writing and breaking things** — write a script, run it, see what error you get, fix it, and try again. Every error message is a small lesson!

---
*Made for revision purposes — practice each example on your own terminal for best learning.*
