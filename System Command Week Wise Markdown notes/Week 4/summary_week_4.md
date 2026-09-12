# Week 4 Revision Summary — `find`, `grep` & Pattern Matching (Regex)

Quick-revision notes covering all three source files: `find`, the complete `grep` guide, and the Pattern Matching (Regex) lecture notes. Edge cases and "gotchas" are folded directly into each topic (marked with ⚠️) instead of a separate section, so you learn the trap right where the rule lives.

---

## 1. `find` — Locating Files

**Syntax:** `find [path] [expression]` — walks the **live filesystem** in real time (unlike `locate`, which uses a possibly stale database).

⚠️ **Path must come first.** `find -name "x" .` → error (`paths must precede expression`). Always: `find . -name "x"`.

### 1.1 Core search tests

| Test | Purpose | Example |
|---|---|---|
| `-name` / `-iname` | Match filename (case-sensitive / insensitive) | `find . -iname "readme*"` |
| `-type f/d/l/b/c/p/s` | Filter by file/dir/symlink/device/pipe/socket | `find . -type f -name "*.sh"` |
| `-size +N/-N/N[c\|k\|M\|G]` | Filter by size (greater/less/exact) | `find . -size +100M` |
| `-mtime/-atime/-ctime ±N` | Filter by modify/access/change time in days | `find . -mtime +30` |
| `-mmin/-amin/-cmin ±N` | Same, but in minutes | `find . -mmin -30` |
| `-newer file` | Modified more recently than reference file | `find . -newer ref.txt` |
| `-perm mode` | Filter by permission bits | `find . -perm -4000` |
| `-user/-group/-uid` | Filter by owner | `find /home -user john` |
| `-empty` | Empty files or directories | `find . -type f -empty` |
| `-inum N` | Find by inode (hard links) | `find . -inum 123456` |
| `-maxdepth/-mindepth N` | Limit recursion depth | `find . -maxdepth 1` |

⚠️ **`-perm` has three distinct meanings** — this is a classic exam trap:
- `-perm 755` → **exact** match only
- `-perm -755` → **all** these bits set (others may also be set)
- `-perm /755` → **any** of these bits set is enough

⚠️ **Wildcards must be quoted** (`-name "*.txt"`, not `-name *.txt`) or the shell expands them before `find` ever sees them.

### 1.2 Logical operators

| Operator | Meaning | Example |
|---|---|---|
| `-a` (implicit) | AND | `find . -name "*.txt" -size +1k` |
| `-o` | OR | `find . \( -name "*.jpg" -o -name "*.png" \)` |
| `!` / `-not` | NOT | `find . ! -name "*.txt"` |
| `\( \)` | Grouping (must escape parens in bash) | `find . \( -name "*.txt" -o -name "*.log" \)` |

### 1.3 Taking action on results

| Action | Behavior | Example |
|---|---|---|
| `-exec cmd {} \;` | Run command once **per file** | `find . -name "*.sh" -exec chmod +x {} \;` |
| `-exec cmd {} +` | Run command in **batches** (faster) | `find . -name "*.tmp" -exec rm {} +` |
| `-ok cmd {} \;` | Same as `-exec` but **prompts** before each action | `find . -name "*.tmp" -ok rm {} \;` |
| `-delete` | Delete matches directly | `find . -type d -empty -delete` |
| `\| xargs cmd` | Batch commands via a separate tool (fast on huge result sets) | `find . -name "*.log" \| xargs rm` |

⚠️ **Always dry-run before `-delete`.** Run the identical `find` command *without* `-delete` first to preview what will be removed — accidental mass-deletion is the #1 real-world `find` mistake.

⚠️ **Filenames with spaces break `xargs`.** Whitespace is the default separator, so `My File.txt` gets split into two arguments. Fix: `find ... -print0 | xargs -0 ...` (NUL-separated, safe for spaces/special characters).

### 1.4 Excluding directories (performance)

```bash
find . -path "./node_modules" -prune -o -name "*.js" -print
find . -not -path "./.git/*" -name "*.py"     # simpler alternative to -prune
```
⚠️ Searching `/` without `-maxdepth` or `-prune` (e.g. to skip `/proc`) can be extremely slow — always scope big searches.

### 1.5 Custom output

```bash
find . -type f -printf "%s %p\n"     # size + full path
```
`%p` = full path, `%f` = filename only, `%s` = size, `%u` = owner, `%TY-%Tm-%Td` = mod date.

---

## 2. `grep` — Searching Text

**Full name:** Global Regular Expression Print — historically derived from the Unix `ed` command `g/re/p`.

**Syntax:** `grep [OPTIONS] PATTERN [FILE...]` — reads from files, or **stdin** if no file is given.

### 2.1 The three "flavors" (regex engines)

| Command | Engine | Flag equivalent | Notes |
|---|---|---|---|
| `grep` | BRE (Basic) | `-G` (default) | `+ ? \| ( )` need `\` to be special |
| `egrep` | ERE (Extended) | `-E` | `+ ? \| ( )` work unescaped |
| `fgrep` | Fixed string, no regex | `-F` | treats pattern literally |
| — | PCRE (Perl-compatible) | `-P` (GNU only) | adds `\d`, `\w`, lookahead/lookbehind |

⚠️ `egrep`/`fgrep` are **deprecated** in modern GNU docs in favor of `grep -E` / `grep -F` — know both but prefer the explicit flag form.

### 2.2 Most-used flags

| Flag | Meaning | Flag | Meaning |
|---|---|---|---|
| `-i` | Case-insensitive | `-A/-B/-C n` | Context lines after/before/around |
| `-v` | Invert match | `-e` | Multiple patterns |
| `-n` | Show line numbers | `-f file` | Patterns from a file |
| `-c` | Count matches only | `-q` | Quiet — exit status only |
| `-l` / `-L` | Filenames with / without a match | `-m n` | Stop after n matches |
| `-r` / `-R` | Recursive | `-H` / `-h` | Force show / hide filename |
| `-w` | Whole word only | `-s` | Suppress file-error messages |
| `-x` | Whole **line** only | `-z` | NUL-separated input (pairs with `find -print0`) |
| `-o` | Print only the matched text | `--color` | Highlight matches |

⚠️ **`-w` vs `-x`:** `-w` matches the pattern as a whole *word* anywhere in the line (`cat` but not `category`); `-x` requires the pattern to match the **entire line** (`done` matches only if the whole line is exactly "done", not "task done").

⚠️ Flags can be **stacked**: `grep -rniw "password" ./src` = recursive + case-insensitive + line numbers + whole word.

### 2.3 Regex metacharacters (common to BRE & ERE)

| Symbol | Meaning | Example | Matches |
|---|---|---|---|
| `.` | Any single character | `c.t` | cat, cot, c9t |
| `*` | 0+ of preceding char | `ab*c` | ac, abc, abbbc |
| `^` | Start of line (or negation inside `[]`) | `^Error` | lines starting "Error" |
| `$` | End of line | `done$` | lines ending "done" |
| `[ ]` | Character class | `[a-c]` | a, b, or c |
| `[^ ]` | Negated class | `[^0-9]` | any non-digit |
| `\` | Escape | `\.` | a literal dot |

⚠️ **`.` matches any single character — including a space.** `cat.cat` matches `catXcat`, `cat1cat`, and even `cat cat`.

⚠️ To match a **literal dot**, you must escape it: `\.`. Unescaped, `.` is a wildcard, so `report.txt` as a pattern would also match `reportXtxt`.

### 2.4 BRE vs ERE — the syntax that actually changes

| Feature | BRE (default) | ERE (`-E` / `egrep`) |
|---|---|---|
| Grouping | `\( \)` | `( )` |
| Repetition range | `\{n,m\}` | `{n,m}` |
| One or more | not directly available | `+` |
| Zero or one | not directly available | `?` |
| Alternation (OR) | not available | `\|` |

⚠️ **Classic exam trap:** `\{n,m\}` (or `{n,m}` in ERE) repeats only the **single character or group immediately before it** — not the whole preceding word. `pattern\{2,4\}` repeats just the final `n`, matching `patternn`…`patternnnn`, **not** two-to-four repeats of the word "pattern". To repeat the whole word, group it first: `\(pattern\)\{2,4\}`.

⚠️ **Alternation has the lowest precedence in ERE.** `ab|cd` matches `ab` OR `cd` as whole tokens — it does **not** mean `a(b|c)d`. Use explicit grouping `(ab|cd)` when combining alternation with other characters.

### 2.5 POSIX character classes

`[[:digit:]]`, `[[:alpha:]]`, `[[:alnum:]]`, `[[:upper:]]`, `[[:lower:]]`, `[[:space:]]`, `[[:punct:]]`, `[[:print:]]`, `[[:graph:]]`, `[[:blank:]]`, `[[:xdigit:]]`, `[[:cntrl:]]`

⚠️ Must be used **inside** a bracket expression — `[[:digit:]]` is valid, `:digit:` alone is not.
⚠️ They are **locale-aware** (e.g. `[[:alpha:]]` may include accented letters depending on system locale).
⚠️ `-v '[[:cntrl:]]'` inverts the whole **line** match, not individual characters — a very commonly tested point.

**Easy-to-confuse classes, side by side:**

| Class | Includes space? | Letters/digits? | Punctuation? |
|---|---|---|---|
| `[[:print:]]` | ✅ Yes | ✅ Yes | ✅ Yes |
| `[[:graph:]]` | ❌ No | ✅ Yes | ✅ Yes |
| `[[:blank:]]` | ✅ Yes (space & tab only) | ❌ No | ❌ No |
| `[[:space:]]` | ✅ Yes (all whitespace incl. newline) | ❌ No | ❌ No |

⚠️ `[[:graph:]]` = printable chars **except space**; `[[:print:]]` = printable chars **including space** — the pair most likely to be mixed up on an exam.

**Skipping blank lines** (two equivalent ways):
```bash
grep -v '^$' file      # explicitly exclude lines with nothing between start/end
grep '.' file           # requires at least one character — same effect
```

### 2.6 Backreferences

`\1`–`\9` refer to text captured by an earlier `\( \)` group — used to detect repeated/duplicate content.

```bash
grep '\(hello\).*\1' file.txt      # a line containing "hello" twice
```
⚠️ A backreference matches the **exact matched text**, not the pattern that produced it. If `\(a*\)` matched `aaa`, then `\1` will only match `aaa` again — not `a`, `aa`, or any other count.

### 2.7 Quantifier traps that look similar but aren't

| Pattern | Meaning | Matches | Doesn't match |
|---|---|---|---|
| `M*a` | 0+ **literal `M`**, then `a` | `a`, `Ma`, `MMa` | `Mxa` |
| `M.*a` | `M`, then 0+ **any** char, then `a` | `Ma`, `Mxa`, `M123a` | — |
| `^M*` | Start, then 0+ `M`'s | **every line** (0 M's still satisfies it) | — |
| `^M+` | Start, then 1+ `M`'s | `M...`, `MM...` | `AM...` (M not at start) |

⚠️ `^M*` is a classic trick pattern — because `*` allows **zero** occurrences, it technically matches every line, not just lines with M's. Use `+` (ERE) when you mean "at least one."

**Grouped repetition** — `+`/`*` can apply to a whole `( )` group, not just one character:
```bash
egrep '(ma)+' file.txt     # ma, mama, mamama — one or more repeats of the whole group
egrep '(ma)*' file.txt     # same, but zero repeats also counts (matches almost everything)
```

**Alternation grouping** — `egrep '(ED|ME)' file.txt` matches lines containing either `ED` or `ME` (e.g. LOVED, NAMED, SOME) as a substring anywhere on the line.

⚠️ **Standalone-number matching needs `\b` on both sides.** `egrep '\b[[:digit:]]{6}\b'` matches a clean 6-digit PIN like `221005`, but will **not** match the 6-digit substring inside a longer number like `1234567` — because there's no word boundary in the middle of a longer digit run. Forgetting the boundaries is a common way to over-match.

### 2.8 Word boundaries & practical patterns

- `\b` = word boundary (GNU extension, not POSIX-core). `cat\b` matches `cat`, `cat.`, `cat,` but not `category`.
- `grep -oP '(?<=\$)\d+'` — PCRE lookbehind, extracts digits after a `$` without capturing the `$` itself.
- `grep -oE "([0-9]{1,3}\.){3}[0-9]{1,3}"` — simplified IPv4 matcher.
- `grep -oE "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}"` — simplified email matcher.
- Structured-ID pattern with boundaries: `egrep '\b[[:alpha:]]{2}[[:digit:]]{2}[[:alpha:]][[:digit:]]{2}\b'`.

⚠️ **Exclude `grep` matching itself in `ps` output** with the classic bracket trick: `ps aux | grep "[c]hrome"` — the bracket makes the pattern not match the literal string "grep chrome" in the process list.

### 2.9 Exit status (for scripting)

| Code | Meaning |
|---|---|
| `0` | Match found |
| `1` | No match found |
| `2` | Error (e.g. file not found, bad pattern) |

```bash
grep -q "ERROR" log.txt && echo "Errors found!" || echo "All clear."
```

### 2.10 grep variants

`zgrep` (search inside `.gz` without unzipping), `rgrep` (= `grep -r`).

⚠️ **Use `-F` for literal strings with regex-special characters.** `grep -F "3.14 * r^2" formulas.txt` — without `-F`, the `.`, `*`, and `^` would all be interpreted as regex metacharacters instead of literal text.

---

## 3. `cut` — Companion Field-Extraction Tool

Often piped after `grep` to pull out specific columns.

| Flag | Meaning | Example |
|---|---|---|
| `-c range` | Character positions | `cut -c 1-4 file` |
| `-d "delim"` | Delimiter (default: tab) | `cut -d "," -f 2 file` |
| `-f n` | Field number(s), used with `-d` | |

```bash
cat file | cut -d ";" -f 2 | cut -d "," -f 1     # chain cuts to drill into nested delimiters
```

⚠️ `-d` is **required** whenever you specify a delimiter — `cut "," -f 1` (missing `-d`) is a common typo and won't work as intended.

⚠️ **`grep -oP` with lookaround can replace a `cut` pipeline.** Example: `echo "id;name,age;email" | grep -oP '(?<=;)[^,;]+(?=,)'` extracts `name` directly, equivalent to chaining two `cut` calls.

---

## 4. Cross-Tool Combinations Worth Memorizing

| Goal | Command |
|---|---|
| Find `.js` files, skip `node_modules` | `find . -path "./node_modules" -prune -o -name "*.js" -print` |
| Delete all `.tmp` files safely | `find . -name "*.tmp" -exec rm {} +` |
| Search recursively for TODOs, exclude logs | `grep -r "TODO" ./project --exclude="*.log"` |
| Count matching files (not lines) | `grep -c "ERROR" *.log` |
| Safe batch operation on files with spaces | `find . -name "*.jpg" -print0 \| xargs -0 rm` |
| Live log filtering | `tail -f server.log \| grep "ERROR"` |
| AND logic (line must match both) | `grep "error" file.txt \| grep "database"` |
| Isolate one field from one specific line | `cut -d "/" -f 3 \| cut -d " " -f 1 \| head -n 19 \| tail -n 1` |

---

## 5. Quick Cheat Sheet

```bash
# find
find . -name "*.txt"                    # by name
find . -type f -size +100M              # by type + size
find . -mtime +30 -delete               # cleanup, always dry-run first
find . -maxdepth 1 -type f              # limit depth
find . \( -name "*.jpg" -o -name "*.png" \) -size +5M

# grep
grep -i "pattern" file                  # case-insensitive
grep -rn "pattern" dir/                 # recursive + line numbers
grep -v "pattern" file                  # invert match
grep -oE "x{2,4}" file                  # ERE repetition
grep -E "a|b" file                      # OR
grep -P "\d+" file                      # PCRE
grep -q "pattern" file                  # scripting (exit status only)
grep '\(x\).*\1' file                   # backreference / duplicate check

# cut
cut -c 1-4 file
cut -d "," -f 2 file
```

---

*End of Week 4 revision summary — sources: `Linux_Find_Command_Notes.md`, `grep-complete-guide.md`, `Pattern_Matching_Week4_Notes.md`.*
