# Week 3 — Lecture 3 & 4: Software Management (Package Management Systems)

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Checking System Information](#2-checking-system-information)
   - 2.1 [Check Type of Operating System](#21-check-type-of-operating-system)
   - 2.2 [Check Type of Kernel and Architecture](#22-check-type-of-kernel-and-architecture)
3. [Need for a Package Manager](#3-need-for-a-package-manager)
4. [Package Types](#4-package-types)
   - 4.1 [RPM-Based Distributions](#41-rpm-based-distributions)
   - 4.2 [DEB-Based Distributions](#42-deb-based-distributions)
5. [Package Architectures](#5-package-architectures)
6. [Package Management Tools](#6-package-management-tools)
   - 6.1 [Tools for RPM-Based Systems](#61-tools-for-rpm-based-systems)
   - 6.2 [Tools for DEB-Based Systems](#62-tools-for-deb-based-systems)
7. [Package Naming Convention](#7-package-naming-convention)
8. [Package Priorities](#8-package-priorities)
9. [Package Sections](#9-package-sections)
10. [Checksums](#10-checksums)
11. [Permissions for Package Management](#11-permissions-for-package-management)
12. [Package-Related Log Files — `/var/log`](#12-package-related-log-files--varlog)
13. [Package Management in Ubuntu using `apt`](#13-package-management-in-ubuntu-using-apt)
    - 13.1 [Configuration Files for `apt`](#131-configuration-files-for-apt)
    - 13.2 [Inquiring the Package Database](#132-inquiring-the-package-database)
    - 13.3 [Installing / Updating Packages](#133-installing--updating-packages)
    - 13.4 [Removing / Cleaning Up Packages](#134-removing--cleaning-up-packages)
14. [Package Management in Ubuntu using `dpkg`](#14-package-management-in-ubuntu-using-dpkg)
    - 14.1 [Configuration Files for `dpkg`](#141-configuration-files-for-dpkg)
    - 14.2 [Using `dpkg` to Query Packages](#142-using-dpkg-to-query-packages)
    - 14.3 [Advanced Querying with `dpkg-query`](#143-advanced-querying-with-dpkg-query)
    - 14.4 [Installing a `.deb` Package with `dpkg`](#144-installing-a-deb-package-with-dpkg)
15. [How to Retrieve Package Properties via Commands](#15-how-to-retrieve-package-properties-via-commands)
    - 15.1 [Retrieving Version](#151-retrieving-version)
    - 15.2 [Retrieving Architecture](#152-retrieving-architecture)
    - 15.3 [Retrieving Priority](#153-retrieving-priority)
    - 15.4 [Retrieving Section](#154-retrieving-section)
    - 15.5 [Retrieving Dependencies](#155-retrieving-dependencies)
    - 15.6 [Retrieving Installed Size / Download Size](#156-retrieving-installed-size--download-size)
    - 15.7 [Retrieving Installation Status](#157-retrieving-installation-status)
    - 15.8 [Retrieving Checksums (MD5/SHA256) of a Package](#158-retrieving-checksums-md5sha256-of-a-package)
    - 15.9 [Quick Reference: Property → Command](#159-quick-reference-property--command)
16. [Consolidated Command Reference Table](#16-consolidated-command-reference-table)
17. [Summary](#17-summary)

---

## 1. Introduction

Modern Linux distributions organize software into **packages** — pre-compiled bundles of files, metadata, and installation instructions. A **package management system** is the collection of tools and databases used to install, update, remove, and track software on a system in a controlled, reliable, and automated way. This lecture covers the theory of package management, the major package formats and tools, and the practical use of `apt` and `dpkg` on Ubuntu/Debian-based systems.

---

## 2. Checking System Information

**Concept:**
Before managing packages, it is useful to know exactly **which operating system, kernel version, and CPU architecture** a machine is running — since the correct package format (RPM vs DEB) and the correct package architecture (e.g. `amd64` vs `arm`) depend entirely on this information.

### 2.1 Check Type of Operating System

**Concept:**
Linux provides several built-in commands and files that reveal the distribution name, version, and codename of the operating system currently installed.

**Syntax:**
```bash
cat /etc/os-release
lsb_release -a
hostnamectl
```

**Explanation:**
- `/etc/os-release` is a standard file present on nearly all modern Linux distributions containing OS name, version, and ID fields.
- `lsb_release -a` prints Linux Standard Base information (distributor, description, release, codename).
- `hostnamectl` (systemd-based systems) also displays the OS name along with hostname and kernel details in one summary.

**Examples:**
```bash
# Example 1: View OS release information file
cat /etc/os-release

# Example 2: View LSB-formatted OS details (Distributor ID, Release, Codename)
lsb_release -a

# Example 3: View OS name as part of a full system summary
hostnamectl
```

**Sample Output (for Ubuntu):**
```
NAME="Ubuntu"
VERSION="22.04.3 LTS (Jammy Jellyfish)"
ID=ubuntu
VERSION_ID="22.04"
```

---

### 2.2 Check Type of Kernel and Architecture

**Concept:**
The `uname` command reports low-level system information, including the kernel name, kernel release version, and the machine's hardware/CPU architecture. This is important because it tells you which **architecture-specific package** (e.g. `amd64`, `arm`) you must download.

**Syntax:**
```bash
uname -a
uname -r
uname -m
arch
```

**Explanation:**
- `uname -a` → prints **all** available system information in one line (kernel name, hostname, kernel release, kernel version, machine hardware name, OS).
- `uname -r` → prints only the **kernel release** version.
- `uname -m` → prints only the **machine hardware architecture** (e.g. `x86_64`).
- `arch` → shorthand command that also prints the machine's hardware architecture.

**Examples:**
```bash
# Example 1: View complete kernel and system information
uname -a

# Example 2: View only the kernel version/release
uname -r

# Example 3: View only the CPU architecture
uname -m

# Example 4: Alternative shorthand for checking architecture
arch
```

**Sample Output:**
```
$ uname -a
Linux ubuntu-server 5.15.0-91-generic #101-Ubuntu SMP x86_64 GNU/Linux

$ uname -m
x86_64
```

---

## 3. Need for a Package Manager

**Concept:**
A package manager solves the problems of manually downloading, compiling, and tracking software by providing a structured system for handling software throughout its lifecycle.

A package manager provides the following core functions:

| Function | Description |
|----------|-------------|
| Install / Update / Remove | Core tools to manage the lifecycle of software on a system |
| Network installation | Ability to install new or updated software from remote repositories across a network |
| Package ↔ File lookup | Look up which package a file belongs to, and which files belong to a package (bidirectional lookup) |
| Package database | Maintains a database of all packages installed on the system, including their versions |
| Dependency checking | Automatically resolves and installs other packages that a given package depends on |
| Signature verification | Verifies the authenticity and integrity of packages using cryptographic signatures |
| Package building tools | Provides tools to build/create new packages |

---

## 4. Package Types

**Concept:**
Linux distributions are generally grouped into two major families based on the packaging format they use: **RPM** and **DEB**. Each format has its own file structure, tools, and set of distributions that use it.

### 4.1 RPM-Based Distributions

RPM (**Red Hat Package Manager**) format is used by:

- Red Hat
    - CentOS
    - Fedora
    - Oracle Linux
- SUSE Enterprise Linux
    - openSUSE

### 4.2 DEB-Based Distributions

DEB (Debian package format) is used by:

- Debian
    - Ubuntu
        - Mint
    - Knoppix

---

## 5. Package Architectures

**Concept:**
Each package is built for a specific **CPU architecture**, since compiled binaries are architecture-dependent. The architecture is usually indicated in the package filename.

| Architecture Label | Meaning |
|----------------------|---------|
| `amd64` / `x86_64` | 64-bit Intel/AMD processors |
| `i386` / `x86` | 32-bit Intel/AMD processors |
| `arm` | ARM-based processors |
| `ppc64el` | OpenPOWER (64-bit little-endian PowerPC) processors |
| `all` / `noarch` / `src` | Architecture-independent packages, or source packages |

> **Tip:** Use `uname -m` or `arch` (see Section 2.2) to check your machine's architecture before downloading a package manually.

---

## 6. Package Management Tools

**Concept:**
Each package format (RPM or DEB) has its own ecosystem of low-level and high-level tools. Low-level tools work directly with individual package files; high-level tools manage repositories, dependencies, and updates automatically.

### 6.1 Tools for RPM-Based Systems

| Tool | Full Name | Role |
|------|-----------|------|
| `rpm` | Red Hat Package Manager | Low-level tool: installs, queries, verifies, and removes individual `.rpm` files |
| `yum` | Yellowdog Updater Modifier | High-level tool: manages repositories, dependencies, and updates (used in older Red Hat/CentOS) |
| `dnf` | Dandified YUM | Next-generation replacement for `yum`, with improved dependency resolution (used in modern Fedora/RHEL/CentOS) |

### 6.2 Tools for DEB-Based Systems

| Tool | Role |
|------|------|
| `dpkg` | Low-level tool: installs, queries, and removes individual `.deb` files |
| `dpkg-deb` | Utility to build and manipulate `.deb` package archives directly |
| `apt` (Advanced Package Tool) | High-level tool: manages repositories, dependency resolution, and updates |
| `aptitude` | Alternative high-level, interactive front-end for package management |
| `synaptic` | Graphical (GUI) front-end for package management |

**Tool Hierarchy (as per the diagram):**

```
Package Type
├── RPM
│   └── Yellowdog Updater Modifier (yum)
│       ├── Red Hat Package Manager (rpm)
│       └── Dandified YUM (dnf)
└── DEB
    ├── synaptic ─┐
    ├── aptitude ─┴─ Advanced Package Tool (apt)
    └── dpkg
        └── dpkg-deb
```

---

## 7. Package Naming Convention

**Concept:**
Package filenames follow a standardized naming pattern that encodes the package name, version, release/revision number, and target architecture. Understanding this naming pattern helps identify exactly what a package file contains before installing it.

| Format Type | Naming Pattern |
|-------------|------------------|
| RPM | `package-version-release.architecture.rpm` |
| DEB | `package_version-revision_architecture.deb` |

**Examples:**
```
# RPM example 1
httpd-2.4.6-97.el7.centos.x86_64.rpm

# RPM example 2
vim-enhanced-8.0.1763-19.el8.x86_64.rpm

# DEB example 1
firefox_115.0.2-1_amd64.deb

# DEB example 2
google-chrome-stable_115.0.5790.170-1_amd64.deb
```

**Breaking down a DEB filename — `firefox_115.0.2-1_amd64.deb`:**

| Component | Value | Meaning |
|-----------|-------|---------|
| Package name | `firefox` | Name of the software |
| Version | `115.0.2` | Upstream software version |
| Revision | `1` | Debian packaging revision number |
| Architecture | `amd64` | Target CPU architecture |
| Extension | `.deb` | Debian package file format |

---

## 8. Package Priorities

**Concept:**
Debian-based systems classify packages into **priority levels**, which indicate how essential a package is to the correct and complete functioning of the system. This helps administrators decide what is safe to remove versus what should always remain installed.

| Priority | Description |
|----------|-------------|
| `required` | Essential to the proper functioning of the system; should never be removed |
| `important` | Provides functionality that enables the system to run well |
| `standard` | Included in a standard system installation |
| `optional` | Can be omitted if there is not enough storage available |
| `extra` | Could conflict with packages of higher priority; has specialized requirements; install only if specifically needed |

---

## 9. Package Sections

**Concept:**
Packages are further organized into **sections** (categories) based on their purpose, making it easier to browse and locate software of a particular type. The Ubuntu package repository organizes packages into sections such as:

> Administration Utilities, Mono/CLI, Communication Programs, Databases, Debug packages, Development, Documentation, Editors, Electronics, Embedded software, Fonts, Games, GNOME, GNU R, GNUstep, Graphics, Haskell, Web Servers, Interpreters, Java, KDE, Kernels, Library development, Libraries, Lisp, Language packs, Mail, Mathematics, Miscellaneous, Network, Newsgroups, OCaml, Perl, PHP, Python, Ruby, Science, Shells, Sound, TeX, Text Processing, Translations, Utilities, Version Control Systems, Video, Web Software, X Window System software, Xfce, Zope/Plone Framework

**Reference:** [https://packages.ubuntu.com/focal](https://packages.ubuntu.com/focal)

---

## 10. Checksums

**Concept:**
A **checksum** (or hash) is a fixed-length value calculated from a file's contents, used to verify the file's **integrity** — confirming that a package has not been corrupted or tampered with during download or transfer.

| Algorithm | Output Length |
|-----------|-----------------|
| `md5sum` | 128-bit |
| `SHA1` | 160-bit |
| `SHA256` | 256-bit |

**Example:**
```bash
# Verify integrity of a downloaded package using md5sum
md5sum package.deb

# Verify integrity using SHA256 (stronger, recommended)
sha256sum package.deb

# Verify integrity using SHA1
sha1sum package.deb

# Compare a downloaded checksum against a published value
echo "d41d8cd98f00b204e9800998ecf8427e  package.deb" | md5sum -c -
```

---

## 11. Permissions for Package Management

**Concept:**
Installing, upgrading, or removing packages modifies core parts of the system, so these operations are **restricted to privileged users**. Only users listed as **sudoers** (users authorized to use `sudo`) can install, upgrade, or remove packages.

| Item | Path |
|------|------|
| Sudoers configuration file | `/etc/sudoers` |

**Accessing the sudoers file:**
The sudoers file defines which users/groups have administrative (`sudo`) privileges. It should always be edited using the dedicated `visudo` command (never a plain text editor), since `visudo` checks for syntax errors before saving — a broken sudoers file can lock every user out of `sudo`.

**Syntax:**
```bash
sudo cat /etc/sudoers
sudo visudo
```

**Examples:**
```bash
# Example 1: A regular user must prefix package commands with sudo
sudo apt-get install package

# Example 2: Attempting without sudo results in a permission error
apt-get install nginx
# Output: E: Could not open lock file /var/lib/dpkg/lock-frontend - Permission denied

# Example 3: View the contents of the sudoers file (read-only check)
sudo cat /etc/sudoers

# Example 4: Safely edit the sudoers file with syntax checking
sudo visudo
```

---

## 12. Package-Related Log Files — `/var/log`

**Concept:**
Every package operation (install, upgrade, remove, purge) is recorded in log files under `/var/log`. These logs are essential for **troubleshooting** failed installs, **auditing** what was changed and when, and **reviewing** package history.

| Log File | Path | Contents |
|----------|------|-----------|
| APT history log | `/var/log/apt/history.log` | High-level record of `apt` actions (install/remove/upgrade) with timestamps |
| APT terminal log | `/var/log/apt/term.log` | Raw terminal output produced during `apt` operations |
| dpkg log | `/var/log/dpkg.log` | Low-level record of every `dpkg` package state change |

**Syntax:**
```bash
cd /var/log
cat /var/log/apt/history.log
cat /var/log/dpkg.log
```

**Examples:**
```bash
# Example 1: Navigate to the main system log directory
cd /var/log

# Example 2: View a high-level history of installs/removals/upgrades performed via apt
cat /var/log/apt/history.log

# Example 3: View the low-level dpkg log of every package state change
cat /var/log/dpkg.log

# Example 4: Search the dpkg log for when a specific package was installed
grep "install nginx" /var/log/dpkg.log
```

---

## 13. Package Management in Ubuntu using `apt`

**Concept:**
`apt` (**Advanced Package Tool**) is the primary high-level package management tool on Debian/Ubuntu systems. It automates downloading, dependency resolution, and configuration of software packages from configured repositories.

### 13.1 Configuration Files for `apt`

| Item | Path |
|------|------|
| Main configuration directory | `/etc/apt` |
| Main sources file | `/etc/apt/sources.list` |
| Additional sources folder | `/etc/apt/sources.list.d` |

**Explanation:**
- `/etc/apt` is the root directory holding all of `apt`'s configuration.
- `sources.list` is a plain-text file listing the repository URLs `apt` uses to find packages (mirrors, main/universe/restricted/multiverse components, etc.).
- `sources.list.d` is a folder where **additional** repository definitions can be placed as separate `.list` files (commonly added by third-party software installers), instead of editing the main file directly.

**Syntax:**
```bash
cd /etc/apt
cat sources.list
cd sources.list.d
ls
```

**Examples:**
```bash
# Example 1: Move into the main apt configuration directory
cd /etc/apt

# Example 2: View the contents of the main repository list
cat sources.list

# Example 3: Move into the folder holding additional repository files
cd sources.list.d

# Example 4: List all extra repository definition files added by third-party software
ls -l /etc/apt/sources.list.d
```

**Sample `sources.list` entry:**
```
deb http://archive.ubuntu.com/ubuntu jammy main restricted universe multiverse
```

---

### 13.2 Inquiring the Package Database

**Concept:**
Before installing software, `apt-cache` allows you to search and inspect the package database — finding packages by keyword, listing all available packages, filtering packages by name prefix, or viewing detailed metadata about a specific package.

**Syntax and Examples:**

**a) Search packages for a keyword**
```bash
apt-cache search keyword
```
Examples:
```bash
# Example 1: Search for text editors
apt-cache search "text editor"

# Example 2: Search for FTP-related packages
apt-cache search ftp

# Example 3: Search for a package and filter with grep
apt-cache search python3 | grep -i "web framework"
```

**b) List all packages**
```bash
apt-cache pkgnames
```

**Explanation:** `apt-cache pkgnames` prints the name of **every package known to the local package cache** (i.e., every package available from the configured repositories — this list is not limited only to packages currently installed).

Examples:
```bash
# Example 1: List all packages containing "python" in the name
apt-cache pkgnames | grep python

# Example 2: Count the total number of available packages
apt-cache pkgnames | wc -l

# Example 3: Save the full package name list to a file
apt-cache pkgnames > all_packages.txt
```

**c) List packages starting with a given prefix**
```bash
apt-cache pkgnames prefix
```

**Explanation:** Passing a string directly after `pkgnames` filters the result to only package names that **start with** that prefix — useful for narrowing down a large list quickly.

Examples:
```bash
# Example 1: List all packages whose names begin with "nm"
apt-cache pkgnames nm

# Example 2: List all packages whose names begin with "lib"
apt-cache pkgnames lib | head -20

# Example 3: List all packages whose names begin with "python3-"
apt-cache pkgnames python3-
```

**d) Display package records of a package**
```bash
apt-cache show -a package
```
Examples:
```bash
# Example 1: Show full detailed record for the nmap package
apt-cache show nmap

# Example 2: Show all available version records for vim
apt-cache show -a vim

# Example 3: Extract only the Version field from the record
apt-cache show nginx | grep -i version
```

---

### 13.3 Installing / Updating Packages

**Concept:**
These `apt-get` commands handle synchronizing the local package index with remote repositories, upgrading installed software, and installing or reinstalling specific packages.

**a) Synchronize package overview files (update the local package index)**
```bash
apt-get update
```
Examples:
```bash
# Example 1: Standard package index refresh
sudo apt-get update

# Example 2: Update and immediately upgrade in one line using &&
sudo apt-get update && sudo apt-get upgrade

# Example 3: Update quietly, suppressing extra output
sudo apt-get update -qq
```

**b) Upgrade all installed packages**
```bash
apt-get upgrade
```
Examples:
```bash
# Example 1: Upgrade all installed packages
sudo apt-get upgrade

# Example 2: Upgrade without being prompted for confirmation
sudo apt-get upgrade -y

# Example 3: Perform a full upgrade, allowing packages to be added/removed as needed
sudo apt-get full-upgrade
```

**c) Install a package**
```bash
apt-get install package
```
Examples:
```bash
# Example 1: Install a web server
sudo apt-get install nginx

# Example 2: Install multiple packages in a single command
sudo apt-get install git curl vim

# Example 3: Install a specific version of a package
sudo apt-get install nginx=1.18.0-0ubuntu1
```

**d) Reinstall a package**
```bash
apt-get reinstall package
```
Examples:
```bash
# Example 1: Reinstall a package that may have corrupted files
sudo apt-get reinstall curl

# Example 2: Reinstall multiple packages together
sudo apt-get reinstall bash coreutils

# Example 3: Reinstall using the --reinstall flag with install
sudo apt-get install --reinstall openssh-server
```

> **Note:** It is best practice to always run `apt-get update` before `apt-get upgrade` or `apt-get install`, so that the package index reflects the latest available versions.

---

### 13.4 Removing / Cleaning Up Packages

**Concept:**
`apt-get` also provides commands to safely remove packages, clean up automatically-installed dependencies that are no longer needed, and clear cached package files to free up disk space.

**a) Remove packages automatically installed to satisfy a dependency and no longer needed**
```bash
apt-get autoremove
```
Examples:
```bash
# Example 1: Clean up unused dependency packages
sudo apt-get autoremove

# Example 2: Auto-remove without confirmation prompt
sudo apt-get autoremove -y

# Example 3: Auto-remove and also purge configuration files of removed packages
sudo apt-get autoremove --purge
```

**b) Clean the local repository of retrieved package files**
```bash
apt-get clean
```
Examples:
```bash
# Example 1: Free up disk space by clearing the download cache
sudo apt-get clean

# Example 2: Remove only outdated package files (keep latest)
sudo apt-get autoclean

# Example 3: Check cache size before and after cleaning
du -sh /var/cache/apt/archives && sudo apt-get clean && du -sh /var/cache/apt/archives
```

**c) Remove a package** (keeps configuration files)
```bash
apt-get remove package
```
Examples:
```bash
# Example 1: Remove a web server package
sudo apt-get remove nginx

# Example 2: Remove multiple packages at once
sudo apt-get remove apache2 mysql-server

# Example 3: Remove without confirmation prompt
sudo apt-get remove -y nginx
```

**d) Purge package files from the system** (removes package **and** its configuration files)
```bash
apt-get purge package
```
Examples:
```bash
# Example 1: Completely remove nginx and its config
sudo apt-get purge nginx

# Example 2: Purge a package that is already removed but has leftover config files
sudo apt-get purge apache2

# Example 3: Combine remove and purge with autoremove for full cleanup
sudo apt-get purge nginx && sudo apt-get autoremove
```

**`remove` vs `purge` — Key Difference:**

| Command | Removes Program Files | Removes Configuration Files |
|---------|-------------------------|-------------------------------|
| `apt-get remove package` | ✅ Yes | ❌ No (kept) |
| `apt-get purge package` | ✅ Yes | ✅ Yes (deleted) |

---

## 14. Package Management in Ubuntu using `dpkg`

**Concept:**
`dpkg` is the **low-level** package manager underlying `apt` on Debian-based systems. It works directly on individual `.deb` package files and the local package database, but — unlike `apt` — it does **not** automatically resolve or download dependencies from a repository.

### 14.1 Configuration Files for `dpkg`

| Item | Path |
|------|------|
| Files: `arch`, `available`, `status` | `/var/lib/dpkg` |
| Folder: `info` | `/var/lib/dpkg` |

---

### 14.2 Using `dpkg` to Query Packages

**a) List all packages whose names match a pattern**
```bash
dpkg -l pattern
```
Examples:
```bash
# Example 1: List all installed packages with "python" in the name
dpkg -l "*python*"

# Example 2: List all installed packages (no pattern = full list)
dpkg -l

# Example 3: List packages matching "nginx" and check their status flags
dpkg -l "nginx*"
```

**b) List installed files that came from a package**
```bash
dpkg -L package
```
Examples:
```bash
# Example 1: List all files installed by the bash package
dpkg -L bash

# Example 2: List files installed by curl and filter for binaries only
dpkg -L curl | grep bin

# Example 3: Count how many files a package installed
dpkg -L openssh-server | wc -l
```

**c) Report the status of a package**
```bash
dpkg -s package
```
Examples:
```bash
# Example 1: Check status of the SSH server package
dpkg -s openssh-server

# Example 2: Check status and extract only the Version field
dpkg -s vim | grep -i version

# Example 3: Check whether a package is installed (status field)
dpkg -s git | grep -i status
```

**d) Search installed packages for a file**
```bash
dpkg -S pattern
```
Examples:
```bash
# Example 1: Find which package owns the python3 binary
dpkg -S /usr/bin/python3

# Example 2: Find which package a configuration file belongs to
dpkg -S /etc/ssh/sshd_config

# Example 3: Search using a partial filename pattern
dpkg -S "*bashrc*"
```

---

### 14.3 Advanced Querying with `dpkg-query`

**Concept:**
`dpkg-query` is a more flexible querying tool than plain `dpkg`. Its most powerful feature is the `--showformat` (`-f`) option, which lets you print **custom-formatted output** built from specific package metadata fields — instead of the default dense table `dpkg -l` produces. This is extremely useful for generating clean, script-friendly reports (e.g., "package name and its section, one per line").

**Syntax:**
```bash
dpkg-query -W -f='FORMAT_STRING'
```

**Explanation:**
- `-W` / `--show` → lists installed packages (like `dpkg -l`, but designed to work with `-f`).
- `-f='...'` / `--showformat='...'` → defines a custom output template using field placeholders such as `${Package}`, `${Version}`, `${Section}`, `${binary:Package}`, `${Architecture}`, etc.
- `\n` inside the format string inserts a newline after each package's entry, so each package appears on its own line.

**Examples:**

**a) Print each package's Section and Package name**
```bash
dpkg-query -W -f='${Section} ${binary:Package}\n'
```
This prints one line per installed package in the form: `section packagename`.

**b) Page through the output using `less`**
```bash
dpkg-query -W -f='${Section} ${binary:Package}\n' | less
```
Since the full list can be hundreds of lines long, piping to `less` lets you scroll through it comfortably instead of it flooding the terminal.

**c) Sort the output alphabetically by section, then view with `less`**
```bash
dpkg-query -W -f='${Section} ${binary:Package}\n' | sort | less
```
Piping through `sort` groups all packages by section alphabetically before paging through the results — much easier to browse by category.

**d) Filter the output for a specific pattern using `grep`**
```bash
dpkg-query -W -f='${Section} ${binary:Package}\n' | grep pattern
```
Example:
```bash
# Show only packages belonging to the "net" (networking) section
dpkg-query -W -f='${Section} ${binary:Package}\n' | grep net

# Show only packages whose name contains "ssh"
dpkg-query -W -f='${Section} ${binary:Package}\n' | grep ssh
```

**More `dpkg-query -f` examples:**
```bash
# Example: Print package name and version together
dpkg-query -W -f='${binary:Package} ${Version}\n'

# Example: Print package name and installed size, sorted by size
dpkg-query -W -f='${Installed-Size}\t${binary:Package}\n' | sort -n

# Example: Print package name, version, and architecture for a single package
dpkg-query -W -f='${binary:Package} ${Version} ${Architecture}\n' nginx
```

---

### 14.4 Installing a `.deb` Package with `dpkg`

**Syntax:**
```bash
dpkg -i package_version-revision_architecture.deb
```

**Example:**
```bash
sudo dpkg -i googlechrome_115.0.5790.170-1_amd64.deb
```

> **⚠️ Important Notes:**
> - By default, it is recommended to use a **package management tool pointing to a reliable repository** (such as `apt`) rather than installing standalone `.deb` files directly, since `apt` automatically resolves dependencies.
> - **Uninstalling packages using `dpkg` is NOT recommended**, since `dpkg` does not manage dependency relationships the way `apt` does. Use `apt-get remove` / `apt-get purge` instead wherever possible.

---

## 15. How to Retrieve Package Properties via Commands

**Concept:**
Every package carries a set of **metadata properties** — version, architecture, priority, section, dependencies, size, and installation status. These properties are stored in the package database and repository metadata, and can be retrieved using `apt-cache show`, `apt list`, `dpkg -s`, and `dpkg-query`. This section explains **exactly which command reveals which property**, since exam questions often ask "how do you find out X about a package?"

> **Tip:** `apt-cache show package` and `dpkg -s package` both print a structured metadata block (called the package's **control fields**). Nearly every property below is simply one field from that block, so learning to `grep` the right field name — or extract it with `dpkg-query -f`— is the key skill.

---

### 15.1 Retrieving Version

**Syntax:**
```bash
apt-cache show package | grep -i "^Version"
dpkg -s package | grep -i "^Version"
apt list --installed | grep package
dpkg-query -W -f='${Version}\n' package
```

**Examples:**
```bash
# Example 1: Get version of an available (not necessarily installed) package
apt-cache show nginx | grep -i "^Version"

# Example 2: Get version of an already installed package
dpkg -s nginx | grep -i "^Version"

# Example 3: Quick one-line check via apt list
apt list --installed | grep nginx

# Example 4: Extract just the version number using dpkg-query
dpkg-query -W -f='${Version}\n' nginx
```

---

### 15.2 Retrieving Architecture

**Syntax:**
```bash
apt-cache show package | grep -i "^Architecture"
dpkg -s package | grep -i "^Architecture"
dpkg --print-architecture
```

**Examples:**
```bash
# Example 1: Check architecture of an available package
apt-cache show firefox | grep -i "^Architecture"

# Example 2: Check architecture of an installed package
dpkg -s firefox | grep -i "^Architecture"

# Example 3: Check the system's default/native architecture
dpkg --print-architecture

# Example 4: Confirm the machine's hardware architecture directly from the kernel
uname -m
```

---

### 15.3 Retrieving Priority

**Syntax:**
```bash
apt-cache show package | grep -i "^Priority"
dpkg -s package | grep -i "^Priority"
```

**Examples:**
```bash
# Example 1: Check priority of the bash package
apt-cache show bash | grep -i "^Priority"

# Example 2: Check priority of an installed package
dpkg -s coreutils | grep -i "^Priority"

# Example 3: List all "required" priority packages currently installed
dpkg -l | awk '$1=="ii"{print $2}' | xargs -I{} dpkg -s {} 2>/dev/null | grep -B5 "Priority: required"
```

---

### 15.4 Retrieving Section

**Syntax:**
```bash
apt-cache show package | grep -i "^Section"
dpkg -s package | grep -i "^Section"
dpkg-query -W -f='${Section}\n' package
```

**Examples:**
```bash
# Example 1: Check which section vim belongs to
apt-cache show vim | grep -i "^Section"

# Example 2: Check section for an installed package
dpkg -s python3 | grep -i "^Section"

# Example 3: List every installed package along with its section
dpkg-query -W -f='${Section} ${binary:Package}\n' | sort

# Example 4: List only installed packages belonging to the "net" section
dpkg-query -W -f='${Section} ${binary:Package}\n' | grep net
```

---

### 15.5 Retrieving Dependencies

**Syntax:**
```bash
apt-cache show package | grep -i "^Depends"
apt-cache depends package
dpkg -s package | grep -i "^Depends"
```

**Examples:**
```bash
# Example 1: See what a package depends on before installing it
apt-cache depends nginx

# Example 2: See dependencies listed in the repository metadata
apt-cache show nginx | grep -i "^Depends"

# Example 3: See dependencies of an already installed package
dpkg -s nginx | grep -i "^Depends"
```

---

### 15.6 Retrieving Installed Size / Download Size

**Syntax:**
```bash
apt-cache show package | grep -i "Size"
dpkg -s package | grep -i "Installed-Size"
```

**Examples:**
```bash
# Example 1: Check both download size and installed size from repository metadata
apt-cache show mysql-server | grep -i "Size"

# Example 2: Check installed size of a package already on the system
dpkg -s mysql-server | grep -i "Installed-Size"

# Example 3: Simulate an install to preview total download/disk space needed
apt-get install --simulate nginx

# Example 4: List installed packages sorted from smallest to largest
dpkg-query -W -f='${Installed-Size}\t${binary:Package}\n' | sort -n
```

---

### 15.7 Retrieving Installation Status

**Syntax:**
```bash
dpkg -s package | grep -i "^Status"
dpkg -l package
apt list --installed | grep package
```

**Examples:**
```bash
# Example 1: Check full status line (want/error-flag/status) of a package
dpkg -s curl | grep -i "^Status"

# Example 2: Quickly check install status flag from dpkg -l output (ii = installed)
dpkg -l curl

# Example 3: Confirm installation via apt list
apt list --installed 2>/dev/null | grep curl
```

> **Understanding `dpkg -l` status flags:**

| Flag | Meaning |
|------|---------|
| `ii` | Package is fully **installed** |
| `rc` | Package was **removed**, but config files remain |
| `un` | Package is **unknown** (not installed) |
| `iU` | Package is **unpacked but not configured** |

---

### 15.8 Retrieving Checksums (MD5/SHA256) of a Package

**Syntax:**
```bash
md5sum file.deb
sha256sum file.deb
apt-cache show package | grep -i "SHA256\|MD5sum"
```

**Examples:**
```bash
# Example 1: Compute MD5 checksum of a locally downloaded .deb file
md5sum firefox_115.0.2-1_amd64.deb

# Example 2: Compute SHA256 checksum for stronger verification
sha256sum firefox_115.0.2-1_amd64.deb

# Example 3: View the official checksum published in the repository metadata
apt-cache show firefox | grep -i "SHA256"
```

---

### 15.9 Quick Reference: Property → Command

| Property Needed | Best Command to Use |
|------------------|------------------------|
| Operating system type | `cat /etc/os-release` or `lsb_release -a` |
| Kernel version | `uname -r` |
| CPU architecture (machine) | `uname -m` or `arch` |
| Version | `apt-cache show package \| grep Version` or `dpkg -s package` |
| Architecture (of a package) | `apt-cache show package \| grep Architecture` or `dpkg --print-architecture` |
| Priority | `apt-cache show package \| grep Priority` |
| Section (category) | `apt-cache show package \| grep Section` or `dpkg-query -W -f='${Section}\n' package` |
| Dependencies | `apt-cache depends package` |
| Download / Installed Size | `apt-cache show package \| grep Size` |
| Installation status | `dpkg -s package \| grep Status` or `dpkg -l package` |
| Files installed by package | `dpkg -L package` |
| Which package owns a file | `dpkg -S /path/to/file` |
| Checksum of downloaded file | `md5sum` / `sha256sum` on the `.deb` file |
| Custom multi-field report | `dpkg-query -W -f='${Field1} ${Field2}\n'` |
| Full metadata block (all properties at once) | `apt-cache show package` **or** `dpkg -s package` |

> **Exam Tip:** If you are ever asked "how do I find out `<property>` of a package," the safest general answer is:
> ```bash
> apt-cache show package    # for any package in the repository (installed or not)
> dpkg -s package            # for a package already installed on the system
> ```
> Both output a full block of labelled fields (`Package:`, `Version:`, `Architecture:`, `Priority:`, `Section:`, `Depends:`, `Installed-Size:`, `Status:`, etc.) — pipe the output to `grep -i "fieldname"` to isolate exactly what you need, or use `dpkg-query -f='${FieldName}\n'` to extract it cleanly without grep.

---

## 16. Consolidated Command Reference Table

| Command | Tool | Purpose |
|---------|------|---------|
| `cat /etc/os-release` / `lsb_release -a` | core-utils | Check operating system type and version |
| `uname -a` / `uname -r` / `uname -m` / `arch` | core-utils | Check kernel version and CPU architecture |
| `apt-cache search keyword` | apt | Search packages by keyword |
| `apt-cache pkgnames` | apt | List all available package names |
| `apt-cache pkgnames prefix` | apt | List package names starting with a given prefix |
| `apt-cache show -a package` | apt | Show detailed records of a package |
| `apt-get update` | apt | Synchronize local package index with repositories |
| `apt-get upgrade` | apt | Upgrade all installed packages |
| `apt-get install package` | apt | Install a package |
| `apt-get reinstall package` | apt | Reinstall a package |
| `apt-get autoremove` | apt | Remove unneeded auto-installed dependency packages |
| `apt-get clean` | apt | Clean cached package files from local repository |
| `apt-get remove package` | apt | Remove a package (keep config files) |
| `apt-get purge package` | apt | Remove a package and its configuration files |
| `dpkg -l pattern` | dpkg | List installed packages matching a pattern |
| `dpkg -L package` | dpkg | List files installed by a package |
| `dpkg -s package` | dpkg | Show status/details of a package |
| `dpkg -S pattern` | dpkg | Find which package owns a file |
| `dpkg-query -W -f='FORMAT'` | dpkg | Print custom-formatted metadata for installed packages |
| `dpkg -i file.deb` | dpkg | Install a `.deb` package file directly |
| `apt-cache show package \| grep field` | apt | Extract a specific property (version, arch, priority, etc.) |
| `apt-cache depends package` | apt | List a package's dependencies |
| `apt list --installed` | apt | List all installed packages with versions |
| `dpkg --print-architecture` | dpkg | Show the system's native architecture |
| `md5sum` / `sha256sum` file | core-utils | Verify checksum/integrity of a downloaded package file |
| `cat /var/log/apt/history.log` | logs | View high-level history of apt install/remove/upgrade actions |
| `cat /var/log/dpkg.log` | logs | View low-level dpkg package state-change log |

---

## 17. Summary

- Before managing packages, check **system information**: OS type (`cat /etc/os-release`, `lsb_release -a`), and kernel/architecture (`uname -a`, `uname -m`, `arch`) — this determines which package format and architecture you need.
- A **package manager** provides installation, updating, removal, dependency resolution, signature verification, and package-building capabilities, and maintains a database of installed software.
- Linux packaging is split into two major families: **RPM** (Red Hat, CentOS, Fedora, openSUSE, SUSE, Oracle Linux) and **DEB** (Debian, Ubuntu, Mint, Knoppix).
- Packages target specific **architectures** (`amd64`, `i386`, `arm`, `ppc64el`, or architecture-independent `all`/`noarch`/`src`).
- RPM-based systems use `rpm` (low-level) with `yum`/`dnf` (high-level) as management tools; DEB-based systems use `dpkg`/`dpkg-deb` (low-level) with `apt`/`aptitude`/`synaptic` (high-level).
- Package filenames follow strict naming conventions encoding **name, version, release/revision, and architecture**.
- Packages carry a **priority** (`required`, `important`, `standard`, `optional`, `extra`) indicating how critical they are to the system, and are grouped into **sections** by functional category.
- **Checksums** (`md5sum`, `SHA1`, `SHA256`) verify package integrity after download.
- Only **sudoers** (defined in `/etc/sudoers`, safely edited with `visudo`) can install, upgrade, or remove packages system-wide.
- **Log files** under `/var/log` (`apt/history.log`, `apt/term.log`, `dpkg.log`) record every package operation for troubleshooting and auditing.
- On Ubuntu, **`apt`** is the primary high-level tool:
  - Configuration lives in `/etc/apt`, `/etc/apt/sources.list`, and `/etc/apt/sources.list.d`.
  - `apt-cache` is used to **query** the package database (`search`, `pkgnames`, `pkgnames prefix`, `show`).
  - `apt-get update` / `upgrade` / `install` / `reinstall` handle **installation and updates**.
  - `apt-get autoremove` / `clean` / `remove` / `purge` handle **removal and cleanup**.
- **`dpkg`** is the low-level tool underlying `apt`, used to query (`-l`, `-L`, `-s`, `-S`) and install (`-i`) individual `.deb` files directly. Its database lives in `/var/lib/dpkg`.
- **`dpkg-query -W -f='FORMAT'`** offers powerful custom-formatted reporting (e.g., section + package name per line), which can be combined with `less` (paging), `sort` (ordering), and `grep` (filtering) for readable, targeted reports.
- Best practice: prefer `apt` over raw `dpkg` for installs/removals, since `apt` resolves dependencies automatically — `dpkg` does not, and using it to uninstall packages is discouraged.
- **Retrieving package properties:** Almost every property (version, architecture, priority, section, dependencies, size, status) is a labelled field inside the output of `apt-cache show package` (for any repository package) or `dpkg -s package` (for an installed package). Pipe either command through `grep -i "fieldname"`, or use `dpkg-query -f='${FieldName}\n'`, to isolate the exact property needed — this is the single most useful exam technique for "how do I find out X" style questions.

---

*End of Notes — Week 3, Lecture 3 & 4: Software Management (Package Management Systems)*
