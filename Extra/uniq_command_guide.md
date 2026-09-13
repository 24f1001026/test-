# The `uniq` Command — Complete Guide

The `uniq` command is used to filter out **adjacent (consecutive) duplicate lines** from text input. It's commonly paired with `sort`, since `uniq` only detects duplicates that are next to each other — not duplicates scattered throughout a file.

---

## 1. Basic Syntax

```bash
uniq [OPTIONS] [INPUT [OUTPUT]]
```

- If no `INPUT` file is given, `uniq` reads from **standard input**.
- If `OUTPUT` is given, results are written there instead of stdout.
- By default, `uniq` does **not** modify the input file.

---

## 2. Why You Almost Always Sort First

`uniq` only removes duplicates that are **adjacent** to each other.

```bash
echo -e "apple\napple\nbanana\napple" | uniq
# apple
# banana
# apple        <- not removed, not adjacent to the first "apple"
```

Correct approach — sort first so duplicates become adjacent:

```bash
sort file.txt | uniq
```

---

## 3. Basic Example

```bash
cat fruits.txt
# apple
# apple
# banana
# banana
# banana
# cherry

uniq fruits.txt
# apple
# banana
# cherry
```

---

## 4. Commonly Used Options

| Option | Description |
|--------|-------------|
| `-c` | Prefix each output line with the number of times it occurred |
| `-d` | Only print lines that are duplicated (one copy of each) |
| `-D` | Print **all** duplicate lines (not collapsed to one) |
| `-u` | Only print lines that are **unique** (appear exactly once) |
| `-i` | Case-insensitive comparison |
| `-f N` | Ignore (skip) the first N fields when comparing lines |
| `-s N` | Skip the first N characters when comparing lines |
| `-w N` | Compare only the first N characters of each line |
| `-z` | Use NUL character as the line delimiter instead of newline |
| `--group[=METHOD]` | Group items with a visual separator (`prepend`, `append`, `both`, `separate`) |

---

## 5. Counting Occurrences (`-c`)

```bash
sort fruits.txt | uniq -c
#       2 apple
#       3 banana
#       1 cherry
```

Useful combined with sorting the counts themselves:

```bash
sort fruits.txt | uniq -c | sort -nr
#       3 banana
#       2 apple
#       1 cherry
```

This pattern (`sort | uniq -c | sort -nr`) is extremely common for **frequency analysis** — e.g., finding the most common IPs in a log file, most repeated words, etc.

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

---

## 6. Show Only Duplicated Lines (`-d`)

```bash
sort fruits.txt | uniq -d
# apple
# banana
```
Prints only lines that occurred more than once (one copy each).

### Show ALL copies of duplicated lines (`-D`)

```bash
sort fruits.txt | uniq -D
# apple
# apple
# banana
# banana
# banana
```
Useful for extracting every instance of a duplicate rather than a single representative line.

---

## 7. Show Only Unique Lines (`-u`)

```bash
sort fruits.txt | uniq -u
# cherry
```
Prints only lines that appear **exactly once** in the input.

---

## 8. Case-Insensitive Comparison (`-i`)

```bash
echo -e "Apple\napple\nBanana\nbanana" | sort | uniq -i
# Apple
# Banana
```
Without `-i`, `Apple` and `apple` would be treated as different lines.

---

## 9. Ignoring Fields (`-f`)

Skips the first N whitespace-separated fields when comparing lines (but still prints the full line).

### Example (`data.txt`):
```
1 apple
2 apple
3 banana
```

```bash
uniq -f1 data.txt
# 1 apple
# 3 banana
```
Here, field 1 (the number) is ignored during comparison, so `1 apple` and `2 apple` are considered duplicates (only the first is kept).

---

## 10. Skipping Characters (`-s`)

Skips the first N **characters** (not fields) when comparing.

```bash
echo -e "xxapple\nyyapple\nzzbanana" | uniq -s2
# xxapple
# zzbanana
```
The first 2 characters of each line are ignored during comparison, so `xxapple` and `yyapple` are seen as duplicates.

---

## 11. Comparing Only First N Characters (`-w`)

```bash
echo -e "apple123\napple456\nbanana789" | uniq -w5
# apple123
# banana789
```
Only the first 5 characters are compared. Since `apple123` and `apple456` share the first 5 characters (`apple`), the second is treated as a duplicate.

---

## 12. NUL-Delimited Input (`-z`)

Used for safely handling filenames with spaces or special characters, often paired with `find -print0` and `sort -z`.

```bash
find . -print0 | sort -z | uniq -z
```

---

## 13. Practical Real-World Examples

### a) Find the most frequently occurring lines in a file:
```bash
sort file.txt | uniq -c | sort -nr | head -10
```

### b) Find lines that appear more than once:
```bash
sort file.txt | uniq -d
```

### c) Find lines that appear only once (e.g., unique log entries):
```bash
sort file.txt | uniq -u
```

### d) Count unique visitor IPs in a log file:
```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr
```

### e) Remove duplicate words in a word list (case-insensitive):
```bash
sort wordlist.txt | uniq -i
```

### f) Check for duplicate entries in a CSV column (2nd column):
```bash
cut -d, -f2 data.csv | sort | uniq -d
```

---

## 14. `uniq` vs `sort -u`

Both can remove duplicates, but they behave differently:

| Feature | `sort -u` | `sort \| uniq` |
|---------|-----------|----------------|
| Removes duplicates | Yes (all, not just adjacent) | Yes (after sorting, so effectively all) |
| Preserves original order | No — always sorted | No — always sorted (since sort runs first) |
| Can count occurrences | No | Yes (`uniq -c`) |
| Can show only duplicates | No | Yes (`uniq -d`) |
| Extra flexibility (field/char skipping) | No | Yes (`-f`, `-s`, `-w`) |

**Rule of thumb:** use `sort -u` for a quick unique+sorted list; use `sort | uniq` (or its variants) when you need counts, duplicate-only output, or field-level comparison logic.

---

## 15. Common Pitfalls

1. **Forgetting to sort first** — `uniq` only catches adjacent duplicates, so unsorted input gives misleading results.
2. **Case sensitivity** — `Apple` and `apple` are treated as different lines unless `-i` is used.
3. **Confusing `-d` and `-D`** — `-d` shows one copy of each duplicate; `-D` shows every copy.
4. **`-f` counts fields, not characters** — use `-s` if you want to skip by character count instead.
5. **Whitespace differences** — trailing spaces or tabs can prevent lines from being recognized as duplicates.

---

## 16. Quick Reference Cheat Sheet

```bash
uniq file.txt                  # remove adjacent duplicate lines
sort file.txt | uniq           # remove all duplicates (sort first)
sort file.txt | uniq -c        # count occurrences of each line
sort file.txt | uniq -d        # show only duplicated lines (one copy)
sort file.txt | uniq -D        # show all copies of duplicated lines
sort file.txt | uniq -u        # show only lines that appear once
sort file.txt | uniq -i        # case-insensitive comparison
uniq -f1 file.txt              # ignore first field when comparing
uniq -s3 file.txt              # skip first 3 characters when comparing
uniq -w5 file.txt              # compare only first 5 characters
sort file.txt | uniq -c | sort -nr   # frequency count, most common first
```

---

## 17. Related Commands

- `sort` — sorts lines; almost always used before `uniq`
- `awk` / `cut` — extract specific fields/columns before piping into `sort | uniq`
- `comm` — compare two sorted files line by line
- `wc -l` — count total lines (often used alongside `uniq -c` for summaries)
- `shuf` — randomly shuffle lines (opposite goal of dedup/sort)

---

## 18. Summary

`uniq` is a lightweight but essential tool for deduplication and frequency analysis in text-processing pipelines. Its real power shows when combined with `sort` — the `sort | uniq -c | sort -nr` pattern is one of the most widely used one-liners in shell scripting for counting and ranking occurrences of anything from log entries to word frequencies.
