# The `sort` Command — Complete Guide

The `sort` command is a Unix/Linux utility used to arrange lines of text files or standard input in a specific order — alphabetical, numerical, reverse, random, or based on specific fields/columns.

---

## 1. Basic Syntax

```bash
sort [OPTIONS] [FILE...]
```

- If no file is given, `sort` reads from **standard input**.
- If multiple files are given, it merges and sorts them together.
- By default, output goes to **standard output** (it does not modify the file unless you use `-o`).

### Basic Example

```bash
cat names.txt
# Charlie
# Alice
# Bob

sort names.txt
# Alice
# Bob
# Charlie
```

---

## 2. Default Sorting Behavior

- Sorts **line by line**.
- Default order is **lexicographic (dictionary/ASCII) order**, not true alphabetical or numeric order.
- Uppercase letters sort before lowercase (in POSIX/C locale) because of ASCII values.
- Numbers are sorted as strings by default, so `10` comes before `2` unless you use `-n`.

```bash
echo -e "10\n2\n33\n4" | sort
# 10
# 2
# 33
# 4
```

---

## 3. Commonly Used Options

| Option | Description |
|--------|-------------|
| `-n` | Sort numerically (handles integers and floats correctly) |
| `-r` | Reverse the sort order |
| `-f` | Case-insensitive sort (folds upper/lowercase) |
| `-u` | Sort and remove duplicate lines (unique) |
| `-k` | Sort based on a specific key/column |
| `-t` | Specify a field delimiter (used with `-k`) |
| `-M` | Sort by month name (Jan, Feb, Mar, ...) |
| `-h` | Human-numeric sort (handles values like 2K, 1G, 500M) |
| `-b` | Ignore leading blanks/whitespace |
| `-c` | Check if a file is already sorted (doesn't sort, just checks) |
| `-C` | Like `-c` but silent — no output, only exit status |
| `-o FILE` | Write output to a file instead of stdout (safe to use same file as input) |
| `-R` | Random sort (shuffle lines) |
| `-s` | Stable sort — preserves order of equal elements |
| `-z` | Use NUL as line delimiter instead of newline (useful with `find -print0`) |
| `-d` | Dictionary order — ignores punctuation/special characters |
| `-i` | Ignore non-printable characters |
| `--ignore-case` | Same as `-f` |

---

## 4. Numeric Sort

```bash
echo -e "10\n2\n33\n4" | sort -n
# 2
# 4
# 10
# 33
```

For **human-readable sizes** (like `du -h` output: 1K, 2M, 3G):

```bash
echo -e "2K\n1G\n500M\n10K" | sort -h
# 10K
# 2K
# 500M
# 1G
```

---

## 5. Reverse Sort

```bash
sort -r names.txt
# Charlie
# Bob
# Alice
```

Combine with numeric:

```bash
sort -nr numbers.txt
```

---

## 6. Case-Insensitive Sort

```bash
echo -e "banana\nApple\ncherry" | sort
# Apple
# banana
# cherry

echo -e "banana\nApple\ncherry" | sort -f
# Apple
# banana
# cherry
```

---

## 7. Removing Duplicates (`-u`)

```bash
echo -e "apple\nbanana\napple\ncherry" | sort -u
# apple
# banana
# cherry
```

This is different from `sort | uniq` — `-u` does it in one pass and is slightly more efficient.

---

## 8. Sorting by Specific Column/Field (`-k`)

This is one of the most powerful and commonly used features — sorting structured data like CSV, logs, or space-separated tables.

### Example data (`data.txt`):
```
John 25 Engineer
Alice 30 Doctor
Bob 22 Artist
```

### Sort by 2nd column (age), numerically:
```bash
sort -k2 -n data.txt
# Bob 22 Artist
# John 25 Engineer
# Alice 30 Doctor
```

### Sort by 3rd column alphabetically:
```bash
sort -k3 data.txt
```

### Syntax details for `-k`:
```
-k START[,END]
```
- `START` and `END` define the field range to sort by.
- If `END` is omitted, sorting continues to the end of the line.
- Fields are 1-indexed.

Example — sort only by field 2 (not 2 onward):
```bash
sort -k2,2 -n data.txt
```

---

## 9. Custom Delimiter (`-t`)

Useful for CSV or `:`-delimited files like `/etc/passwd`.

### Example (`data.csv`):
```
John,25,Engineer
Alice,30,Doctor
Bob,22,Artist
```

Sort by age (2nd field), using comma as delimiter:

```bash
sort -t, -k2 -n data.csv
```

Sort `/etc/passwd` by username:
```bash
sort -t: -k1 /etc/passwd
```

---

## 10. Sorting Months (`-M`)

```bash
echo -e "Mar\nJan\nDec\nJul" | sort -M
# Jan
# Mar
# Jul
# Dec
```

---

## 11. Checking if a File Is Sorted (`-c` / `-C`)

```bash
sort -c names.txt
```
- If sorted → no output, exit code `0`.
- If not sorted → prints the first out-of-order line and returns non-zero exit code.

```bash
sort -C names.txt   # silent version, only exit code matters
echo $?
```

---

## 12. Writing Output to a File (`-o`)

```bash
sort names.txt -o sorted_names.txt
```

Important: You can safely use the **same file** as input and output:
```bash
sort file.txt -o file.txt
```
This is safe because `sort` reads the whole file into memory/temp storage before writing — unlike `>` redirection, which would truncate the file first and destroy your data.

**Never do this:**
```bash
sort file.txt > file.txt   # WRONG — this will empty the file!
```

---

## 13. Random Sort / Shuffle (`-R`)

```bash
echo -e "one\ntwo\nthree" | sort -R
```
Produces a random order each time (not cryptographically secure — use `shuf` for a dedicated shuffle tool).

---

## 14. Stable Sort (`-s`)

By default, when multiple lines have equal sort keys, their relative order may change. `-s` preserves original input order for ties.

```bash
sort -k1,1 -s data.txt
```

---

## 15. Sorting with NUL Delimiter (`-z`)

Used often with `find -print0` to safely handle filenames with spaces or newlines.

```bash
find . -print0 | sort -z
```

---

## 16. Ignoring Leading Blanks (`-b`)

```bash
echo -e "   banana\napple\n  cherry" | sort -b
```
Ignores leading whitespace when comparing.

---

## 17. Locale and Sorting Behavior

Sorting behavior can change based on system locale settings (`LC_COLLATE`).

```bash
LC_ALL=C sort file.txt      # strict byte/ASCII order
LC_ALL=en_US.UTF-8 sort file.txt   # locale-aware, dictionary style
```

- `LC_ALL=C` gives predictable, script-safe results (recommended in shell scripts).
- Locale-aware sort might treat accented characters or cases differently.

---

## 18. Combining Multiple Options

You can chain multiple flags together:

```bash
sort -t, -k2,2n -k3,3r file.csv
```
This means:
- Use comma as delimiter
- Sort by field 2 numerically
- Then by field 3 in reverse (as a tiebreaker)

Note: appending `n` or `r` directly after the field range (e.g., `-k2,2n`) applies that sort type **only to that key**, which is useful for mixed sort types across columns.

---

## 19. Sorting in Reverse by Specific Column

```bash
sort -k2,2nr data.txt
```
Sorts by column 2, numeric, descending.

---

## 20. Practical Real-World Examples

### a) Sort a list of files by size (with `du`):
```bash
du -h * | sort -h
```

### b) Sort process list by memory usage:
```bash
ps aux | sort -k4 -nr | head
```

### c) Sort `/etc/passwd` by UID:
```bash
sort -t: -k3 -n /etc/passwd
```

### d) Sort log file by timestamp (if timestamp is the first field):
```bash
sort -k1,1 access.log
```

### e) Remove duplicate lines and sort:
```bash
sort -u file.txt
```

### f) Sort a CSV by 3rd column (descending, numeric):
```bash
sort -t, -k3,3nr data.csv
```

### g) Merge two already-sorted files into one sorted output:
```bash
sort -m file1.txt file2.txt
```
(`-m` assumes inputs are already sorted and merges efficiently without re-sorting everything.)

---

## 21. Exit Status

| Code | Meaning |
|------|---------|
| 0 | Success (and file is sorted, if `-c`/`-C` used) |
| 1 | Input not sorted (with `-c`/`-C`) or minor error |
| 2 | Serious error (e.g., invalid option, file not found) |

---

## 22. Common Pitfalls

1. **Numbers sorted as text**: Forgetting `-n` causes `10` to appear before `2`.
2. **Case sensitivity**: Uppercase letters sort before lowercase by default — use `-f` if unwanted.
3. **Overwriting files with `>`**: Always use `-o` instead of `sort file > file`.
4. **Locale differences**: Scripts may behave differently across systems — use `LC_ALL=C` for consistency.
5. **Field counting confusion**: Fields with `-k` are separated by whitespace by default, unless `-t` is specified.
6. **Trailing/leading spaces** can affect sort order unless `-b` is used.

---

## 23. Quick Reference Cheat Sheet

```bash
sort file.txt                  # basic alphabetical sort
sort -n file.txt                # numeric sort
sort -r file.txt                # reverse sort
sort -nr file.txt               # numeric + reverse
sort -f file.txt                # case-insensitive
sort -u file.txt                # unique + sorted
sort -k2 file.txt               # sort by 2nd column
sort -k2,2n file.txt            # sort by 2nd column, numeric only
sort -t, -k3 file.csv           # sort CSV by 3rd column
sort -M file.txt                # sort by month name
sort -h file.txt                # human-readable numbers (K, M, G)
sort -c file.txt                # check if sorted
sort -o out.txt file.txt        # write to another file (safe overwrite too)
sort -R file.txt                # random shuffle
sort -m a.txt b.txt             # merge sorted files
```

---

## 24. Related Commands

- `uniq` — remove/report duplicate lines (usually used after `sort`)
- `shuf` — proper random shuffling of lines
- `comm` — compare two sorted files line by line
- `join` — join lines of two sorted files based on a common field
- `awk` / `cut` — often combined with `sort` for field extraction before sorting

---

## 25. Summary

`sort` is a simple but extremely powerful text-processing tool, especially useful in shell scripting, log analysis, and data processing pipelines. Mastering flags like `-k`, `-t`, `-n`, `-u`, and `-o` allows you to handle almost any column-based or structured sorting task directly from the command line without needing external scripting languages.
