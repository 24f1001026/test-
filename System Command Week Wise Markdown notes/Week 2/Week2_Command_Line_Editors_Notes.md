# Week 2 (Lecture 1–3): Command Line Editors
### *Working with Text Files in the Terminal*

---

## 📑 Table of Contents

1. [Introduction to Text Editors](#1-introduction-to-text-editors)
   - [1.1 Categories of Editors](#11-categories-of-editors)
   - [1.2 Common Features of Editors](#12-common-features-of-editors)
2. [The `ed` Line Editor](#2-the-ed-line-editor)
   - [2.1 Concept](#21-concept)
   - [2.2 Syntax](#22-syntax)
   - [2.3 Address / Location Commands](#23-address--location-commands)
   - [2.4 Editing Commands](#24-editing-commands)
   - [2.5 File & Shell Commands](#25-file--shell-commands)
   - [2.6 Examples](#26-examples)
3. [The `ex` Commands](#3-the-ex-commands)
   - [3.1 Concept](#31-concept)
   - [3.2 Commands](#32-commands)
   - [3.3 Examples](#33-examples)
4. [The `nano` Editor](#4-the-nano-editor)
   - [4.1 Concept](#41-concept)
   - [4.2 File Handling](#42-file-handling)
   - [4.3 Editing](#43-editing)
   - [4.4 Search and Replace](#44-search-and-replace)
   - [4.5 Deletion](#45-deletion)
   - [4.6 Information](#46-information)
   - [4.7 Moving Around](#47-moving-around)
   - [4.8 Special Movement](#48-special-movement)
   - [4.9 Various / Miscellaneous](#49-various--miscellaneous)
   - [4.10 Examples](#410-examples)
5. [The `vi` Editor](#5-the-vi-editor)
   - [5.1 Concept](#51-concept)
   - [5.2 Modes in vi](#52-modes-in-vi)
   - [5.3 Exiting vi](#53-exiting-vi)
   - [5.4 Basic Movement](#54-basic-movement)
   - [5.5 Advanced Movement (Command Mode)](#55-advanced-movement-command-mode)
   - [5.6 Screen Manipulation](#56-screen-manipulation)
   - [5.7 Changing Text](#57-changing-text)
   - [5.8 Deleting Text](#58-deleting-text)
   - [5.9 Copy–Paste Text](#59-copypaste-text)
   - [5.10 Searching Text](#510-searching-text)
   - [5.11 Examples](#511-examples)
6. [The `emacs` Editor](#6-the-emacs-editor)
   - [6.1 Concept & Notation](#61-concept--notation)
   - [6.2 Moving Around](#62-moving-around)
   - [6.3 Exiting emacs](#63-exiting-emacs)
   - [6.4 Copy–Paste](#64-copypaste)
   - [6.5 Searching Text](#65-searching-text)
   - [6.6 Examples](#66-examples)
7. [Comparison of All Editors](#7-comparison-of-all-editors)
8. [Summary](#8-summary)

---

## 1. Introduction to Text Editors

### 1.1 Categories of Editors

Text editors used for working with files can be broadly classified into **four categories**, ranging from the most basic (line-based) to the most advanced (full IDEs).

| Category | Description | Examples |
|---|---|---|
| **Line Editors** | Operate on one line of text at a time; no visual screen representation | `ed` |
| **Terminal Editors** | Run inside a terminal but provide a full-screen, interactive view of the file | `vi`, `emacs`, `pico`, `nano` |
| **GUI Editors** | Graphical, mouse-driven editors with visual interfaces | KDE, Gnome, Bluefish, `kate`, `kwrite`, `gedit`, Sublime Text, Atom, Brackets |
| **IDE (Integrated Development Environment)** | Full development environments combining editing, debugging, compiling, and project management | Eclipse, NetBeans |

> **Note:** As we move from Line Editors → Terminal Editors → GUI Editors → IDEs, the tools become more visual and feature-rich, but line/terminal editors remain essential for remote server administration (SSH sessions) where GUI tools are unavailable.

### 1.2 Common Features of Editors

Most text editors (regardless of category) support the following core features:

| # | Feature | Description |
|---|---|---|
| 1 | **Scrolling & View Modes** | Ability to scroll through file content and know the current cursor position |
| 2 | **Navigation** | Move by character, word, line, or pattern (search) |
| 3 | **Insert / Replace / Delete** | Basic text-modification operations |
| 4 | **Cut–Copy–Paste** | Standard clipboard-style operations |
| 5 | **Search–Replace** | Find and substitute text patterns |
| 6 | **Syntax Highlighting** | Language-aware color coding of code |
| 7 | **Key-maps, Init Scripts, Macros** | Custom key bindings, startup configuration, recorded action sequences |
| 8 | **Plugins** | Extendable functionality via add-ons |

---

## 2. The `ed` Line Editor

### 2.1 Concept

`ed` is the **original UNIX line editor** — the ancestor of `vi`, `ex`, and `sed`. It edits text **one line at a time** and has **no visual display** of the file; it only shows what you explicitly ask it to print. It works by operating on an internal **buffer**, which is a temporary in-memory copy of the file being edited.

To display the `ed` prompt (`*`), use:

```
P
```

### 2.2 Syntax

The general command format in `ed` is:

```
[address[,address]] command [parameters]
```

- `address` — specifies which line(s) the command applies to
- `command` — a single-letter command (e.g., `p`, `d`, `a`)
- `parameters` — optional extra arguments for the command

### 2.3 Address / Location Commands

| Command | Meaning |
|---|---|
| `.` | Current line |
| `$` | Last line of the file |
| `%` | All lines in the file |
| `+` | Next line |
| `-` | Previous line |
| `,` | Range separator (e.g., `1,5`) |
| `;` | Range separator using current line as start |
| `/RE/` | Line matching regular expression `RE` |

### 2.4 Editing Commands

| Command | Meaning |
|---|---|
| `f` | Show name of file being edited |
| `p` | Print the current line |
| `a` | Append text after the current line |
| `c` | Change (replace) the current line |
| `d` | Delete the current line |
| `i` | Insert text before the current line |
| `j` | Join lines |
| `s` | Search for a regex pattern |
| `m` | Move current line to a new position |
| `u` | Undo the latest change |

### 2.5 File & Shell Commands

| Command | Meaning |
|---|---|
| `!command` | Execute a shell command |
| `e filename` | Edit a file |
| `r filename` | Read file contents into the buffer |
| `r !command` | Read output of a shell command into the buffer |
| `w filename` | Write buffer contents to a file |
| `q` | Quit `ed` |

### 2.6 Examples

```bash
$ ed myfile.txt        # Open myfile.txt in ed
*                       # ed prompt (after enabling with P)
1,5p                    # Print lines 1 to 5
3d                      # Delete line 3
a                       # Enter append/insert mode after current line
This is new text.
.                       # A single dot ends insert mode
5,10s/old/new/          # Search and replace 'old' with 'new' in lines 5-10
w                       # Write (save) the buffer to file
q                       # Quit ed
```

---

## 3. The `ex` Commands

### 3.1 Concept

`ex` is a **line-oriented editor** closely related to `ed`, and it forms the command-line backbone of `vi` (every `vi` command starting with `:` is actually an `ex` command). It shares many command names with `ed` but is more powerful in terms of scripting and batch editing.

### 3.2 Commands

| Command | Meaning |
|---|---|
| `f` | Show the name of the file being edited |
| `p` | Print the current line |
| `a` | Append text at the current line |
| `c` | Change the current line |
| `d` | Delete the current line |
| `i` | Insert a line at the current position |
| `j` | Join lines |
| `s` | Search for a regex pattern |
| `m` | Move the current line to a new position |
| `u` | Undo the latest change |

### 3.3 Examples

```
:1,10d          " Delete lines 1 through 10
:%s/foo/bar/g   " Replace all occurrences of 'foo' with 'bar' in the file
:w              " Write (save) changes
:q              " Quit
```

---

## 4. The `nano` Editor

*Reference: [https://www.nano-editor.org/dist/latest/cheatsheet.html](https://www.nano-editor.org/dist/latest/cheatsheet.html)*

### 4.1 Concept

`nano` is a **simple, beginner-friendly, full-screen terminal text editor**. Unlike `ed`/`ex`, it shows the entire file content on screen along with a help bar of shortcut keys at the bottom, making it ideal for quick edits without needing to memorize modes.

**Basic syntax to open nano:**

```bash
nano filename
```

### 4.2 File Handling

| Shortcut | Action |
|---|---|
| `Ctrl+S` | Save current file |
| `Ctrl+O` | Offer to write file ("Save As") |
| `Ctrl+R` | Insert a file into the current one |
| `Ctrl+X` | Close buffer / exit nano |

### 4.3 Editing

| Shortcut | Action |
|---|---|
| `Ctrl+K` | Cut current line into cut-buffer |
| `Alt+6` | Copy current line into cut-buffer |
| `Ctrl+U` | Paste contents of cut-buffer |
| `Alt+T` | Cut until end of buffer |
| `Ctrl+]` | Complete current word |
| `Alt+3` | Comment / uncomment line or region |
| `Alt+U` | Undo last action |
| `Alt+E` | Redo last undone action |

### 4.4 Search and Replace

| Shortcut | Action |
|---|---|
| `Ctrl+Q` | Start backward search |
| `Ctrl+W` | Start forward search |
| `Alt+Q` | Find next occurrence backward |
| `Alt+W` | Find next occurrence forward |
| `Alt+R` | Start a replace session |

### 4.5 Deletion

| Shortcut | Action |
|---|---|
| `Ctrl+H` | Delete character before cursor |
| `Ctrl+D` | Delete character under cursor |
| `Alt+Backspace` | Delete word to the left |
| `Ctrl+Del` | Delete word to the right |
| `Alt+Del` | Delete current line |

### 4.6 Information

| Shortcut | Action |
|---|---|
| `Ctrl+C` | Report cursor position |
| `Alt+D` | Report line / word / character count |
| `Ctrl+G` | Display help text |

### 4.7 Moving Around

| Shortcut | Action |
|---|---|
| `Ctrl+B` | One character backward |
| `Ctrl+F` | One character forward |
| `Ctrl+←` | One word backward |
| `Ctrl+→` | One word forward |
| `Ctrl+A` | To start of line |
| `Ctrl+E` | To end of line |
| `Ctrl+P` | One line up |
| `Ctrl+N` | One line down |
| `Ctrl+↑` | To previous block |
| `Ctrl+↓` | To next block |
| `Ctrl+Y` | One page up |
| `Ctrl+V` | One page down |
| `Alt+\` | To top of buffer |
| `Alt+/` | To end of buffer |

### 4.8 Special Movement

| Shortcut | Action |
|---|---|
| `Alt+G` | Go to specified line |
| `Alt+]` | Go to complementary bracket |
| `Alt+↑` | Scroll viewport up |
| `Alt+↓` | Scroll viewport down |
| `Alt+<` | Switch to preceding buffer |
| `Alt+>` | Switch to succeeding buffer |

### 4.9 Various / Miscellaneous

| Shortcut | Action |
|---|---|
| `Alt+A` | Turn the mark on/off |
| `Tab` | Indent marked region |
| `Shift+Tab` | Unindent marked region |
| `Alt+V` | Enter next keystroke verbatim |
| `Alt+N` | Turn line numbers on/off |
| `Alt+P` | Turn visible whitespace on/off |
| `Alt+X` | Hide or unhide the help lines |
| `Ctrl+L` | Refresh the screen |

### 4.10 Examples

```bash
nano notes.txt        # Open (or create) notes.txt in nano
# Inside nano:
#   Ctrl+W  -> search for a word
#   Ctrl+K  -> cut the current line
#   Ctrl+U  -> paste it elsewhere
#   Ctrl+O then Enter -> save the file
#   Ctrl+X  -> exit nano
```

---

## 5. The `vi` Editor

*Reference: [https://vimhelp.org/](https://vimhelp.org/)*

### 5.1 Concept

`vi` (Visual Editor) is a powerful **modal** terminal editor — meaning it has distinct **modes**, and keystrokes behave differently depending on the current mode. This is the key conceptual difference from `nano`, which has only one editing mode.

### 5.2 Modes in vi

| Mode | How to Enter | Purpose |
|---|---|---|
| **Command Mode** | Default mode / press `Esc` from any other mode | Navigate, delete, copy, run commands |
| **Insert Mode** | `i`, `o`, `a` (or `I`, `O`, `A`) from Command Mode | Type/insert new text |
| **Ex (Last-line) Mode** | `:` from Command Mode | Run `ex`-style commands (save, quit, search-replace) |

**Mode transition diagram (conceptual):**

```
        i / o / a / I / O / A
Command Mode  ────────────────►  Insert Mode
     ▲                                │
     │            Esc                │
     └────────────────────────────────┘

Command Mode  ───────:───────►  Ex Mode
     ▲                                │
     │            Enter/Esc          │
     └────────────────────────────────┘
```

### 5.3 Exiting vi

| Command | Meaning |
|---|---|
| `:w` | Write out (save) the file |
| `:x` | Write out and quit |
| `:wq` | Write out and quit |
| `:q` | Quit (only if changes are already saved) |
| `:q!` | Ignore changes and quit forcibly |

### 5.4 Basic Movement

*(Executed in Command Mode)*

| Key(s) | Direction |
|---|---|
| `l` / `space` / Right-arrow | Move right |
| `h` / `Backspace` / Left-arrow | Move left |
| `k` / Up-arrow | Move up |
| `j` / `Return` / Down-arrow | Move down |

### 5.5 Advanced Movement (Command Mode)

| Command | Meaning |
|---|---|
| `0` | Start of the current line |
| `$` | End of the current line |
| `w` | Beginning of next word |
| `b` | Beginning of preceding word |
| `:0` or `1G` | First line of the file |
| `:n` or `nG` | Go to nth line of the file |
| `:$` or `G` | Last line of the file |

### 5.6 Screen Manipulation

| Command | Meaning |
|---|---|
| `Ctrl+F` | Scroll forward one screen |
| `Ctrl+B` | Scroll backward one screen |
| `Ctrl+D` | Scroll down half screen |
| `Ctrl+U` | Scroll up half screen |
| `Ctrl+L` | Redraw screen |
| `Ctrl+R` | Redraw screen, removing deleted content |

### 5.7 Changing Text

| Command | Meaning |
|---|---|
| `r` | Replace single character under cursor |
| `R` | Replace characters from cursor until `Esc` |
| `cw` | Change word under cursor, from current character till `Esc` |
| `cNw` | Change **N** words, from current character till `Esc` |
| `C` | Change characters in current line till `Esc` |
| `cc` | Change (replace) the entire current line till `Esc` |
| `Ncc` | Change next **N** lines, starting from current line, till `Esc` |

### 5.8 Deleting Text

| Command | Meaning |
|---|---|
| `x` | Delete single character under cursor |
| `Nx` | Delete **N** characters from cursor |
| `dw` | Delete one word, from character under cursor |
| `dNw` | Delete **N** words, from character under cursor |
| `D` | Delete rest of the line, from character under cursor |
| `dd` | Delete current line |
| `Ndd` | Delete next **N** lines, starting from current line |

### 5.9 Copy–Paste Text

| Command | Meaning |
|---|---|
| `yy` | Copy (yank) current line into buffer |
| `Nyy` | Copy next **N** lines (including current) into buffer |
| `p` | Paste buffer contents after current line |
| `u` | Undo previous action |

### 5.10 Searching Text

| Command | Meaning |
|---|---|
| `/string` | Search forward for `string` |
| `?string` | Search backward for `string` |
| `n` | Move cursor to next occurrence of the search string |
| `N` | Move cursor to previous occurrence of the search string |
| `:se nu` | Turn on (set) line numbers |
| `:se nonu` | Turn off (unset) line numbers |

### 5.11 Examples

```bash
vi report.txt
```
Inside `vi`:
```
Esc          " Ensure you are in Command Mode
:se nu       " Turn on line numbers
5G           " Jump to line 5
dd           " Delete line 5
i            " Enter Insert mode
Hello World  " Type text
Esc          " Return to Command mode
/error       " Search forward for the word 'error'
n            " Jump to next match
:wq          " Save and quit
```

---

## 6. The `emacs` Editor

*Reference: [https://www.gnu.org/software/emacs/refcards/pdf/refcard.pdf](https://www.gnu.org/software/emacs/refcards/pdf/refcard.pdf)*

### 6.1 Concept & Notation

`emacs` is a highly extensible, **non-modal** text editor (unlike `vi`, you don't switch between insert/command modes — you type text directly, and commands are invoked with key combinations).

**Key notation used by emacs:**

| Notation | Meaning |
|---|---|
| `C-x` | Hold `Ctrl` + press `x` |
| `M-x` | Hold `Alt` + press `x` (M = "Meta" key) |

### 6.2 Moving Around

| Command | Meaning |
|---|---|
| `C-p` | Move up one line |
| `C-b` | Move left one character |
| `C-f` | Move right one character |
| `C-n` | Move down one line |
| `C-a` | Go to beginning of current line |
| `C-e` | Go to end of current line |
| `C-v` | Move forward one screen |
| `M-<` | Move to first line of the file |
| `M-b` | Move left to previous word |
| `M-f` | Move right to next word |
| `M->` | Move to last line of the file |
| `M-a` | Move to beginning of current sentence |
| `M-e` | Move to end of current sentence |
| `M-v` | Move back one screen |

### 6.3 Exiting emacs

| Command | Meaning |
|---|---|
| `C-x C-s` | Save buffer to file |
| `C-z` | Exit emacs but keep it running (suspend) |
| `C-x C-c` | Exit emacs and stop it completely |

### 6.4 Copy–Paste

| Command | Meaning |
|---|---|
| `M-Backspace` | Cut the word before cursor |
| `M-d` | Cut the word after cursor |
| `C-k` | Cut from cursor to end of line |
| `M-k` | Cut from cursor to end of the sentence |
| `C-y` | Paste ("yank") the content at the cursor |

### 6.5 Searching Text

| Command | Meaning |
|---|---|
| `C-s` | Search forward |
| `C-r` | Search backward |
| `M-x` | Replace string (invokes command prompt; e.g., `replace-string`) |

### 6.6 Examples

```bash
emacs notes.txt
```
Inside `emacs`:
```
C-s error         " Search forward for the word 'error'
C-k               " Cut the current line from cursor to end
C-y               " Paste it back elsewhere
C-x C-s           " Save the file
C-x C-c           " Exit emacs
```

---

## 7. Comparison of All Editors

| Feature | `ed` | `ex` | `nano` | `vi` | `emacs` |
|---|---|---|---|---|---|
| **Type** | Line editor | Line editor | Terminal (screen) editor | Terminal (screen) editor | Terminal (screen) editor |
| **Modal?** | No (command-based) | No | No | **Yes** (Command/Insert/Ex) | No |
| **Beginner-Friendly** | ✗ Difficult | ✗ Difficult | ✅ Very easy | ✗ Steep learning curve | ✗ Steep learning curve |
| **On-screen Help** | ✗ None | ✗ None | ✅ Bottom shortcut bar | ✗ None (needs `:help`) | ✅ Menu bar / `C-h` help |
| **Save Command** | `w` | `:w` | `Ctrl+O` | `:w` | `C-x C-s` |
| **Quit Command** | `q` | `:q` | `Ctrl+X` | `:q` / `:q!` | `C-x C-c` |
| **Search** | `/RE/`, `s` | `s` | `Ctrl+W` | `/string`, `?string` | `C-s`, `C-r` |
| **Undo** | `u` | `u` | `Alt+U` | `u` | (varies by config) |
| **Best Use Case** | Scripted/batch edits | Scripting basis for vi | Quick edits, beginners | Fast editing once mastered; ubiquitous on servers | Highly customizable, programmer-focused editing |

---

## 8. Summary

- **Editors are classified into 4 categories**: Line Editors (`ed`), Terminal Editors (`vi`, `emacs`, `pico`, `nano`), GUI Editors (gedit, Sublime, Atom, etc.), and IDEs (Eclipse, NetBeans).
- All editors share **common core features**: navigation, insert/replace/delete, cut-copy-paste, search-replace, syntax highlighting, macros, and plugin support.
- **`ed`** is the original UNIX line editor; it works on one line/buffer at a time using an `[address]command[params]` syntax — foundational but has no visual screen.
- **`ex`** is closely related to `ed` and forms the backbone of every `:`-prefixed command inside `vi`.
- **`nano`** is the most beginner-friendly full-screen editor, with all shortcuts visible on-screen (`Ctrl`/`Alt` based), making it ideal for quick terminal edits.
- **`vi`** is a powerful **modal editor** with three modes — **Command**, **Insert**, and **Ex (last-line)** — and is nearly universal across UNIX/Linux systems, making it essential to learn for server administration.
- **`emacs`** is a **non-modal**, highly extensible editor using `C-x` (Ctrl) and `M-x` (Alt/Meta) key combinations, popular for its deep customizability and programming support.
- **Recommended learning order for exam & practical use:**
  1. Understand editor categories and common features (conceptual foundation)
  2. Learn `nano` first (simplest, gentlest introduction to terminal editing)
  3. Learn `vi`/`vim` next (modal concept — most widely used in real-world Linux systems)
  4. Learn `ed`/`ex` (to understand the historical/scripting foundation of `vi`)
  5. Learn `emacs` (as an alternative, highly customizable power-editor)

---

*End of Notes — Week 2, Lectures 1–3: Command Line Editors*
