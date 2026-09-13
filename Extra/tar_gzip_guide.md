# The `tar` and `gzip` Commands — Complete Guide

`tar` and `gzip` are two of the most commonly used Linux utilities for **archiving** and **compressing** files. They're often used together — `tar` bundles multiple files/folders into a single archive, and `gzip` compresses that archive to save space.

---

# PART 1: `tar` (Tape Archive)

## 1. What `tar` Does

`tar` combines multiple files and directories into a single archive file (commonly `.tar`). By itself, `tar` does **not compress** — it just bundles files together. Compression is usually added via `gzip`, `bzip2`, or `xz`.

## 2. Basic Syntax

```bash
tar [OPTIONS] [ARCHIVE_NAME] [FILES/DIRS...]
```

## 3. Core Options

| Option | Long form | Description |
|--------|-----------|--------------|
| `-c` | `--create` | Create a new archive |
| `-x` | `--extract` | Extract files from an archive |
| `-t` | `--list` | List contents of an archive without extracting |
| `-f` | `--file` | Specify the archive filename (almost always required) |
| `-v` | `--verbose` | Show files being processed |
| `-z` | `--gzip` | Compress/decompress using gzip (`.tar.gz`) |
| `-j` | `--bzip2` | Compress/decompress using bzip2 (`.tar.bz2`) |
| `-J` | `--xz` | Compress/decompress using xz (`.tar.xz`) |
| `-r` | `--append` | Append files to an existing archive |
| `-u` | `--update` | Append files newer than those in archive |
| `-d` | `--diff/--compare` | Compare archive with filesystem |
| `--delete` | | Delete a file from an archive |
| `-C DIR` | `--directory` | Change to directory before performing action |
| `-p` | `--preserve-permissions` | Preserve file permissions |
| `--exclude=PATTERN` | | Exclude files matching pattern |

## 4. Creating an Archive

```bash
tar -cvf archive.tar file1 file2 folder/
```
- `-c` create
- `-v` verbose (show progress)
- `-f` filename follows

## 5. Creating a Compressed Archive (`.tar.gz`)

```bash
tar -czvf archive.tar.gz folder/
```
This is the most common command you'll use — creates a gzip-compressed tarball.

Other compressions:
```bash
tar -cjvf archive.tar.bz2 folder/   # bzip2 (better compression, slower)
tar -cJvf archive.tar.xz folder/    # xz (best compression, slowest)
```

## 6. Extracting an Archive

```bash
tar -xvf archive.tar
```

Extracting a `.tar.gz`:
```bash
tar -xzvf archive.tar.gz
```

Extracting a `.tar.bz2`:
```bash
tar -xjvf archive.tar.bz2
```

Extracting a `.tar.xz`:
```bash
tar -xJvf archive.tar.xz
```

> **Note:** Modern `tar` (GNU tar) can usually auto-detect compression type, so `tar -xvf archive.tar.gz` often works even without `-z`.

## 7. Extracting to a Specific Directory

```bash
tar -xzvf archive.tar.gz -C /path/to/destination/
```
The destination directory must already exist.

## 8. Listing Archive Contents (Without Extracting)

```bash
tar -tvf archive.tar
tar -tzvf archive.tar.gz
```

## 9. Extracting Only Specific Files

```bash
tar -xzvf archive.tar.gz file1.txt folder/file2.txt
```

## 10. Excluding Files While Creating an Archive

```bash
tar -czvf archive.tar.gz --exclude='*.log' --exclude='node_modules' folder/
```

## 11. Appending Files to an Existing Archive

```bash
tar -rvf archive.tar newfile.txt
```
> Note: You **cannot** append to a compressed archive (`.tar.gz`) directly — you'd need to decompress, append, then recompress.

## 12. Updating an Archive (Only Newer Files)

```bash
tar -uvf archive.tar file1.txt
```

## 13. Comparing Archive Contents to Disk

```bash
tar -dvf archive.tar
```

## 14. Deleting a File From an Archive

```bash
tar --delete -f archive.tar file1.txt
```
> Only works on uncompressed `.tar` archives.

## 15. Preserving Permissions and Ownership

```bash
tar -cpvf archive.tar folder/
```
Useful for backups where original permissions matter.

## 16. Practical `tar` Examples

**Backup a directory:**
```bash
tar -czvf backup_$(date +%F).tar.gz /home/user/documents
```

**Extract into current directory:**
```bash
tar -xzvf project.tar.gz
```

**Create archive excluding `.git` folder:**
```bash
tar -czvf project.tar.gz --exclude='.git' project/
```

**Check archive size before extracting:**
```bash
tar -tzvf archive.tar.gz | wc -l
```

---

# PART 2: `gzip` (GNU Zip)

## 17. What `gzip` Does

`gzip` compresses a **single file** — it does not bundle multiple files like `tar`. When you compress a file with `gzip`, the original is replaced by a `.gz` version (unless you use `-k`).

## 18. Basic Syntax

```bash
gzip [OPTIONS] FILE
```

## 19. Core Options

| Option | Long form | Description |
|--------|-----------|--------------|
| `-d` | `--decompress` | Decompress a `.gz` file (same as `gunzip`) |
| `-k` | `--keep` | Keep the original file (don't delete after compressing) |
| `-v` | `--verbose` | Show compression ratio/details |
| `-r` | `--recursive` | Compress files recursively in a directory |
| `-1` to `-9` | | Compression level (1 = fastest/least compression, 9 = slowest/best compression) |
| `-c` | `--stdout` | Write output to stdout (don't touch original file) |
| `-l` | `--list` | List compression stats of a `.gz` file |
| `-t` | `--test` | Test integrity of a compressed file |
| `-f` | `--force` | Force compression even if output file exists |

## 20. Compressing a File

```bash
gzip file.txt
# creates file.txt.gz, deletes original file.txt
```

## 21. Keeping the Original File

```bash
gzip -k file.txt
# creates file.txt.gz, keeps file.txt too
```

## 22. Decompressing a File

```bash
gzip -d file.txt.gz
# or
gunzip file.txt.gz
```

## 23. Setting Compression Level

```bash
gzip -9 file.txt      # maximum compression (slower)
gzip -1 file.txt      # fastest compression (larger file)
```

## 24. Viewing Compression Info

```bash
gzip -l file.txt.gz
#          compressed        uncompressed  ratio uncompressed_name
#              1234              5678      78.3% file.txt
```

## 25. Testing a `.gz` File's Integrity

```bash
gzip -t file.txt.gz
```
No output means the file is fine; errors indicate corruption.

## 26. Compressing Multiple Files (Each Separately)

```bash
gzip file1.txt file2.txt file3.txt
# creates file1.txt.gz, file2.txt.gz, file3.txt.gz individually
```
> Note: `gzip` does **not** combine files into one archive — for that, you need `tar` first, then `gzip` the resulting `.tar`.

## 27. Compressing to stdout (Without Modifying the Original)

```bash
gzip -c file.txt > file.txt.gz
```

## 28. Recursively Compressing a Directory's Files

```bash
gzip -r folder/
```
Compresses every file inside `folder/` individually (does not bundle them).

---

# PART 3: `tar` + `gzip` Together

## 29. Why They're Used Together

- `tar` = bundles many files/folders into **one** file.
- `gzip` = compresses **one** file.
- Combined: `tar` bundles everything → `gzip` compresses the bundle → result is a single compressed archive (`.tar.gz` / `.tgz`).

## 30. Manual Two-Step Process (Without `-z`)

```bash
tar -cvf archive.tar folder/
gzip archive.tar
# result: archive.tar.gz
```

## 31. One-Step Process (Using `-z`)

```bash
tar -czvf archive.tar.gz folder/
```
This is equivalent to the two-step process above but done in a single command — the standard, preferred method.

## 32. Extracting a `.tar.gz` in Two Steps

```bash
gunzip archive.tar.gz     # produces archive.tar
tar -xvf archive.tar
```

## 33. Extracting a `.tar.gz` in One Step

```bash
tar -xzvf archive.tar.gz
```

---

## 34. Quick Reference Cheat Sheet

### tar
```bash
tar -cvf archive.tar files/           # create archive
tar -czvf archive.tar.gz files/       # create gzip-compressed archive
tar -xvf archive.tar                  # extract archive
tar -xzvf archive.tar.gz              # extract gzip-compressed archive
tar -tvf archive.tar                  # list contents
tar -xzvf archive.tar.gz -C dir/      # extract to specific directory
tar -czvf a.tar.gz --exclude='*.log' folder/   # exclude files
tar -rvf archive.tar newfile.txt      # append file (uncompressed only)
tar --delete -f archive.tar file.txt  # delete file from archive
```

### gzip
```bash
gzip file.txt              # compress (deletes original)
gzip -k file.txt           # compress, keep original
gzip -d file.txt.gz        # decompress
gunzip file.txt.gz         # decompress (alternative)
gzip -9 file.txt           # max compression
gzip -l file.txt.gz        # show compression stats
gzip -t file.txt.gz        # test integrity
gzip -r folder/            # recursively compress files in a folder
```

---

## 35. Common Pitfalls

1. **Forgetting `-f`** — `tar` almost always needs `-f filename`, or it may try to read/write from a tape device.
2. **Assuming `gzip` archives multiple files** — it doesn't; it only compresses one file at a time. Use `tar` first.
3. **Trying to append to a `.tar.gz`** — you can't append directly to a compressed archive; decompress, modify, recompress.
4. **Losing the original file** — `gzip` deletes the source file by default; use `-k` if you want to keep it.
5. **Wrong extraction flag** — using `-j` (bzip2) on a `.tar.gz` file (gzip) will fail; match the flag to the compression type, or just let modern `tar` auto-detect.
6. **Overwriting files on extract** — `tar` will silently overwrite existing files with the same name during extraction.

---

## 36. Related Commands

- `bzip2` / `bunzip2` — alternative compression, better ratio than gzip, slower
- `xz` / `unxz` — best compression ratio, slowest, common for `.tar.xz`
- `zip` / `unzip` — cross-platform archive format (bundles + compresses, unlike tar+gzip)
- `7z` — high-ratio compression tool (via `p7zip`)
- `zcat` — view contents of a `.gz` file without fully decompressing it

---

## 37. Summary

`tar` handles **bundling** multiple files into one archive, while `gzip` handles **compression** of a single file. Combined via `tar -czvf` / `tar -xzvf`, they form the standard way to package and compress files on Linux — used everywhere from software distribution to backups to log rotation.
