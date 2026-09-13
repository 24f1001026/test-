# Bash Built-ins Deep Dive: `shift`, `exec`, `eval`, and `getopts`

A complete reference with explanations and worked examples for four of the most
misunderstood bash built-ins. Each section covers: what it does, why it
exists, syntax, and every common usage pattern with example code and output.

---

## 1. `shift`

### What it does
`shift` moves the positional parameters (`$1`, `$2`, `$3`, ...) down by a
given number of places. After `shift`, what was `$2` becomes `$1`, what was
`$3` becomes `$2`, and so on. The parameter(s) shifted off are discarded —
there's no way to get them back once shifted.

### Syntax
```bash
shift [n]
```
- `n` is optional and defaults to `1`.
- `$#` (the count of positional parameters) decreases by `n` after the shift.
- If `n` is greater than `$#`, bash sets `$#` to 0 without error (in most
  shells) — but some strict modes will error, so guard against this.

### Why it's used
- Processing an unknown number of command-line arguments in a loop.
- Peeling off a "command" argument before parsing "options" (subcommand
  pattern, like `git <subcommand> ...`).
- Implementing manual option parsers (an alternative to `getopts`) when you
  need to support long options like `--verbose`.

### Example 1: Basic shift
```bash
#!/bin/bash
echo "Before shift: \$1=$1 \$2=$2 \$3=$3 (\$#=$#)"
shift
echo "After shift:  \$1=$1 \$2=$2 \$3=$3 (\$#=$#)"
```
Run as `./script.sh a b c`:
```
Before shift: $1=a $2=b $3=c ($#=3)
After shift:  $1=b $2=c $3= ($#=2)
```

### Example 2: Shift by more than 1
```bash
#!/bin/bash
echo "\$#=$# args: $@"
shift 2
echo "After shift 2: \$#=$# args: $@"
```
Run as `./script.sh a b c d`:
```
$#=4 args: a b c d
After shift 2: $#=2 args: c d
```

### Example 3: Looping through all arguments (the classic use case)
```bash
#!/bin/bash
while [ "$#" -gt 0 ]; do
    echo "Processing: $1"
    shift
done
```
Run as `./script.sh apple banana cherry`:
```
Processing: apple
Processing: banana
Processing: cherry
```
This pattern is preferred over `for arg in "$@"` when you also need to
consume "the next argument" inside the loop (e.g., an option that takes a
value), because `shift` lets you advance the cursor manually.

### Example 4: Subcommand dispatch (peel off the first arg, pass the rest along)
```bash
#!/bin/bash
cmd="$1"
shift   # remove the subcommand, everything else remains for the handler

case "$cmd" in
    start)
        echo "Starting with options: $@"
        ;;
    stop)
        echo "Stopping with options: $@"
        ;;
    *)
        echo "Unknown command: $cmd"
        ;;
esac
```
Run as `./deploy.sh start --force --env prod`:
```
Starting with options: --force --env prod
```

### Example 5: Manual long-option parsing with `shift 2`
```bash
#!/bin/bash
while [ "$#" -gt 0 ]; do
    case "$1" in
        --name)
            NAME="$2"
            shift 2      # consume both the flag and its value
            ;;
        --verbose)
            VERBOSE=1
            shift        # consume just the flag
            ;;
        *)
            echo "Unknown arg: $1"
            shift
            ;;
    esac
done
echo "NAME=$NAME VERBOSE=$VERBOSE"
```
Run as `./script.sh --name Alice --verbose`:
```
NAME=Alice VERBOSE=1
```
This is the standard way to support `--long-options` in bash, since
`getopts` (built-in) only understands single-character flags.

### Example 6: Guarding against shifting too far
```bash
#!/bin/bash
shift 5 2>/dev/null || echo "Nothing left to shift"
echo "Remaining: $#"
```
Run with no arguments:
```
Remaining: 0
```
Best practice: always check `$#` before calling `shift` in a loop condition,
as shown in Example 3, to avoid "shift: shift count out of range" errors
under `set -e` / strict shells.

---

## 2. `exec`

### What it does
`exec` has two related but distinct uses:

1. **Replace the current shell process** with another command, instead of
   forking a child process. The new program takes over the same PID; the
   shell script does not continue after this line.
2. **Redirect file descriptors for the current shell** for the rest of the
   script (or until changed again), without running any command.

### Syntax
```bash
exec [command [arguments]]
exec [n]<file          # redirect input
exec [n]>file          # redirect output
exec [n]>>file         # append output
exec [n]<&-            # close a file descriptor
exec [n]<&m            # duplicate file descriptor m onto n
```

### Why it's used
- Saves memory/process overhead: no new process is forked, so it's used as
  the last line of wrapper scripts.
- Ensures signals (like Ctrl+C) go directly to the replacing program instead
  of an intermediate shell.
- Lets you open persistent file descriptors for logging, reading line-by-line,
  or redirecting all future output of the script in one shot.

### Example 1: Replacing the shell with a command
```bash
#!/bin/bash
echo "About to exec..."
exec ls -la
echo "This line will NEVER run"
```
Because `exec` replaces the shell process, control never returns to the
script — the line after `exec ls -la` is dead code. This matters: if you
just called `ls -la` without `exec`, the shell would fork a child, wait for
it, then continue to the next line normally.

### Example 2: Wrapper script pattern (very common in Docker entrypoints)
```bash
#!/bin/bash
echo "Setting up environment..."
export APP_ENV=production
exec java -jar myapp.jar "$@"
```
This is the canonical Docker `ENTRYPOINT` pattern: the wrapper does setup,
then `exec`'s into the real process so that process becomes PID 1 and
receives signals (like `SIGTERM` on container stop) directly, instead of
being an orphaned grandchild of a shell that ignores the signal.

### Example 3: Redirecting all output of a script to a log file
```bash
#!/bin/bash
exec > logfile.txt 2>&1
echo "This goes to logfile.txt"
echo "So does this error" >&2
echo "Every subsequent command's output is captured too"
ls /nonexistent
```
After `exec > logfile.txt 2>&1`, every command in the rest of the script has
its stdout and stderr redirected to `logfile.txt`, without needing to
redirect each line individually.

### Example 4: Redirecting input from a file (read line-by-line)
```bash
#!/bin/bash
exec 3< names.txt        # open file on descriptor 3
while read -u 3 line; do
    echo "Name: $line"
done
exec 3<&-                 # close descriptor 3
```
This opens `names.txt` on a custom file descriptor (`3`) so the `while read`
loop doesn't collide with stdin (descriptor `0`), which is useful when the
script itself also needs to read user input elsewhere.

### Example 5: Duplicating file descriptors to temporarily silence output
```bash
#!/bin/bash
exec 4>&1          # save current stdout to descriptor 4
exec 1>/dev/null   # silence stdout
echo "You will not see this"
exec 1>&4          # restore stdout from descriptor 4
echo "You will see this"
```
Output:
```
You will see this
```
This "save and restore" pattern is used when a script needs to temporarily
suppress noisy command output, then resume normal output afterward.

### Example 6: Using `exec` to switch users/shells (login-style scripts)
```bash
#!/bin/bash
echo "Switching to bash login shell..."
exec bash -l
```
Common in `.profile`/`.bash_profile` chains or container init scripts that
hand off control to an interactive shell at the end.

### Key distinction to remember
| Without `exec`            | With `exec`                          |
|----------------------------|----------------------------------------|
| `command` forks a new child process, parent waits | `exec command` replaces the current process, no new PID |
| Script continues after the command finishes | Script never continues; that line is the end |
| Signals go to parent shell first | Signals go directly to the new program |

---

## 3. `eval`

### What it does
`eval` takes its arguments, concatenates them into a single string, and then
**parses and executes that string as if it had been typed directly into the
shell**. It runs the shell's parser twice: once on the literal `eval ...`
line, and again on the resulting string.

### Syntax
```bash
eval [arguments...]
```

### Why it's used
- Building a command dynamically from variables and then running it.
- Evaluating variable indirection (getting/setting a variable whose name is
  stored in another variable) — largely superseded by `${!var}` and
  `declare -n` in modern bash, but still seen in portable scripts.
- Parsing structured output like `key=value` pairs from another command
  (e.g., `eval $(ssh-agent -s)`) into real shell variables.

### ⚠️ Security warning
`eval` executes arbitrary text as code. If any part of the string comes from
untrusted input (user input, a network response, a file you don't control),
`eval` becomes a code-injection vector. Only use `eval` with strings you
fully control or have sanitized.

### Example 1: Building a command from parts
```bash
#!/bin/bash
cmd="echo"
args="Hello World"
eval "$cmd $args"
```
Output:
```
Hello World
```
Here this is trivial and doesn't need `eval` at all (`$cmd $args` would work
directly) — but it illustrates the mechanism before more complex cases.

### Example 2: Variable indirection (the classic pre-bash-4.3 use case)
```bash
#!/bin/bash
name="fruit"
fruit="apple"
eval "value=\$$name"
echo "$value"       # prints: apple
```
What happens: `eval` first sees the literal string `value=$fruit` after
substitution (`$$name` → `$fruit`), then re-parses and runs that, setting
`value=apple`.

Modern equivalent (no `eval` needed, safer):
```bash
name="fruit"
fruit="apple"
echo "${!name}"      # indirect expansion, prints: apple
```

### Example 3: Loading environment variables from a command's output
```bash
eval "$(ssh-agent -s)"
```
`ssh-agent -s` prints text like:
```
SSH_AUTH_SOCK=/tmp/ssh-XXXX/agent.1234; export SSH_AUTH_SOCK;
SSH_AGENT_PID=1234; export SSH_AGENT_PID;
```
`eval` runs that printed text as real shell commands, which is how
`ssh-agent`'s environment variables get exported into your current shell
(a subshell running `ssh-agent -s` on its own can't modify your shell's
environment — `eval` is the trick that lets it happen).

### Example 4: Dynamically constructing and running a pipeline
```bash
#!/bin/bash
filter="grep error"
sort_cmd="sort -k2"
eval "cat access.log | $filter | $sort_cmd"
```
This builds a pipeline from configurable pieces. This is a case where
`eval` genuinely does something string concatenation alone can't: it lets
`|`, `>`, `&&`, etc., inside the variables be *interpreted* as shell syntax
rather than literal characters passed as arguments.

### Example 5: Dynamically naming variables (simulating arrays of variables)
```bash
#!/bin/bash
for i in 1 2 3; do
    eval "var$i=value$i"
done
echo "$var1 $var2 $var3"
```
Output:
```
value1 value2 value3
```
Modern equivalent using an actual associative array (safer, preferred):
```bash
declare -A vars
for i in 1 2 3; do
    vars[$i]="value$i"
done
echo "${vars[1]} ${vars[2]} ${vars[3]}"
```

### Example 6: The danger of `eval` with untrusted input
```bash
#!/bin/bash
user_input='hello; rm -rf /tmp/testdir'
eval "echo $user_input"
```
This doesn't just echo the string — it runs `echo hello` **and then**
`rm -rf /tmp/testdir` because `eval` interprets the `;` as a command
separator. This is exactly why `eval` should never touch unsanitized input.

---

## 4. `getopts`

### What it does
`getopts` is bash's built-in, POSIX-standard parser for **single-character**
command-line options (like `-v`, `-f filename`, `-h`), including bundled
flags like `-vf`. It's the standard, robust way to parse short options in
shell scripts, as opposed to hand-rolling a parser with `shift`/`case`.

### Syntax
```bash
getopts optstring name [args...]
```
- `optstring` is a string listing the valid option letters.
  - A letter followed by a colon (`:`) means that option **requires an
    argument** (e.g., `f:` means `-f` needs a value, accessed via `$OPTARG`).
  - A leading colon in the optstring (`:abc:`) switches on **silent error
    handling**, so you can produce custom error messages instead of bash's
    default ones.
- `name` is the variable that `getopts` will set on each call to the current
  option letter (conventionally `opt`).
- Special variables used by `getopts`:
  - `$OPTARG` — holds the argument value when an option requires one.
  - `$OPTIND` — the index of the next argument to process (starts at 1;
    must be reset to 1 if you call `getopts` more than once in the same
    shell session, e.g., across function calls).

### Why it's used over manual parsing
- Correctly handles bundled short flags: `-vf file.txt` is the same as
  `-v -f file.txt`.
- Handles `--` to signal "end of options."
- Gives consistent error messages and error-handling modes.
- It's POSIX-standard, so scripts using it are portable across `sh`/`bash`/
  `ksh`/`dash`.
- Limitation to know up front: `getopts` does **not** support GNU-style long
  options (`--verbose`) natively — for that, you fall back to a manual
  `shift`/`case` parser (see `shift` Example 5) or `getopt` (the external
  command, note no "s") which is less portable.

### Example 1: A single boolean flag
```bash
#!/bin/bash
while getopts "v" opt; do
    case "$opt" in
        v) echo "Verbose mode ON" ;;
    esac
done
```
Run as `./script.sh -v`:
```
Verbose mode ON
```

### Example 2: An option that requires an argument
```bash
#!/bin/bash
while getopts "f:" opt; do
    case "$opt" in
        f) echo "Filename given: $OPTARG" ;;
    esac
done
```
Run as `./script.sh -f data.txt`:
```
Filename given: data.txt
```
Note the space between `-f` and its value is optional: `-fdata.txt` also
works, because `getopts` knows `f:` requires an argument and grabs the rest
of that token.

### Example 3: Multiple options, some with arguments, some without
```bash
#!/bin/bash
verbose=0
file=""
while getopts "vf:o:" opt; do
    case "$opt" in
        v) verbose=1 ;;
        f) file="$OPTARG" ;;
        o) outfile="$OPTARG" ;;
        \?) echo "Invalid option: -$OPTARG" >&2; exit 1 ;;
    esac
done
echo "verbose=$verbose file=$file outfile=$outfile"
```
Run as `./script.sh -v -f input.txt -o output.txt`:
```
verbose=1 file=input.txt outfile=output.txt
```
Run as `./script.sh -vf input.txt -o output.txt` (bundled flag):
```
verbose=1 file=input.txt outfile=output.txt
```
Both invocations behave identically — this bundling support is one of the
main reasons to use `getopts` over manual parsing.

### Example 4: Handling positional arguments after options (`shift $((OPTIND -1))`)
```bash
#!/bin/bash
while getopts "v" opt; do
    case "$opt" in
        v) verbose=1 ;;
    esac
done
shift $((OPTIND - 1))    # remove all parsed options, leaving positional args

echo "verbose=$verbose"
echo "Remaining positional args: $@"
```
Run as `./script.sh -v file1.txt file2.txt`:
```
verbose=1
Remaining positional args: file1.txt file2.txt
```
This `shift $((OPTIND - 1))` line is the standard idiom that bridges
`getopts` (option parsing) with `shift`/`$@` (positional argument access) —
`OPTIND` tells you how many tokens `getopts` consumed, so you shift exactly
that many off to expose the remaining plain arguments.

### Example 5: Custom error handling with the leading colon
```bash
#!/bin/bash
while getopts ":f:v" opt; do
    case "$opt" in
        f) echo "File: $OPTARG" ;;
        v) echo "Verbose on" ;;
        \?) echo "Error: invalid option -$OPTARG" >&2 ;;
        :) echo "Error: option -$OPTARG requires an argument" >&2 ;;
    esac
done
```
Run as `./script.sh -f` (missing required argument):
```
Error: option -f requires an argument
```
Run as `./script.sh -x` (unknown option):
```
Error: invalid option -x
```
The leading `:` in `":f:v"` disables bash's own built-in error messages and
lets your `case` block produce custom, cleaner ones via the `:` and `\?`
cases.

### Example 6: A complete, realistic option parser
```bash
#!/bin/bash
usage() {
    echo "Usage: $0 [-v] [-o output_file] -i input_file"
    exit 1
}

verbose=0
input=""
output="out.txt"

while getopts ":vi:o:h" opt; do
    case "$opt" in
        v) verbose=1 ;;
        i) input="$OPTARG" ;;
        o) output="$OPTARG" ;;
        h) usage ;;
        \?) echo "Invalid option: -$OPTARG" >&2; usage ;;
        :) echo "Option -$OPTARG requires an argument" >&2; usage ;;
    esac
done
shift $((OPTIND - 1))

if [ -z "$input" ]; then
    echo "Error: -i input_file is required" >&2
    usage
fi

echo "verbose=$verbose input=$input output=$output"
echo "Extra positional args: $@"
```
Run as `./script.sh -v -i in.txt -o out.txt extra1 extra2`:
```
verbose=1 input=in.txt output=out.txt
Extra positional args: extra1 extra2
```
Run as `./script.sh` (missing required `-i`):
```
Error: -i input_file is required
Usage: ./script.sh [-v] [-o output_file] -i input_file
```
This is the idiomatic, production-quality shape of an option-parsing block
in real-world bash scripts: `getopts` loop → `shift $((OPTIND - 1))` →
validate required values → proceed.

### Example 7: Resetting `OPTIND` when calling `getopts` twice in one session
```bash
#!/bin/bash
parse_args() {
    local OPTIND=1     # local + reset, so this function is reusable/reentrant
    while getopts "a:" opt; do
        case "$opt" in
            a) echo "Got: $OPTARG" ;;
        esac
    done
}

parse_args -a first
parse_args -a second
```
Output:
```
Got: first
Got: second
```
Without `local OPTIND=1` inside the function, the second call would silently
fail to parse anything, because `OPTIND` would still be pointing past the
end of the previous argument list. This is one of the most common
`getopts` bugs in real scripts.

---

## Quick Reference Summary

| Command   | Purpose                                                         | Key variables/idioms                         |
|-----------|------------------------------------------------------------------|-----------------------------------------------|
| `shift`   | Drop leading positional parameters, shifting the rest down       | `$#`, `$@`, `shift n`                         |
| `exec`    | Replace the shell process with a command, or redirect FDs        | `exec cmd`, `exec > file`, `exec n<&-`        |
| `eval`    | Re-parse and execute a constructed string as shell code           | Use with fully trusted input only              |
| `getopts` | Parse short (`-x`) options, including bundled flags               | `$OPTARG`, `$OPTIND`, `shift $((OPTIND - 1))` |
