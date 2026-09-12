# Knowing Your Hardware
### A Professional Reference Guide to Command-Line Hardware Inspection & Diagnostics on Linux

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Packages to Install](#2-packages-to-install)
3. [CPU Inspection](#3-cpu-inspection)
   - 3.1 [hwinfo](#31-hwinfo)
   - 3.2 [lshw](#32-lshw)
   - 3.3 [cat /proc/cpuinfo](#33-cat-proccpuinfo)
   - 3.4 [lscpu (bonus companion tool)](#34-lscpu-bonus-companion-tool)
4. [Graphics Card Inspection](#4-graphics-card-inspection)
   - 4.1 [lshw -c display](#41-lshw--c-display)
   - 4.2 [clinfo (OpenCL details)](#42-clinfo-opencl-details)
5. [Storage & Partitions](#5-storage--partitions)
   - 5.1 [cat /proc/partitions](#51-cat-procpartitions)
   - 5.2 [lsblk](#52-lsblk)
   - 5.3 [df -h](#53-df--h)
   - 5.4 [hdparm -Tt](#54-hdparm--tt)
   - 5.5 [iostat -dx](#55-iostat--dx)
6. [Memory (RAM) Inspection](#6-memory-ram-inspection)
   - 6.1 [free](#61-free)
   - 6.2 [dmidecode --type memory](#62-dmidecode---type-memory)
7. [Network Devices & Configuration](#7-network-devices--configuration)
   - 7.1 [lspci (network adapters)](#71-lspci-network-adapters)
   - 7.2 [ifconfig (legacy) vs ip (modern)](#72-ifconfig-legacy-vs-ip-modern)
8. [Battery & Power Status](#8-battery--power-status)
   - 8.1 [upower](#81-upower)
9. [All-in-One / GUI-Style Tools](#9-all-in-one--gui-style-tools)
   - 9.1 [hardinfo](#91-hardinfo)
10. [Master Summary Table](#10-master-summary-table)
11. [Cheat Sheet (Quick Reference)](#11-cheat-sheet-quick-reference)
12. [Exam-Style Q&A](#12-exam-style-qa)
13. [Extended Flag & Type Reference](#13-extended-flag--type-reference)
14. [Common Errors & Troubleshooting](#14-common-errors--troubleshooting)
15. [Automation: Building a Full Hardware-Audit Script](#15-automation-building-a-full-hardware-audit-script)
16. [Reference Sources](#16-reference-sources)

---

## 1. Introduction

Every Linux system exposes its hardware through two layers:

1. **Kernel-provided virtual filesystems** — `/proc` and `/sys` — which reflect **live, real-time** hardware and driver state maintained directly by the kernel.
2. **Userspace diagnostic utilities** — programs like `lshw`, `hwinfo`, `dmidecode`, `lsblk`, etc. — which read from `/proc`, `/sys`, the **DMI/SMBIOS firmware table**, and kernel APIs (`ioctl`, `netlink`) to present that data in a structured, human-readable form.

Understanding both layers matters professionally because:

- **System administrators** use these tools to audit servers before deployment, verify RAM/CPU matches purchase specs, and diagnose failing components.
- **Support engineers** use them to generate hardware reports for tickets.
- **DevOps/SRE teams** use performance tools (`iostat`, `hdparm`) to find bottlenecks in production.
- **Exam candidates** (CompTIA Linux+, RHCSA, LPIC) are frequently tested on exactly these command families.

This guide is organized **by hardware category** — CPU, Graphics, Storage, Memory, Network, Battery — matching the reference PDF and its accompanying video walkthrough, and expands each command with **syntax, real sample output, multiple usage examples, key points to remember, and an official reference source**.

> ⚠️ **Important Point to Remember:** Commands that read hardware-level or firmware-level data (`lshw`, `dmidecode`, `hdparm`) almost always require **root privileges (`sudo`)** for complete and accurate output. Commands that read `/proc` or `/sys` directly (`cat /proc/cpuinfo`, `free`) generally do **not** need root.

---

## 2. Packages to Install

Most of these tools are **not installed by default** on a minimal Linux system (especially server/cloud images). Install them in one shot:

**Debian / Ubuntu:**
```bash
sudo apt update
sudo apt install clinfo coreutils dmidecode fdisk hardinfo hdparm \
                  hwinfo lshw memtester net-tools pciutils procps \
                  sysstat upower util-linux
```

**RHEL / CentOS / Fedora:**
```bash
sudo dnf install clinfo coreutils dmidecode util-linux-ng hardinfo \
                  hdparm hwinfo lshw memtester net-tools pciutils \
                  procps-ng sysstat upower
```

**Arch Linux:**
```bash
sudo pacman -S clinfo coreutils dmidecode hardinfo hdparm hwinfo \
               lshw memtester net-tools pciutils procps-ng sysstat \
               upower util-linux
```

| Package | Provides | Purpose |
|---|---|---|
| `clinfo` | `clinfo` | Displays OpenCL platform/device info (GPU/CPU compute capability) |
| `coreutils` | `df`, `cat`, `free` (partly) | Core GNU utilities — usually pre-installed on every distro |
| `dmidecode` | `dmidecode` | Reads DMI/SMBIOS firmware table — motherboard, RAM, BIOS, chassis details |
| `fdisk` (util-linux) | `fdisk` | Partition table viewer/editor (MBR and GPT) |
| `hardinfo` | `hardinfo` | GUI/report-based system profiler and benchmarking tool |
| `hdparm` | `hdparm` | Get/set SATA/IDE/PATA disk parameters; benchmark disk speed |
| `hwinfo` | `hwinfo` | Deep hardware probing across *all* hardware classes |
| `lshw` | `lshw` | "List Hardware" — structured hardware tree, HTML/XML export |
| `memtester` | `memtester` | Userspace RAM stress-tester (detects faulty memory) |
| `net-tools` | `ifconfig`, `netstat`, `route`, `arp` | Legacy (deprecated) networking utilities |
| `pciutils` | `lspci`, `setpci` | Lists/configures PCI and PCIe devices |
| `procps` / `procps-ng` | `free`, `ps`, `top`, `uptime`, `vmstat` | `/proc`-based process & resource utilities |
| `sysstat` | `iostat`, `mpstat`, `sar`, `pidstat` | Performance monitoring & historical stats collection |
| `upower` | `upower` | Battery/AC power device info via UPower D-Bus service |
| `util-linux` | `lsblk`, `fdisk`, `dmesg`, `blkid` | Core low-level system utilities |

> ✅ **Remember for exams:** Package-to-command mapping is a favorite trick question:
> - `lspci` → **pciutils**
> - `lsblk` → **util-linux**
> - `iostat` → **sysstat**
> - `ifconfig` → **net-tools** (legacy)
> - `ip` → **iproute2** (NOT listed above — it's the modern default, usually pre-installed)

**📚 Source:** [Debian Package Search](https://packages.debian.org/) · [Arch Linux Package Database](https://archlinux.org/packages/) · `man apt` / `man dnf` / `man pacman`

---

## 3. CPU Inspection

### 3.1 hwinfo

**Purpose:** A universal, exhaustive hardware-probing tool developed by SUSE/openSUSE. Reports on nearly every hardware class in one place.

**Basic syntax:**
```bash
hwinfo [--CLASS] [options]
```

**Examples:**
```bash
hwinfo --cpu                 # CPU details only
hwinfo --short               # one-line summary of every hardware class
hwinfo --cpu --short         # short CPU summary only
hwinfo --network             # network hardware
hwinfo --disk                # storage devices
hwinfo --gfxcard              # graphics card
hwinfo --log=/tmp/hw.log --cpu   # save full CPU probe results to a log file
```

**Sample output (`hwinfo --short`):**
```
cpu:
                       AMD Ryzen 7 5800H, 3200 MHz
graphics card:
                       NVIDIA GeForce RTX 3050 Mobile
network interface:
                       enp3s0
                       wlan0
disk:
                       /dev/sda               SAMSUNG MZVL2512
```

> ⚠️ **Important Points to Remember:**
> - `hwinfo` output is **very verbose** by default — always filter with a `--CLASS` flag (`--cpu`, `--disk`, `--network`) in real use or exams.
> - It is one of the **few tools that probes hardware even if no driver is currently loaded**, useful for troubleshooting missing/broken drivers.
> - Run with `sudo` for full detail — some fields (like exact bus addresses) are hidden for unprivileged users.

**📚 Source:** [openSUSE hwinfo documentation](https://github.com/openSUSE/hwinfo) · `man hwinfo`

---

### 3.2 lshw

**Purpose:** "**L**ist **H**ard**w**are" — produces a structured, hierarchical (tree-style) report of the *entire* system, written by Lyonel Vincent (originally for the Ezix project).

**Basic syntax:**
```bash
sudo lshw [-class CLASSNAME] [-short] [-html|-xml] [-sanitize]
```

**Examples:**
```bash
sudo lshw                        # full detailed tree of entire system
sudo lshw -short                 # compact table: device, class, description
sudo lshw -class cpu             # CPU class only
sudo lshw -class disk            # storage devices only
sudo lshw -class memory          # memory controller + DIMMs
sudo lshw -html > report.html    # export as a shareable HTML report
sudo lshw -xml > report.xml      # export as XML (for scripting/parsing)
sudo lshw -sanitize -html > safe_report.html   # hides serial numbers before sharing
```

**Sample output (`sudo lshw -short`):**
```
H/W path          Device      Class          Description
=========================================================
                              system         ThinkPad T14
/0                            bus            Motherboard
/0/4                          memory         16GiB System memory
/0/1                          processor      AMD Ryzen 5 PRO 4650U
/0/100/2                      display        Renoir
/0/100/1.5/0     /dev/sda     disk           512GB NVMe SSD
```

> ⚠️ **Important Points to Remember:**
> - Always run with **`sudo`** — without it, `lshw` cannot read BIOS/DMI data and will show incomplete or "**?**" entries for many fields.
> - `-sanitize` is critical when **sharing a hardware report publicly** (e.g., on a forum or support ticket) — it strips serial numbers and other identifying info.
> - `-class` accepts: `system`, `bus`, `memory`, `processor`, `bridge`, `display`, `input`, `printer`, `multimedia`, `communication`, `network`, `disk`, `storage`, `power`, `volume`.
> - `lshw` can be **much slower** than `lsblk`/`free` because it re-probes the entire bus tree each run.

**📚 Source:** [lshw official project (Ezix.org)](https://ezix.org/project/wiki/HardwareLiSter) · `man lshw`

---

### 3.3 cat /proc/cpuinfo

**Purpose:** Reads directly from the kernel's **`/proc` virtual filesystem** — a live snapshot of every logical CPU core, updated in real time by the kernel.

**Basic syntax:**
```bash
cat /proc/cpuinfo
```

**Examples:**
```bash
cat /proc/cpuinfo                          # full raw dump (all cores)
grep "model name" /proc/cpuinfo | uniq     # CPU model, deduplicated
grep -c ^processor /proc/cpuinfo           # count of logical CPUs (threads)
grep "cpu MHz" /proc/cpuinfo               # current clock speed per core
grep flags /proc/cpuinfo | head -1 | tr ' ' '\n' | grep -E 'vmx|svm'  # check virtualization support
awk -F: '/cache size/ {print $2; exit}' /proc/cpuinfo   # L2/L3 cache size of core 0
```

**Sample output (excerpt, one core):**
```
processor       : 0
vendor_id       : AuthenticAMD
model name      : AMD Ryzen 5 PRO 4650U with Radeon Graphics
cpu MHz         : 1400.000
cache size      : 512 KB
physical id     : 0
siblings        : 12
cpu cores       : 6
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep ... sse4_2 avx2 svm ...
```

**Field-by-field meaning (key ones):**

| Field | Meaning |
|---|---|
| `processor` | Logical CPU index (0-based) — count these for thread total |
| `model name` | Full marketing name of the CPU |
| `cpu MHz` | Current (not max) operating frequency of this core |
| `cache size` | Size of this core's cache (often L2) |
| `physical id` | Which physical socket this core belongs to (matters on multi-socket servers) |
| `siblings` | Logical cores per physical package (includes Hyper-Threading/SMT) |
| `cpu cores` | Actual physical cores per package |
| `flags` | Feature set: `sse`, `avx`, `avx2` (SIMD); `vmx` (Intel VT-x) / `svm` (AMD-V) for virtualization |

> ⚠️ **Important Points to Remember:**
> - `/proc/cpuinfo` lists **one block per logical core** — on a hyper-threaded 6-core CPU, you will see **12** `processor` entries, not 6.
> - `cpu MHz` reflects the **current dynamic frequency** (affected by power-saving/turbo boost), not the CPU's rated base or max clock — don't confuse this with the marketing spec.
> - Checking for `vmx`/`svm` in `flags` is the standard way to verify **hardware virtualization support** before enabling KVM/VirtualBox.
> - No `sudo` required — this file is world-readable.

**📚 Source:** [The Linux Kernel Documentation — /proc filesystem](https://www.kernel.org/doc/html/latest/filesystems/proc.html) · `man proc`

---

### 3.4 lscpu (bonus companion tool)

While not in the original list, `lscpu` (from `util-linux`) is the modern, **structured** companion to `/proc/cpuinfo` and is worth knowing for exams.

```bash
lscpu
```

**Sample output (excerpt):**
```
Architecture:            x86_64
CPU(s):                  12
On-line CPU(s) list:     0-11
Thread(s) per core:      2
Core(s) per socket:      6
Socket(s):               1
Model name:              AMD Ryzen 5 PRO 4650U with Radeon Graphics
CPU max MHz:             4000.0000
CPU min MHz:             1400.0000
```

> ✅ **Remember:** `lscpu` **parses and summarizes** `/proc/cpuinfo` for you — it's faster to read in an exam/interview setting than grepping raw output.

**📚 Source:** `man lscpu` (util-linux)

---

## 4. Graphics Card Inspection

### 4.1 lshw -c display

**Purpose:** Filters the full `lshw` tree down to the **display/graphics class** only.

**Basic syntax:**
```bash
sudo lshw -c display
```
> Note: `-c` and `-class` are interchangeable shorthand/long forms in `lshw`.

**Examples:**
```bash
sudo lshw -c display                     # graphics class detail
sudo lshw -c display -short              # one-line summary form
sudo lshw -c display -numeric            # show numeric PCI/USB IDs alongside names
lspci | grep -i vga                      # cross-check via PCI bus directly
lspci -v -s 03:00.0                      # verbose info for a specific GPU's bus address
lspci -k | grep -A 3 -i vga              # show which kernel driver is bound to the GPU
```

**Sample output:**
```
*-display
       description: VGA compatible controller
       product: Renoir (Radeon Vega Series)
       vendor: Advanced Micro Devices, Inc. [AMD/ATI]
       physical id: 0
       bus info: pci@0000:04:00.0
       version: c1
       width: 64 bits
       clock: 33MHz
       configuration: driver=amdgpu latency=0
       resources: irq:69 memory:e0000000-efffffff memory:f0000000-f01fffff
```

> ⚠️ **Important Points to Remember:**
> - The `configuration: driver=...` line tells you **exactly which kernel driver** is currently bound to the GPU (e.g., `amdgpu`, `nouveau`, `nvidia`, `i915`) — critical for diagnosing display/driver issues.
> - Systems with **hybrid graphics** (integrated + discrete, e.g., laptops) will show **two `*-display` blocks** — always check both.
> - `lspci | grep -i vga` is the fastest one-liner when you just need the GPU **name**, without full `lshw` detail.

**📚 Source:** [lshw project wiki](https://ezix.org/project/wiki/HardwareLiSter) · [The PCI Utilities project (pciutils)](https://mj.ucw.cz/sw/pciutils/) · `man lspci`

---

### 4.2 clinfo (OpenCL details)

**Purpose:** Lists **OpenCL platforms and devices** — relevant when a GPU (or even the CPU) is used for parallel/GPU **compute** workloads, not just display rendering.

**Basic syntax:**
```bash
clinfo [options]
```

**Examples:**
```bash
clinfo                          # full detailed dump of all OpenCL platforms/devices
clinfo -l                       # short list view (platform + device names only)
clinfo --raw                    # machine-parsable raw key/value output (for scripts)
clinfo -l | grep -i "device"    # quickly list all available compute devices
```

**Sample output (`clinfo -l`):**
```
Platform #0: AMD Accelerated Parallel Processing
 `-- Device #0: gfx90c
Platform #1: Intel(R) OpenCL
 `-- Device #0: 11th Gen Intel(R) Core(TM) i7 CPU
```

**Key fields in full `clinfo` output:**

| Field | Meaning |
|---|---|
| `Max compute units` | Number of parallel compute cores available on the device |
| `Max clock frequency` | Peak GPU/compute clock speed |
| `Global memory size` | Total VRAM (or system RAM, for CPU OpenCL) usable for compute |
| `Device Version` | OpenCL spec version supported (1.2, 2.0, 3.0, etc.) |

> ⚠️ **Important Points to Remember:**
> - If `clinfo` reports **"Number of platforms: 0"**, it usually means the **ICD (Installable Client Driver) loader** or vendor GPU driver is missing — a very common exam/troubleshooting trap.
> - OpenCL support is separate from **CUDA** (NVIDIA-proprietary) — a system can have CUDA installed but still show 0 OpenCL platforms if the OpenCL ICD isn't installed.
> - Useful before setting up GPU-accelerated workloads: video transcoding (`ffmpeg` with OpenCL), machine learning, or cryptocurrency mining rigs.

**📚 Source:** [Khronos Group — OpenCL Registry](https://www.khronos.org/opencl/) · [clinfo GitHub repository (Oblomov/clinfo)](https://github.com/Oblomov/clinfo)

---

## 5. Storage & Partitions

### 5.1 cat /proc/partitions

**Purpose:** The kernel's raw, live view of every detected **block device and partition**.

**Basic syntax:**
```bash
cat /proc/partitions
```

**Examples:**
```bash
cat /proc/partitions                     # full raw listing
cat /proc/partitions | awk '{print $4}'  # extract just device names
watch -n 2 cat /proc/partitions          # refresh view every 2 seconds (detect hot-plugged drives)
```

**Sample output:**
```
major minor  #blocks  name
   8        0  500107608 sda
   8        1     524288 sda1
   8        2  499581952 sda2
 259        0  976762584 nvme0n1
```

| Column | Meaning |
|---|---|
| `major` | Device driver ID (e.g., 8 = SCSI/SATA disk, 259 = NVMe) |
| `minor` | Specific device/partition instance number |
| `#blocks` | Size in 1024-byte blocks |
| `name` | Device/partition name as seen in `/dev/` |

> ⚠️ **Remember:** This is the **lowest-level, unformatted** view — it shows raw kernel block-device registration, with no filesystem type, label, or mount point info (unlike `lsblk`).

**📚 Source:** [Linux Kernel Documentation — /proc filesystem](https://www.kernel.org/doc/html/latest/filesystems/proc.html)

---

### 5.2 lsblk

**Purpose:** **"List Block devices"** — the modern, human-readable, tree-formatted replacement for manually reading `/proc/partitions`.

**Basic syntax:**
```bash
lsblk [options]
```

**Examples:**
```bash
lsblk                          # basic tree: disks → partitions → mount points
lsblk -f                       # include filesystem type, label, and UUID
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT,FSTYPE   # choose exact columns to display
lsblk -a                       # include empty devices (e.g., disconnected drives)
lsblk -d                       # list only disks, hide partitions
lsblk /dev/sda                 # restrict output to one specific disk
lsblk -p                       # show full /dev/ paths instead of bare names
```

**Sample output:**
```
NAME        SIZE FSTYPE   TYPE MOUNTPOINT
sda         465.8G                disk
├─sda1        512M vfat    part /boot/efi
└─sda2      465.3G ext4    part /
nvme0n1     931.5G                disk
└─nvme0n1p1 931.5G ntfs    part
```

> ⚠️ **Important Points to Remember:**
> - `lsblk` reads from **`/sys` (sysfs)**, not `/proc/partitions` — it is the officially recommended tool going forward.
> - `-f` is the single most useful flag for **exam/interview** questions — it reveals filesystem type and UUID in one command (otherwise you'd need `blkid`).
> - Works **without `sudo`** for basic viewing.

**📚 Source:** `man lsblk` (util-linux) · [util-linux GitHub repository](https://github.com/util-linux/util-linux)

---

### 5.3 df -h

**Purpose:** **"Disk Free"** — reports **used/available space per mounted filesystem**, from the filesystem's own superblock statistics.

**Basic syntax:**
```bash
df [options] [path]
```

**Examples:**
```bash
df -h                       # human-readable sizes (GB/MB) for all mounted filesystems
df -h /                     # check just the root filesystem
df -hT                      # also display the filesystem type column
df -i                       # show inode usage instead of block/space usage
df -h --total               # add a combined total row at the bottom
```

**Sample output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2       456G   89G  344G  21% /
/dev/sda1       512M   28M  485M   6% /boot/efi
tmpfs           7.8G     0  7.8G   0% /dev/shm
```

> ⚠️ **Important Points to Remember:**
> - `df` reports space based on the **filesystem**, not the raw block device — a full disk (`df` shows 100%) can sometimes still fail to create *new* files even with space left, if **inodes** are exhausted (check with `df -i`).
> - `tmpfs` entries are **RAM-backed virtual filesystems** — they don't represent real disk usage.
> - Confusingly named cousin: `du` (**disk usage**) reports space consumed by *files/directories*, while `df` reports space on the *filesystem* as a whole — don't mix these up in exams.

**📚 Source:** [GNU Coreutils Manual — df](https://www.gnu.org/software/coreutils/manual/html_node/df-invocation.html)

---

### 5.4 hdparm -Tt

**Purpose:** Benchmarks **physical disk I/O performance** directly against SATA/IDE/PATA drives.

**Basic syntax:**
```bash
sudo hdparm -Tt /dev/sdX
```

**Examples:**
```bash
sudo hdparm -Tt /dev/sda              # combined cache + disk read benchmark
sudo hdparm -T /dev/sda               # cached reads only (tests RAM + CPU path)
sudo hdparm -t /dev/sda               # buffered disk reads only (tests actual disk speed)
sudo hdparm -Tt /dev/sda --direct     # bypass OS cache entirely for a truer raw result
sudo hdparm -i /dev/sda               # show identification info (model, firmware, geometry)
sudo hdparm -I /dev/sda               # detailed ATA IDENTIFY data (supported features, SMART)
```

**Sample output:**
```
/dev/sda:
 Timing cached reads:   25412 MB in  2.00 seconds = 12706.44 MB/sec
 Timing buffered disk reads: 1560 MB in  3.00 seconds = 519.87 MB/sec
```

| Flag | Meaning |
|---|---|
| `-T` | Cached reads — measures **RAM + CPU + cache** speed, not the disk itself |
| `-t` | Buffered disk reads — measures actual **physical disk** read throughput |
| `-I` | Full ATA IDENTIFY device data (SMART support, supported transfer modes) |

> ⚠️ **Important Points to Remember:**
> - Run the test **2–3 times** and average — background OS activity (indexing, updates) can skew a single reading.
> - `hdparm` primarily targets **SATA/PATA** drives. For **NVMe** SSDs, use `nvme-cli` (e.g., `sudo nvme smart-log /dev/nvme0`) instead — `hdparm` may report inaccurate/no data on NVMe.
> - This test is **read-only** by default and safe; however, `hdparm` also supports **write/config-changing flags** (like disabling write caching) that *can* be destructive if misused — always double-check flags before running with real drives.

**📚 Source:** [hdparm man page (sourceforge project)](https://sourceforge.net/projects/hdparm/) · `man hdparm`

---

### 5.5 iostat -dx

**Purpose:** Part of the `sysstat` package — reports **extended, per-device I/O statistics**, ideal for diagnosing live disk bottlenecks.

**Basic syntax:**
```bash
iostat [-d] [-x] [device] [interval] [count]
```

**Examples:**
```bash
iostat -dx /dev/sdb              # one-shot extended stats for /dev/sdb
iostat -dx 2 5                   # refresh every 2 seconds, for 5 iterations total
iostat -dx                       # extended stats for ALL block devices
iostat -c                        # CPU utilization stats instead of disk
iostat -x -h                     # human-readable extended report
iostat -dx /dev/sdb 1 | ts       # (with moreutils) timestamp each refreshed block
```

**Sample output:**
```
Device            r/s     w/s     rkB/s     wkB/s   await  %util
sdb              12.00    4.50    512.30    210.10    3.21   18.40
```

| Column | Meaning |
|---|---|
| `r/s` / `w/s` | Reads/writes completed per second |
| `rkB/s` / `wkB/s` | Kilobytes read/written per second |
| `await` | Average time (ms) for I/O requests to be served (includes queue wait) |
| `%util` | Percentage of time the device was busy servicing I/O — **the key bottleneck indicator** |

> ⚠️ **Important Points to Remember:**
> - **`%util` approaching 100%** on a single spinning disk strongly indicates it is the system's I/O bottleneck; for SSDs/NVMe with multiple internal channels, 100% is less immediately alarming but still worth investigating alongside `await`.
> - The **first report** `iostat` prints after boot reflects **averages since boot** — always look at the **second and later** samples (`iostat -dx 2 5`) for a true "current" reading.
> - Related sibling tools in the same package: `mpstat` (per-CPU stats), `sar` (historical system activity logging), `pidstat` (per-process I/O/CPU stats).

**📚 Source:** [sysstat official documentation](https://github.com/sysstat/sysstat) · `man iostat`

---

## 6. Memory (RAM) Inspection

### 6.1 free

**Purpose:** Quick, live summary of **total, used, free, shared, buffer/cache, and available** memory — both RAM and swap.

**Basic syntax:**
```bash
free [options]
```

**Examples:**
```bash
free -h                     # human-readable units (GB/MB)
free -h -s 2                # auto-refresh every 2 seconds (like a lightweight top)
free -m                     # force output in megabytes
free -g                     # force output in gigabytes
free -h --total             # add a combined RAM+swap total row
free -w                     # "wide" mode — splits buffers and cache into separate columns
```

**Sample output:**
```
               total        used        free      shared  buff/cache   available
Mem:            15Gi       4.2Gi       6.1Gi       412Mi       5.0Gi        10Gi
Swap:          2.0Gi          0B       2.0Gi
```

| Column | Meaning |
|---|---|
| `total` | Total installed physical RAM |
| `used` | Memory actively used by processes (excludes buffers/cache) |
| `free` | Completely unused memory |
| `buff/cache` | Memory used by the kernel for disk caching/buffers — **reclaimable on demand** |
| `available` | **The real number to watch** — estimates memory available for new applications without swapping |

> ⚠️ **Important Points to Remember:**
> - **Never judge memory pressure by the `free` column alone.** Linux aggressively uses spare RAM for disk **cache** (`buff/cache`), which is reclaimed instantly when an app needs it. Always check the **`available`** column instead — this is the single most common misunderstanding tested in exams.
> - `Swap` usage of `0B` is healthy; heavy sustained swap usage indicates genuine RAM pressure.
> - No `sudo` required.

**📚 Source:** [GNU Coreutils / procps-ng documentation](https://gitlab.com/procps-ng/procps) · `man free`

---

### 6.2 dmidecode --type memory

**Purpose:** Reads the **DMI/SMBIOS table** — a standardized data structure written by motherboard firmware — to reveal **per-DIMM-slot physical hardware details**.

**Basic syntax:**
```bash
sudo dmidecode --type TYPE
```

**Examples:**
```bash
sudo dmidecode --type memory              # full detail for every DIMM slot (populated + empty)
sudo dmidecode -t 17                      # same as above; type 17 = "Memory Device" in SMBIOS spec
sudo dmidecode --type memory | grep -A2 "Size:"    # quick capacity check per slot
sudo dmidecode -t bios                    # BIOS/UEFI vendor, version, release date
sudo dmidecode -t system                  # system manufacturer, product name, serial (laptop/desktop model)
sudo dmidecode -t baseboard                # motherboard model, manufacturer, serial
sudo dmidecode --type memory | grep -E "Locator|Size|Speed|Manufacturer"   # condensed slot-by-slot summary
```

**Sample output (one populated slot + one empty slot):**
```
Memory Device
        Size: 8192 MB
        Form Factor: SODIMM
        Locator: ChannelA-DIMM0
        Type: DDR4
        Speed: 3200 MT/s
        Manufacturer: SK Hynix
        Part Number: HMA81GS6DJR8N-XN

Memory Device
        Size: No Module Installed
        Locator: ChannelB-DIMM0
```

> ⚠️ **Important Points to Remember:**
> - `dmidecode` gives you **physical hardware truth** (slot layout, max supported speed, empty slots for upgrades) — this is fundamentally different from `free`, which only shows **logical/OS-level usage**. A very common exam distinction.
> - **Requires `sudo`/root** — DMI table access needs elevated privileges; without it, you'll get a "Permission denied" error.
> - **Does not work reliably inside virtual machines** — hypervisors often present fake or minimal SMBIOS data, so DIMM details may show as generic/blank in a VM.
> - SMBIOS **Type 17** = Memory Device; **Type 16** = Physical Memory Array (overall capacity/slot count); knowing these type numbers is a common certification-exam detail.

**📚 Source:** [DMTF SMBIOS Specification](https://www.dmtf.org/standards/smbios) · `man dmidecode`

---

## 7. Network Devices & Configuration

### 7.1 lspci (network adapters)

**Purpose:** Lists PCI/PCIe-connected hardware — including **Ethernet and Wi-Fi controllers** — with vendor/chipset identification.

**Basic syntax:**
```bash
lspci [options]
```

**Examples:**
```bash
lspci | grep -i net              # quick identification of network adapters
lspci -v -s 02:00.0              # verbose detail for a specific network device's bus ID
lspci -k | grep -A 3 -i eth      # show which kernel driver is bound to the Ethernet card
lspci -nn | grep -i net          # include numeric vendor:device IDs (useful for driver lookup)
lspci -vv                        # very verbose output for every PCI device on the system
```

**Sample output:**
```
02:00.0 Ethernet controller: Intel Corporation Ethernet Connection I219-V
03:00.0 Network controller: Intel Corporation Wi-Fi 6 AX200
```

> ⚠️ **Important Points to Remember:**
> - **"Ethernet controller"** = wired NIC; **"Network controller"** = typically Wi-Fi/wireless hardware — the labeling difference is a common exam trap.
> - `-nn` numeric IDs (e.g., `[8086:15bb]`) are extremely useful when searching for the exact **driver or firmware package** needed for an unrecognized card.
> - No `sudo` required for basic listing; `sudo` needed for some `-vv` extended register-level details.

**📚 Source:** [The PCI Utilities project](https://mj.ucw.cz/sw/pciutils/) · `man lspci`

---

### 7.2 ifconfig (legacy) vs ip (modern)

**Purpose:** Both display and configure **network interfaces** — IP addresses, status, statistics — but represent two different generations of Linux networking tools.

**ifconfig (net-tools) — legacy:**
```bash
ifconfig                      # show all active interfaces
ifconfig -a                   # show ALL interfaces, including down/inactive ones
ifconfig eth0                 # details for one specific interface
sudo ifconfig eth0 up         # bring an interface up
sudo ifconfig eth0 down       # bring an interface down
sudo ifconfig eth0 192.168.1.50 netmask 255.255.255.0   # manually assign an IP
```

**ip (iproute2) — modern standard:**
```bash
ip addr show                  # (or: ip a) — show IP addresses for all interfaces
ip link show                  # show interface status (UP/DOWN) and MAC addresses
ip route show                 # (or: ip r) — display the routing table
ip -s link show eth0          # show interface statistics (packets/errors/drops)
sudo ip link set eth0 up      # bring an interface up
sudo ip addr add 192.168.1.50/24 dev eth0   # assign an IP address with CIDR notation
ip neigh show                 # (replaces "arp -a") — show the ARP/neighbor cache
```

**Sample output (`ip addr show`):**
```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP
    link/ether 3c:52:82:1a:9f:22 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.34/24 brd 192.168.1.255 scope global dynamic eth0
       valid_lft 86340sec preferred_lft 86340sec
```

**Command translation table (legacy → modern):**

| Legacy (`net-tools`) | Modern (`iproute2`) |
|---|---|
| `ifconfig` | `ip addr` (`ip a`) |
| `ifconfig eth0 up` | `ip link set eth0 up` |
| `ifconfig eth0 down` | `ip link set eth0 down` |
| `route -n` | `ip route` (`ip r`) |
| `arp -a` | `ip neigh` |
| `netstat -i` | `ip -s link` |
| `netstat -r` | `ip route` |

> ⚠️ **Important Points to Remember:**
> - `ifconfig` is **deprecated and unmaintained** upstream; many modern minimal/server distros (e.g., current Debian/Ubuntu server images) **do not install it by default** — you must install `net-tools` explicitly.
> - `ip` (from **iproute2**) is the actively maintained, feature-complete standard and is **pre-installed by default** on virtually all modern distros.
> - Any changes made with `ip addr add` / `ip link set` are **not persistent** across reboots by default — permanent changes require editing distro-specific network config files (e.g., Netplan, NetworkManager, `/etc/network/interfaces`).
> - Certification exams (LPIC, RHCSA, CompTIA Linux+) increasingly test **only** the `ip` command family — know it, not just `ifconfig`.

**📚 Source:** [iproute2 project (kernel.org)](https://wiki.linuxfoundation.org/networking/iproute2) · `man ip` · `man ifconfig`

---

## 8. Battery & Power Status

### 8.1 upower

**Purpose:** Interfaces with the **UPower D-Bus service** to report battery and AC adapter status — primarily for laptops.

**Basic syntax:**
```bash
upower [options]
```

**Examples:**
```bash
upower -e                                          # enumerate all power devices (list device paths)
upower -i /org/freedesktop/UPower/devices/battery_BAT0   # detailed info for the battery
upower -d                                          # dump full details for every detected device
upower --monitor                                   # live-monitor power/battery events as they happen
upower -i $(upower -e | grep BAT)                  # one-liner: auto-detect and show battery info
```

**Sample output:**
```
  native-path:          BAT0
  vendor:               SMP
  model:                L19M4PC1
  state:                discharging
  energy:               35.6 Wh
  energy-full:          52.6 Wh
  energy-full-design:   57.0 Wh
  energy-rate:          8.9 W
  percentage:           67%
  time to empty:        4.0 hours
```

| Field | Meaning |
|---|---|
| `state` | `charging`, `discharging`, `fully-charged`, or `pending-charge` |
| `energy-full` | Current maximum charge capacity the battery can hold |
| `energy-full-design` | Original factory-rated maximum capacity (when brand new) |
| `energy-rate` | Current power draw/charge rate in Watts |
| `percentage` | Current charge level |

> ⚠️ **Important Points to Remember:**
> - **Battery health/degradation** is calculated as `energy-full ÷ energy-full-design × 100` — a very common practical/exam calculation (e.g., 52.6 ÷ 57.0 ≈ 92% health remaining).
> - Desktops (no battery) will show `upower -e` returning **only a `line_power` (AC adapter) device**, with no `battery_BAT*` entries — this is expected, not an error.
> - `upower` depends on the **UPower daemon** running via D-Bus; if the service isn't active, commands may return empty/no devices.

**📚 Source:** [freedesktop.org — UPower project](https://upower.freedesktop.org/) · `man upower`

---

## 9. All-in-One / GUI-Style Tools

### 9.1 hardinfo

**Purpose:** A system profiler with both a **GTK-based GUI** and **report-generation** capability, aggregating CPU, memory, storage, network, and sensor data into a single dashboard — beginner-friendly alternative to memorizing every individual CLI tool.

**Basic usage:**
```bash
hardinfo                              # launch the GUI application
hardinfo -r                           # generate a full report (auto-selects all modules)
hardinfo -r -f html -o report.html    # generate and save an HTML report to a file
hardinfo -l                           # list all available report modules/categories
hardinfo -m devices.so -r             # run only a specific module's report (e.g., devices)
```

**What it aggregates in one view:**
- Processor details (equivalent to `/proc/cpuinfo` summarized)
- Memory summary (equivalent to `free` + partial `dmidecode`)
- Storage devices and partitions
- Network interfaces
- Installed software/package list
- Optional **benchmark suite**: CPU Blowfish, CPU CryptoHash, FPU FFT, etc., useful for comparing systems

> ⚠️ **Important Points to Remember:**
> - `hardinfo` is a **wrapper/aggregator** — it does not replace deep tools like `dmidecode` or `iostat` for advanced diagnostics; it's best used for a **quick, shareable overview report**.
> - The **benchmark scores** it generates are only meaningful for **relative comparison** between machines running `hardinfo` itself — they are not universal industry benchmarks.
> - Some distros ship the actively-maintained fork **`hardinfo2`** — if `hardinfo` is unavailable in your package manager, search for `hardinfo2`.

**📚 Source:** [hardinfo2 GitHub repository](https://github.com/hardinfo2/hardinfo2) · `man hardinfo`

---

## 10. Master Summary Table

| Hardware Category | Command(s) | Data Source | Root Needed? | Key Takeaway |
|---|---|---|:---:|---|
| **CPU** | `hwinfo --cpu` | Live hardware probe | Recommended | Universal probing, works even without loaded driver |
| | `sudo lshw -class cpu` | DMI + kernel | ✅ Yes | Structured tree, exportable as HTML/XML |
| | `cat /proc/cpuinfo` | `/proc` (kernel) | ❌ No | Per-logical-core raw data; check `flags` for `vmx`/`svm` |
| | `lscpu` | Parses `/proc/cpuinfo` | ❌ No | Clean architecture summary |
| **Graphics** | `sudo lshw -c display` | DMI + kernel | ✅ Yes | Shows bound driver (`amdgpu`, `nvidia`, etc.) |
| | `clinfo` | OpenCL ICD | ❌ No | GPU/CPU compute capability; 0 platforms = missing driver |
| | `lspci \| grep vga` | PCI bus | ❌ No | Fastest one-liner GPU ID |
| **Storage** | `cat /proc/partitions` | `/proc` (kernel) | ❌ No | Raw block device list, no filesystem info |
| | `lsblk -f` | `/sys` (sysfs) | ❌ No | Tree view + filesystem type + UUID |
| | `df -h` | Filesystem superblock | ❌ No | Space usage per mount; watch inodes with `-i` |
| | `sudo hdparm -Tt` | Direct disk I/O | ✅ Yes | Cache vs. real disk read speed; SATA/PATA only |
| | `iostat -dx` | `sysstat` kernel stats | ❌ No | Live I/O bottleneck detection via `%util`/`await` |
| **Memory** | `free -h` | `/proc/meminfo` | ❌ No | Watch `available`, not `free` |
| | `sudo dmidecode --type memory` | DMI/SMBIOS | ✅ Yes | Physical DIMM specs + empty slots; unreliable in VMs |
| **Network** | `lspci \| grep net` | PCI bus | ❌ No | NIC hardware identification |
| | `ip addr` / `ip route` | Kernel netlink | ❌ No (config: ✅) | Modern standard, replaces `ifconfig`/`route` |
| | `ifconfig` (legacy) | Kernel (via net-tools) | ❌ No (config: ✅) | Deprecated; not installed by default on newer distros |
| **Battery** | `upower -i <device>` | UPower D-Bus | ❌ No | Health = energy-full ÷ energy-full-design |
| **All-in-one** | `hwinfo`, `sudo lshw`, `hardinfo` | Multiple | Mixed | Full-system inventory/reports |

---

## 11. Cheat Sheet (Quick Reference)

```text
╔══════════════════════════════════════════════════════════════════════╗
║                     LINUX HARDWARE CHEAT SHEET                        ║
╠══════════════════════════════════════════════════════════════════════╣
║ GENERAL / ALL HARDWARE                                                ║
║   hwinfo --short              Quick overview of all hardware          ║
║   sudo lshw -short            Tree-style hardware summary             ║
║   sudo lshw -html > r.html    Exportable HTML hardware report         ║
║   hardinfo -r -f html -o r.html   GUI-style aggregated report          ║
║                                                                        ║
║ CPU                                                                   ║
║   cat /proc/cpuinfo           Full raw per-core CPU details           ║
║   lscpu                       Clean CPU architecture summary          ║
║   nproc                       Logical CPU (thread) count              ║
║   grep flags /proc/cpuinfo | grep -Eo 'vmx|svm'   Virtualization check ║
║                                                                        ║
║ GRAPHICS                                                              ║
║   sudo lshw -c display        GPU + bound driver info                 ║
║   lspci | grep -i vga         Quick GPU identification                ║
║   lspci -k | grep -A3 vga     Show which driver is bound              ║
║   clinfo -l                   OpenCL compute device short list        ║
║                                                                        ║
║ STORAGE                                                               ║
║   lsblk -f                    Disk/partition tree + FS type + UUID    ║
║   cat /proc/partitions        Raw kernel partition table              ║
║   df -h                       Free/used space per mount               ║
║   df -i                       Inode usage (space ≠ inode availability)║
║   sudo hdparm -Tt /dev/sdX    Disk read speed benchmark                ║
║   iostat -dx /dev/sdX 2 5     Live disk I/O stats, 5x every 2s        ║
║                                                                        ║
║ MEMORY                                                                ║
║   free -h                     RAM/swap snapshot (check "available")   ║
║   sudo dmidecode --type memory   Physical DIMM slot + empty slot info  ║
║                                                                        ║
║ NETWORK                                                               ║
║   lspci | grep -i net         Network adapter hardware                ║
║   ip addr show                Modern interface/IP info                ║
║   ip route show               Routing table                           ║
║   ifconfig                    Legacy interface info (deprecated)      ║
║                                                                        ║
║ BATTERY/POWER                                                         ║
║   upower -e                   List power devices                      ║
║   upower -i <device_path>     Battery charge/health details           ║
╚══════════════════════════════════════════════════════════════════════╝
```

**Golden Rules to Remember:**
1. `/proc` and `/sys` = **live kernel data**, always accurate, no install needed.
2. Firmware/hardware-level tools (`lshw`, `dmidecode`, `hdparm`) need **`sudo`** for complete, accurate output.
3. `ifconfig` → legacy; **`ip`** is the modern standard — know both, but default to `ip` in practice.
4. `lsblk`/`df -h` = **logical/filesystem view**; `dmidecode`/`fdisk` = **physical hardware view**.
5. `iostat`/`hdparm` = **live performance benchmarking**, not just static inventory listing.
6. For `free`, always trust the **`available`** column — not `free` — when judging real memory pressure.
7. Battery health = `energy-full ÷ energy-full-design × 100`.

---

## 12. Exam-Style Q&A

**Q1. Which command shows OpenCL-capable devices on a system, and what does "0 platforms" typically indicate?**
> `clinfo` — "0 platforms" usually means the vendor GPU driver or ICD loader is missing.

**Q2. What is the modern replacement for `ifconfig`, and which package provides it?**
> The `ip` command, from the `iproute2` package.

**Q3. Which command distinguishes CPU/cache-bound reads from truly disk-bound reads, and how?**
> `sudo hdparm -Tt /dev/sdX` — `-T` tests cached (RAM/CPU) reads; `-t` tests buffered (actual) disk reads.

**Q4. How can you check for empty/unused RAM slots on a motherboard, and why might this fail in a VM?**
> `sudo dmidecode --type memory` — empty slots show "No Module Installed". It's unreliable in VMs because hypervisors often expose fake/minimal SMBIOS data.

**Q5. Which command gives a live, continuously refreshing view of disk I/O load, and which column indicates a bottleneck?**
> `iostat -dx /dev/sdX [interval] [count]` — the `%util` column (near 100%) indicates the device is the bottleneck.

**Q6. Name two/three tools that give a full, structured hardware overview of the entire machine.**
> `lshw`, `hwinfo`, and `hardinfo` (GUI/report-based).

**Q7. What's the key structural difference between `/proc/cpuinfo` and `lscpu`?**
> `/proc/cpuinfo` lists one raw block per **logical** core (kernel-generated); `lscpu` parses and presents that same data as a clean, aggregated architecture summary (sockets, cores, threads).

**Q8. Which package must be installed to get the `lspci` command?**
> `pciutils`.

**Q9. How does `upower` calculate/imply battery health degradation?**
> By comparing `energy-full` (current max charge capacity) to `energy-full-design` (original factory capacity) — a ratio below 100% indicates wear.

**Q10. Which command shows mounted filesystem space usage in human-readable form, and what's the equally important but different `-i` flag for?**
> `df -h` for space; `df -i` shows **inode** usage — a filesystem can be "full" on inodes even with free space remaining.

**Q11. Why should the FIRST sample from `iostat -dx 2 5` generally be ignored?**
> The first report shows **averages since system boot**, not current live activity — only the second and later samples reflect the actual current state.

**Q12. On a 6-core, 12-thread CPU, how many `processor` entries will `/proc/cpuinfo` show, and why?**
> 12 — because `/proc/cpuinfo` enumerates **logical** cores (including SMT/Hyper-Threading siblings), not physical cores.

---

## 13. Extended Flag & Type Reference

This section documents the **full option/type space** for the tools with the largest flag sets — `lshw`, `dmidecode`, and `ip` — so you have complete reference, not a partial subset.

### 13.1 `lshw -class` — full list of hardware classes

```bash
sudo lshw -class <CLASSNAME>
```

| Class name | Shows |
|---|---|
| `system` | Overall system: manufacturer, product name, serial |
| `bus` | Motherboard/system bus |
| `memory` | RAM, cache, ROM, BIOS memory regions |
| `processor` | CPU(s) |
| `bridge` | PCI/ISA bridges connecting buses |
| `display` | Graphics cards |
| `input` | Keyboard, mouse, touchpad |
| `printer` | Printers |
| `multimedia` | Sound cards, capture devices |
| `communication` | Modems, serial ports |
| `network` | Ethernet, Wi-Fi adapters |
| `disk` | Hard drives, SSDs, optical drives |
| `storage` | Storage controllers (SATA, NVMe, RAID controllers) |
| `power` | Battery, power supply |
| `volume` | Filesystem/partition volumes |
| `generic` | Anything not matching another class (e.g., some USB devices) |

**Extra practical `lshw` flags:**
```bash
sudo lshw -businfo           # show bus info (PCI/USB address) for every device, compact table
sudo lshw -numeric           # show numeric vendor/device IDs alongside names
sudo lshw -quiet             # suppress warnings (useful when scripting/piping output)
sudo lshw -json > report.json    # JSON export (newer lshw versions) — best for scripts
```

---

### 13.2 `dmidecode --type` — full list of SMBIOS types

`dmidecode` types correspond directly to the numbered structures in the **DMTF SMBIOS specification**. The most exam/practically relevant:

| Type # | Keyword (`--type X`) | Shows |
|---|---|---|
| 0 | `bios` | BIOS/UEFI vendor, version, release date, ROM size |
| 1 | `system` | System manufacturer, product name, UUID, serial |
| 2 | `baseboard` | Motherboard manufacturer, model, serial |
| 3 | `chassis` | Case type (laptop, desktop, server), manufacturer |
| 4 | `processor` | CPU socket, manufacturer, max speed, voltage |
| 7 | `cache` | L1/L2/L3 cache size, speed, associativity |
| 8 | `connector` | Physical port connectors (USB, serial, etc.) |
| 9 | `slot` | Expansion slots (PCIe x16, x1, etc.) — populated or empty |
| 11 | — | OEM strings |
| 13 | — | BIOS language info |
| 16 | `memory` (array) | Physical Memory Array — total slots, max capacity |
| 17 | `memory` (device) | Individual DIMM details (the one used earlier in this guide) |
| 19 | — | Memory address mapping |
| 22 | — | Portable battery (laptop battery hardware details — separate from `upower`!) |
| 32 | — | System boot information |
| 41 | — | Onboard device information |

**Examples using additional types:**
```bash
sudo dmidecode --type bios          # BIOS/UEFI details
sudo dmidecode --type system        # system serial/UUID (asset tagging)
sudo dmidecode --type baseboard     # motherboard model
sudo dmidecode --type chassis       # chassis/form-factor type
sudo dmidecode --type processor     # CPU socket + rated max speed from firmware's view
sudo dmidecode --type cache         # cache hierarchy sizes from firmware
sudo dmidecode --type slot          # PCIe expansion slots — see which are free
sudo dmidecode --type 22            # laptop battery hardware info (design voltage, chemistry)
sudo dmidecode -t 0 -t 1 -t 2        # combine multiple types in one call
sudo dmidecode --list-types          # print every type number/keyword dmidecode supports
```

> ⚠️ **Remember:** Type 22 (`Portable Battery`) is a **static, firmware-reported hardware spec** (design voltage, chemistry, capacity) — it does NOT update live. For **live** charge/health/state, you still need `upower`. Don't confuse the two in an exam answer.

---

### 13.3 `ip` — full subcommand reference

`ip` is organized into **objects** (`addr`, `link`, `route`, `neigh`, etc.), each with its own verbs (`show`, `add`, `del`, `set`).

| Object | Purpose | Common commands |
|---|---|---|
| `addr` (`a`) | IP addresses | `ip addr show`, `ip addr add 10.0.0.5/24 dev eth0`, `ip addr del ...` |
| `link` (`l`) | Interface state/properties | `ip link show`, `ip link set eth0 up/down`, `ip link set eth0 mtu 1400` |
| `route` (`r`) | Routing table | `ip route show`, `ip route add default via 192.168.1.1`, `ip route del ...` |
| `neigh` (`n`) | ARP/neighbor cache | `ip neigh show`, `ip neigh flush all` |
| `rule` | Policy routing rules | `ip rule show` |
| `tunnel` | Tunnel interfaces (GRE, IPIP) | `ip tunnel show` |
| `netns` | Network namespaces | `ip netns list`, `ip netns add myns` |
| `-s` (stats) flag | Adds packet/byte/error counters | `ip -s link show eth0` |
| `-4` / `-6` | Restrict output to IPv4 or IPv6 only | `ip -4 addr show` |

**Extra practical examples:**
```bash
ip -4 addr show                     # IPv4 addresses only
ip -6 addr show                     # IPv6 addresses only
ip route get 8.8.8.8                # show which interface/route would be used to reach an IP
ip -s -s link show eth0             # extra-detailed statistics (double -s)
sudo ip addr flush dev eth0         # remove all IP addresses from an interface
sudo ip route add default via 192.168.1.1 dev eth0   # set default gateway manually
ip netns list                       # list network namespaces (containers/VPNs)
```

**📚 Source:** [DMTF SMBIOS Reference Specification (full type list, Table 0)](https://www.dmtf.org/standards/smbios) · [iproute2 documentation](https://wiki.linuxfoundation.org/networking/iproute2) · `man ip`, `man dmidecode`, `man lshw`

---

## 14. Common Errors & Troubleshooting

A command is only half-learned if you don't know what it looks like **when it fails**. This section documents the most common real-world error messages for each tool, why they happen, and the fix.

### 14.1 hwinfo / lshw

**Error:**
```
$ lshw
WARNING: you should run this program as super user.
```
**Cause:** Not running with `sudo` — DMI/BIOS-level data requires root.
**Fix:**
```bash
sudo lshw
```

**Error:**
```
$ lshw -class cpu
error: unrecognized hardware class "cpu"
```
**Cause:** Typo — the correct class is exactly `cpu` (lowercase) or `processor` in some versions; check with `lshw -class processor` if `cpu` fails on older versions.
**Fix:** Use `sudo lshw -short | grep -i processor` to confirm the exact class name your version expects.

---

### 14.2 clinfo

**Error:**
```
$ clinfo
Number of platforms: 0
```
**Cause:** No OpenCL ICD (Installable Client Driver) is installed for your GPU vendor — the GPU driver itself may be installed, but the OpenCL "bridge" library is missing.
**Fix (NVIDIA):**
```bash
sudo apt install nvidia-opencl-icd
```
**Fix (AMD):**
```bash
sudo apt install mesa-opencl-icd
```
**Fix (Intel):**
```bash
sudo apt install intel-opencl-icd
```

---

### 14.3 lsblk / df

**Error:**
```
$ df -h /mnt/mydrive
df: /mnt/mydrive: No such file or directory
```
**Cause:** The path doesn't exist or the drive isn't mounted yet.
**Fix:** Check `lsblk` first to confirm the device exists, then mount it:
```bash
lsblk
sudo mount /dev/sdb1 /mnt/mydrive
```

**Error:**
```
$ df -h
df: Warning: cannot read table of mounted file systems: No such file or directory
```
**Cause:** `/etc/mtab` or `/proc/mounts` is missing or corrupted (rare, usually in minimal containers/chroots).
**Fix:** Ensure `/proc` is mounted: `mount -t proc proc /proc` (inside a chroot/container).

---

### 14.4 hdparm

**Error:**
```
$ hdparm -Tt /dev/nvme0n1
/dev/nvme0n1:
 HDIO_DRIVE_CMD(identify) failed: Inappropriate ioctl for device
```
**Cause:** `hdparm` uses legacy ATA `ioctl` calls that **NVMe drives do not support** — this is expected behavior, not a bug.
**Fix:** Use `nvme-cli` instead for NVMe drives:
```bash
sudo apt install nvme-cli
sudo nvme smart-log /dev/nvme0
sudo nvme id-ctrl /dev/nvme0
```

**Error:**
```
$ hdparm -Tt /dev/sda
/dev/sda:
Permission denied
```
**Cause:** Missing `sudo`.
**Fix:** `sudo hdparm -Tt /dev/sda`

---

### 14.5 iostat

**Error:**
```
$ iostat
bash: iostat: command not found
```
**Cause:** `sysstat` package not installed (very common on fresh installs).
**Fix:**
```bash
sudo apt install sysstat
```

**Error:**
```
$ iostat -dx /dev/sdb
Device /dev/sdb not found in /proc/diskstats
```
**Cause:** Wrong device name (typo) or the drive was unmounted/removed since boot.
**Fix:** Confirm the exact name with `lsblk` or `cat /proc/partitions` first.

---

### 14.6 dmidecode

**Error:**
```
$ dmidecode --type memory
/dev/mem: Permission denied
```
**Cause:** Not run with `sudo` — reading `/dev/mem` to access the DMI table requires root.
**Fix:** `sudo dmidecode --type memory`

**Error / limitation (not a crash, but a trap):**
```
$ sudo dmidecode --type memory
# dmidecode 3.3
SMBIOS entry point missing / table is broken!
```
**Cause:** Running inside certain **virtual machines/containers** where the hypervisor doesn't expose a proper SMBIOS table (common in some cloud VM types or minimal containers).
**Fix:** There often isn't one — this is an environment limitation, not a command error. Rely on `free -h` and the cloud provider's own instance-spec documentation instead in these cases.

---

### 14.7 upower

**Error:**
```
$ upower -e
(no output / empty list)
```
**Cause:** Either running on a desktop with no battery (expected/normal), or the `upower` D-Bus daemon isn't running.
**Fix (check the daemon):**
```bash
systemctl status upower
sudo systemctl start upower
```

---

### 14.8 ip / ifconfig

**Error:**
```
$ ifconfig
bash: ifconfig: command not found
```
**Cause:** `net-tools` is not installed by default on many modern distros (e.g., current Ubuntu Server).
**Fix (two options):**
```bash
sudo apt install net-tools     # Option 1: install the legacy tool
ip addr show                   # Option 2 (recommended): just use the modern replacement
```

**Error:**
```
$ sudo ip addr add 192.168.1.50/24 dev eth0
RTNETLINK answers: File exists
```
**Cause:** That IP address (or an overlapping one) is already assigned to the interface.
**Fix:** Check current addresses first, remove if needed:
```bash
ip addr show eth0
sudo ip addr del 192.168.1.50/24 dev eth0
```

---

## 15. Automation: Building a Full Hardware-Audit Script

Putting it all together — a single Bash script that runs the key command from every category and writes a consolidated, timestamped report. This demonstrates how these individual commands are used **together** in real sysadmin practice, not just standalone.

```bash
#!/usr/bin/env bash
# =========================================================
#  hw_audit.sh — Full Hardware Audit Report Generator
#  Usage: sudo ./hw_audit.sh
#  Output: hw_report_<hostname>_<date>.txt
# =========================================================

set -euo pipefail

if [[ $EUID -ne 0 ]]; then
   echo "This script should be run with sudo for complete data." >&2
fi

OUTFILE="hw_report_$(hostname)_$(date +%Y%m%d_%H%M%S).txt"

{
  echo "======================================"
  echo " HARDWARE AUDIT REPORT"
  echo " Host: $(hostname)   Date: $(date)"
  echo "======================================"

  echo -e "\n--- CPU ---"
  lscpu
  echo -e "\n[Virtualization support check]"
  grep -Eo 'vmx|svm' /proc/cpuinfo | sort -u || echo "No hardware virtualization flags found."

  echo -e "\n--- GRAPHICS ---"
  lspci | grep -i vga
  command -v clinfo >/dev/null && clinfo -l || echo "clinfo not installed."

  echo -e "\n--- STORAGE ---"
  lsblk -f
  echo -e "\n[Disk space]"
  df -h --total
  echo -e "\n[Inode usage]"
  df -i

  echo -e "\n--- MEMORY ---"
  free -h
  if command -v dmidecode >/dev/null; then
    echo -e "\n[DIMM slot detail]"
    dmidecode --type memory | grep -E "Locator|Size|Speed|Manufacturer" || true
  fi

  echo -e "\n--- NETWORK ---"
  lspci | grep -i net
  ip -brief addr show

  echo -e "\n--- BATTERY / POWER ---"
  if command -v upower >/dev/null; then
    BATTERY=$(upower -e | grep BAT || true)
    if [[ -n "$BATTERY" ]]; then
      upower -i "$BATTERY" | grep -E "state|percentage|energy-full|energy-full-design"
    else
      echo "No battery detected (desktop system)."
    fi
  fi

  echo -e "\n======================================"
  echo " END OF REPORT"
  echo "======================================"

} | tee "$OUTFILE"

echo -e "\nReport saved to: $OUTFILE"
```

**What this script demonstrates:**

| Technique used | Why it matters |
|---|---|
| `set -euo pipefail` | Standard defensive Bash header — stops the script on errors instead of silently continuing |
| `command -v TOOL >/dev/null` | Checks a tool exists **before** calling it — avoids crashing on minimal systems missing optional packages (`clinfo`, `dmidecode`) |
| `\|\| true` / `\|\| echo "..."` | Gracefully handles a command that legitimately returns nothing (e.g., no battery, no virtualization flags) |
| `tee "$OUTFILE"` | Prints to screen **and** saves to a file simultaneously |
| `$(hostname)_$(date +...)` | Makes each report's filename unique and sortable |

> ⚠️ **Remember:** This script is a **template** — in real production use, sysadmins extend this pattern with `cron` scheduling, emailing the report (`mail -s "HW Audit" admin@example.com < report.txt`), or converting output to JSON (`lshw -json`) for ingestion into monitoring systems like Nagios, Zabbix, or a Grafana dashboard.

**📚 Source:** [Bash manual — Set Builtin](https://www.gnu.org/software/bash/manual/bash.html#The-Set-Builtin) · [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html)

---

## 16. Reference Sources

| Tool | Official / Authoritative Source |
|---|---|
| `hwinfo` | [github.com/openSUSE/hwinfo](https://github.com/openSUSE/hwinfo) |
| `lshw` | [ezix.org/project/wiki/HardwareLiSter](https://ezix.org/project/wiki/HardwareLiSter) |
| `/proc/cpuinfo` | [kernel.org — Linux Kernel /proc documentation](https://www.kernel.org/doc/html/latest/filesystems/proc.html) |
| `lscpu`, `lsblk`, `fdisk` | [github.com/util-linux/util-linux](https://github.com/util-linux/util-linux) |
| `clinfo` | [github.com/Oblomov/clinfo](https://github.com/Oblomov/clinfo) · [Khronos OpenCL Registry](https://www.khronos.org/opencl/) |
| `lspci`, `pciutils` | [mj.ucw.cz/sw/pciutils](https://mj.ucw.cz/sw/pciutils/) |
| `df`, `free` (coreutils portion) | [GNU Coreutils Manual](https://www.gnu.org/software/coreutils/manual/) |
| `free` (procps-ng) | [gitlab.com/procps-ng/procps](https://gitlab.com/procps-ng/procps) |
| `dmidecode` | [DMTF SMBIOS Specification](https://www.dmtf.org/standards/smbios) |
| `hdparm` | [sourceforge.net/projects/hdparm](https://sourceforge.net/projects/hdparm/) |
| `iostat`, `sysstat` | [github.com/sysstat/sysstat](https://github.com/sysstat/sysstat) |
| `ip`, `iproute2` | [wiki.linuxfoundation.org/networking/iproute2](https://wiki.linuxfoundation.org/networking/iproute2) |
| `ifconfig`, `net-tools` | Standard Linux man pages (`man ifconfig`) — legacy, no longer actively maintained upstream |
| `upower` | [upower.freedesktop.org](https://upower.freedesktop.org/) |
| `hardinfo` | [github.com/hardinfo2/hardinfo2](https://github.com/hardinfo2/hardinfo2) |
| DMI/SMBIOS general spec | [dmtf.org/standards/smbios](https://www.dmtf.org/standards/smbios) |

> 📌 **Note:** Every command listed in this guide also has an authoritative local reference on any Linux machine via its **man page** — run `man <command>` at any time (e.g., `man lshw`) for the definitive, version-matched documentation installed on that system.

---

*End of notes — compiled as a professional reference guide for exam preparation, system administration practice, and long-term technical documentation on Linux hardware diagnostics.*



