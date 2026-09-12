# The Complete Guide to `grep`

`grep` stands for **G**lobal **R**egular **E**xpression **P**rint. It searches text (files or input streams) for lines matching a pattern and prints them. It's one of the most useful command-line tools ever written.

---

## 1. Basic Syntax

```
grep [OPTIONS] PATTERN [FILE...]
```

- **PATTERN** — the text or regular expression to search for
- **FILE** — one or more files to search (if omitted, grep reads from standard input)
- **OPTIONS** — flags that change grep's behavior

### The simplest example

```bash
grep "hello" file.txt
```
Prints every line in `file.txt` that contains "hello".

```bash
echo "hello world" | grep "hello"
```
grep also works on piped input — this prints "hello world" because it matched.

---

## 2. Core Options (the ones you'll use constantly)

| Flag | Meaning | Example |
|------|---------|---------|
| `-i` | Case-insensitive search | `grep -i "error" log.txt` |
| `-v` | Invert match (show lines that **don't** match) | `grep -v "debug" log.txt` |
| `-n` | Show line numbers | `grep -n "TODO" script.py` |
| `-c` | Count matching lines (not the lines themselves) | `grep -c "fail" log.txt` |
| `-l` | List only filenames that contain a match | `grep -l "TODO" *.py` |
| `-L` | List only filenames that **don't** contain a match | `grep -L "TODO" *.py` |
| `-r` / `-R` | Recursive search through directories | `grep -r "API_KEY" ./src` |
| `-w` | Match whole words only | `grep -w "cat" file.txt` |
| `-x` | Match whole lines only | `grep -x "yes" file.txt` |
| `-o` | Print only the matched part, not the whole line | `grep -o "[0-9]\+" file.txt` |
| `-A n` | Show n lines **after** each match | `grep -A 3 "Error" log.txt` |
| `-B n` | Show n lines **before** each match | `grep -B 3 "Error" log.txt` |
| `-C n` | Show n lines of context **around** each match | `grep -C 2 "Error" log.txt` |
| `-E` | Extended regex (same as `egrep`) | `grep -E "cat|dog" file.txt` |
| `-F` | Fixed string, no regex interpretation (same as `fgrep`) | `grep -F "a.b" file.txt` |
| `-q` | Quiet — no output, just an exit status (useful in scripts) | `grep -q "root" /etc/passwd` |
| `-e` | Specify multiple patterns | `grep -e "cat" -e "dog" file.txt` |
| `-f` | Read patterns from a file | `grep -f patterns.txt file.txt` |
| `--color` | Highlight the matches in color | `grep --color "error" log.txt` |
| `-m n` | Stop after n matches | `grep -m 5 "error" log.txt` |
| `-H` | Always print filename (default when multiple files given) | `grep -H "x" file.txt` |
| `-s` | Suppress error messages about nonexistent/unreadable files | `grep -s "x" maybe.txt` |
| `-z` | Treat input as NUL-separated (useful with `find -print0`) | `find . -print0 \| grep -z "x"` |

### Example walkthrough

Say `log.txt` contains:
```
2024-01-01 INFO Starting service
2024-01-01 DEBUG Loading config
2024-01-01 ERROR Failed to connect
2024-01-02 INFO Retrying
2024-01-02 ERROR Connection refused
```

```bash
grep "ERROR" log.txt
```
```
2024-01-01 ERROR Failed to connect
2024-01-02 ERROR Connection refused
```

```bash
grep -i "error" log.txt          # same result, case-insensitive
grep -v "DEBUG" log.txt          # everything except DEBUG lines
grep -c "ERROR" log.txt          # → 2
grep -n "ERROR" log.txt          # shows line numbers: 3: and 5:
grep -A 1 "ERROR" log.txt        # shows the ERROR line plus 1 line after each
```

---

## 3. Regular Expressions in grep

This is where grep becomes powerful. There are three "flavors":

- **BRE** (Basic Regular Expressions) — grep's default. Special characters like `+`, `?`, `|`, `(`, `)` must be escaped with `\` to have special meaning.
- **ERE** (Extended Regular Expressions) — enabled with `-E` (or use `egrep`). No escaping needed for `+ ? | ( )`.
- **PCRE** (Perl-Compatible Regular Expressions) — enabled with `-P` (GNU grep only). Full Perl regex power (lookahead, `\d`, `\w`, etc.)

### Common regex metacharacters

| Symbol | Meaning | Example | Matches |
|--------|---------|---------|---------|
| `.` | Any single character | `c.t` | cat, cot, c9t |
| `*` | Zero or more of previous char | `ca*t` | ct, cat, caaat |
| `^` | Start of line | `^Error` | lines starting with "Error" |
| `$` | End of line | `done$` | lines ending in "done" |
| `[]` | Character class | `[abc]` | a, b, or c |
| `[^]` | Negated character class | `[^0-9]` | any non-digit |
| `[a-z]` | Range | `[a-z]` | any lowercase letter |
| `\|` (ERE) | Alternation (OR) | `cat\|dog` | cat or dog |
| `+` (ERE) | One or more | `[0-9]+` | one or more digits |
| `?` (ERE) | Zero or one | `colou?r` | color or colour |
| `{n,m}` | Between n and m repetitions | `[0-9]{2,4}` | 2 to 4 digits |
| `()` (ERE) | Grouping | `(ab)+` | ab, abab, ababab |
| `\b` (GNU) | Word boundary | `\bcat\b` | "cat" but not "category" |

### Examples

**Match lines starting with a capital letter:**
```bash
grep "^[A-Z]" file.txt
```

**Match lines ending in a digit:**
```bash
grep "[0-9]$" file.txt
```

**Match any 3-letter word starting with "c" and ending in "t":**
```bash
grep "c.t" file.txt      # cat, cot, cut, c8t, etc.
```

**Match "color" OR "colour" (BRE — need escaping):**
```bash
grep "colou\?r" file.txt
```

**Same thing in ERE (no escaping):**
```bash
grep -E "colou?r" file.txt
```

**Match "cat" or "dog":**
```bash
grep -E "cat|dog" file.txt
```

**Match valid-looking IP addresses (simplified):**
```bash
grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" file.txt
```

**Match email addresses (simplified):**
```bash
grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" file.txt
```

**Match empty lines:**
```bash
grep "^$" file.txt
```

**Match lines that are NOT empty:**
```bash
grep -v "^$" file.txt
```

**Extract only phone numbers from text (using -o):**
```bash
grep -oE "[0-9]{3}-[0-9]{3}-[0-9]{4}" file.txt
```

**Using PCRE for lookahead (GNU grep with -P):**
```bash
grep -P "foo(?=bar)" file.txt     # matches "foo" only if followed by "bar"
grep -P "\d+" file.txt            # \d works like Perl — digits
```

---

## 4. Working with Multiple Files & Directories

**Search multiple files at once:**
```bash
grep "TODO" file1.py file2.py file3.py
```
Output automatically prefixes each match with the filename.

**Search every file in a directory recursively:**
```bash
grep -r "API_KEY" ./project
```

**Recursive + show only filenames:**
```bash
grep -rl "TODO" ./project
```

**Recursive but exclude certain files/dirs:**
```bash
grep -r "TODO" ./project --exclude="*.log"
grep -r "TODO" ./project --exclude-dir="node_modules"
```

**Search only specific file types:**
```bash
grep -r "import" ./project --include="*.py"
```

**Search across many files and show line numbers + filenames:**
```bash
grep -rn "FIXME" ./src
```

---

## 5. Combining grep with Pipes (real-world usage)

This is where grep truly shines — as a filter in a pipeline.

**Find running processes matching "chrome":**
```bash
ps aux | grep chrome
```

**Count how many .txt files are in a directory:**
```bash
ls -la | grep ".txt" | wc -l
```

**Find which package versions are installed:**
```bash
pip list | grep -i requests
```

**Search command history for a previous command:**
```bash
history | grep "docker run"
```

**Filter log output live (tail + grep):**
```bash
tail -f server.log | grep "ERROR"
```

**Exclude grep itself from `ps` results (classic trick):**
```bash
ps aux | grep "[c]hrome"     # clever regex trick avoids matching the grep command itself
```

**Chained greps (AND logic — line must match both):**
```bash
cat file.txt | grep "error" | grep "database"
```

**OR logic with -E:**
```bash
grep -E "error|warning" file.txt
```

---

## 6. Useful Flag Combinations

**Case-insensitive, recursive, line numbers, whole word:**
```bash
grep -rniw "user" ./project
```

**Print 2 lines of context around each match, with color:**
```bash
grep -C 2 --color "Exception" app.log
```

**Count matches per file (not total):**
```bash
grep -c "ERROR" *.log
```

**Show only unique matched values:**
```bash
grep -o "user_[0-9]\+" file.txt | sort -u
```

**Search for a literal string containing regex special characters (use -F):**
```bash
grep -F "3.14 * r^2" formulas.txt
```
(Without `-F`, the `.` and `*` and `^` would be interpreted as regex metacharacters.)

**Find lines containing pattern A but NOT pattern B:**
```bash
grep "A" file.txt | grep -v "B"
```

---

## 7. Exit Status (useful in scripts)

`grep` returns:
- `0` if a match was found
- `1` if no match was found
- `2` if there was an error (e.g., file not found)

**Example — use in an `if` statement:**
```bash
if grep -q "ERROR" log.txt; then
    echo "Errors found!"
else
    echo "All clear."
fi
```

---

## 8. grep Variants

- `grep` — standard, BRE by default
- `egrep` — same as `grep -E` (extended regex)
- `fgrep` — same as `grep -F` (fixed strings, no regex)
- `zgrep` — grep that works directly on `.gz` compressed files
- `rgrep` — same as `grep -r`

```bash
zgrep "ERROR" logfile.gz          # search inside a gzipped file without unzipping
egrep "cat|dog" file.txt          # same as grep -E "cat|dog" file.txt
fgrep "a.b*c" file.txt            # treats a.b*c as a literal string
```

---

## 9. Practical Real-World Examples

**Find all TODO/FIXME comments in a codebase:**
```bash
grep -rn "TODO\|FIXME" ./src
```

**Find lines with trailing whitespace:**
```bash
grep -n " $" file.txt
```

**Find duplicate lines quickly (combine with sort/uniq, not grep alone, but a common pairing):**
```bash
sort file.txt | uniq -d
```

**Search for IPv4 addresses in a firewall log:**
```bash
grep -oE "([0-9]{1,3}\.){3}[0-9]{1,3}" firewall.log | sort -u
```

**Find all lines with a hex color code:**
```bash
grep -oE "#[0-9a-fA-F]{6}" style.css
```

**Check if a package is in requirements.txt:**
```bash
grep -i "^flask" requirements.txt
```

**Find all functions defined in a Python file:**
```bash
grep -n "^def " script.py
```

**Search git log messages for a keyword:**
```bash
git log --oneline | grep -i "fix"
```

**List all environment variables containing "PATH":**
```bash
env | grep PATH
```

---

## 10. Quick Cheat Sheet

```bash
grep "pattern" file                # basic search
grep -i "pattern" file             # case-insensitive
grep -v "pattern" file             # invert match
grep -c "pattern" file             # count matches
grep -n "pattern" file             # show line numbers
grep -r "pattern" dir/             # recursive
grep -l "pattern" *.txt            # filenames only
grep -w "pattern" file             # whole word
grep -o "pattern" file             # only matched text
grep -A 3 -B 3 "pattern" file      # context lines
grep -E "a|b" file                 # extended regex / OR
grep -P "\d+" file                 # Perl regex (GNU only)
grep -q "pattern" file             # quiet, for scripts
```

---

### Tips to remember
1. Always quote your pattern (`"pattern"`) to avoid the shell interpreting special characters.
2. Use `-E` (or `egrep`) whenever you need `+`, `?`, `|`, or `()` without backslash-escaping.
3. Use `-F` (or `fgrep`) when your "pattern" is really just a literal string with symbols in it.
4. Combine with `sort`, `uniq`, `wc -l`, and pipes — grep is designed to be one link in a chain of small tools.
5. `man grep` is your friend for anything not covered here — especially on macOS, whose default grep (BSD grep) has slightly different flag support than GNU grep (Linux).
