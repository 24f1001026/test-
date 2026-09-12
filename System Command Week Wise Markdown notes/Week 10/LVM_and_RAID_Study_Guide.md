# Managing Storage: LVM & RAID — Complete Study Guide

> A professional, exam-ready reference covering Logical Volume Management (LVM) and RAID (Redundant Array of Independent Disks) — concepts, architecture, commands, and practical examples.

---

## 📑 Table of Contents

1. [Introduction to Storage Management](#1-introduction-to-storage-management)
2. [Logical Volume Management (LVM)](#2-logical-volume-management-lvm)
   - [2.1 What is LVM?](#21-what-is-lvm)
   - [2.2 LVM Architecture](#22-lvm-architecture)
   - [2.3 LVM Components Explained](#23-lvm-components-explained)
   - [2.4 lvm2 Tools & Commands](#24-lvm2-tools--commands)
   - [2.5 Practical LVM Workflow Example](#25-practical-lvm-workflow-example)
   - [2.6 Resizing, Snapshots & Advanced Operations](#26-resizing-snapshots--advanced-operations)
   - [2.7 Advantages & Disadvantages of LVM](#27-advantages--disadvantages-of-lvm)
3. [RAID — Redundant Array of Independent Disks](#3-raid--redundant-array-of-independent-disks)
   - [3.1 What is RAID?](#31-what-is-raid)
   - [3.2 RAID Controllers: Hardware vs Software](#32-raid-controllers-hardware-vs-software)
   - [3.3 Key RAID Concepts: Striping, Mirroring, Parity](#33-key-raid-concepts-striping-mirroring-parity)
4. [RAID Modes in Detail](#4-raid-modes-in-detail)
   - [4.1 RAID 0 — Striping](#41-raid-0--striping)
   - [4.2 RAID 1 — Mirroring](#42-raid-1--mirroring)
   - [4.3 RAID 5 — Striping with Distributed Parity](#43-raid-5--striping-with-distributed-parity)
   - [4.4 RAID 6 — Striping with Dual Distributed Parity](#44-raid-6--striping-with-dual-distributed-parity)
   - [4.5 Comparison Table of RAID Modes](#45-comparison-table-of-raid-modes)
5. [Distributed Parity Explained](#5-distributed-parity-explained)
6. [Software RAID with mdadm — Practical Examples](#6-software-raid-with-mdadm--practical-examples)
7. [LVM on Top of RAID — Combining Both](#7-lvm-on-top-of-raid--combining-both)
8. [Usable Capacity vs Actual Capacity](#8-usable-capacity-vs-actual-capacity)
9. [Choosing the Right RAID Level](#9-choosing-the-right-raid-level)
10. [Summary](#10-summary)
11. [Cheat Sheet](#11-cheat-sheet)

---

## 1. Introduction to Storage Management

Modern systems (servers, NAS devices, workstations) rarely rely on a single raw disk partition for storage. Two major technologies make storage **flexible, scalable, fast, and fault-tolerant**:

| Technology | Primary Goal |
|---|---|
| **LVM** | Flexible, resizable pooling of storage space |
| **RAID** | Redundancy, speed, and/or increased capacity through multiple disks |

These two technologies are **complementary**, not competing — LVM is often layered *on top of* RAID arrays in real-world deployments (e.g., NAS boxes like Synology, enterprise servers).

---

## 2. Logical Volume Management (LVM)

### 2.1 What is LVM?

**LVM (Logical Volume Management)** is a storage virtualization technology in Linux that allows administrators to:

- Pool **multiple physical storage devices** (disks/partitions) into a **single logical volume**.
- Allocate, resize, move, and manage disk space **dynamically**, without needing to unmount the filesystem or reboot in most cases.
- Abstract the physical layout of disks from the logical view the operating system uses.

In simple terms: instead of a filesystem being tied to one fixed-size physical partition, LVM creates a flexible "pool" of storage that can grow, shrink, or span multiple physical disks.

### 2.2 LVM Architecture

LVM works in **three layers**, from physical hardware to usable volumes:

```
┌─────────────────────────────────────────────┐
│           Logical Volumes (LV)               │  ← what the OS/filesystem uses
├─────────────────────────────────────────────┤
│         Volume Group (VG) — the "pool"        │  ← combined storage pool
├─────────────────────────────────────────────┤
│   Physical Volumes (PV) — /dev/sda1, /dev/sdb │  ← actual disks/partitions
└─────────────────────────────────────────────┘
```

### 2.3 LVM Components Explained

| Component | Abbreviation | Description |
|---|---|---|
| **Physical Volume** | PV | A physical disk or partition initialized for use by LVM (e.g., `/dev/sdb1`) |
| **Volume Group** | VG | A pool created by combining one or more PVs — the "storage pool" |
| **Logical Volume** | LV | A virtual partition carved out of a VG — this is what gets formatted and mounted |
| **Physical Extent** | PE | Small fixed-size chunks (default 4 MiB) into which a PV is divided |
| **Logical Extent** | LE | Corresponding chunks that map to PEs, used to build an LV |

**Analogy:** Think of a VG as a big bucket of "storage bricks" (PEs). You can build LVs (rooms) by picking however many bricks you need, from any of the source disks (PVs) contributing to the bucket.

### 2.4 lvm2 Tools & Commands

`lvm2` is the toolset used to **create and manage virtual block devices from physical devices**. Below are the essential commands grouped by layer:

#### Physical Volume (PV) Commands
```bash
# Initialize a disk/partition for LVM use
pvcreate /dev/sdb1 /dev/sdc1

# Display PV information
pvdisplay

# Quick summary of all PVs
pvs

# Remove a PV (must not be part of a VG)
pvremove /dev/sdb1
```

#### Volume Group (VG) Commands
```bash
# Create a Volume Group from one or more PVs
vgcreate my_vg /dev/sdb1 /dev/sdc1

# Add a new PV to an existing VG (extend the pool)
vgextend my_vg /dev/sdd1

# Remove a PV from a VG
vgreduce my_vg /dev/sdd1

# Display VG details
vgdisplay

# Quick summary
vgs

# Remove a VG entirely
vgremove my_vg
```

#### Logical Volume (LV) Commands
```bash
# Create a Logical Volume of 10 GB from a VG
lvcreate -L 10G -n my_lv my_vg

# Create an LV using 100% of free space in the VG
lvcreate -l 100%FREE -n my_lv my_vg

# Display LV details
lvdisplay

# Quick summary
lvs

# Remove a Logical Volume
lvremove /dev/my_vg/my_lv
```

### 2.5 Practical LVM Workflow Example

**Scenario:** You have two new disks, `/dev/sdb` and `/dev/sdc`, and you want to combine them into one large 40GB storage volume for `/data`.

```bash
# Step 1: Create partitions (using fdisk or parted) - assume /dev/sdb1, /dev/sdc1 exist

# Step 2: Mark them as physical volumes
pvcreate /dev/sdb1 /dev/sdc1

# Step 3: Create a volume group combining both disks
vgcreate data_vg /dev/sdb1 /dev/sdc1

# Step 4: Create a logical volume using all available space
lvcreate -l 100%FREE -n data_lv data_vg

# Step 5: Format the logical volume with a filesystem
mkfs.ext4 /dev/data_vg/data_lv

# Step 6: Create a mount point and mount it
mkdir /data
mount /dev/data_vg/data_lv /data

# Step 7 (optional): Make it persistent across reboots
echo "/dev/data_vg/data_lv /data ext4 defaults 0 2" >> /etc/fstab
```

### 2.6 Resizing, Snapshots & Advanced Operations

One of LVM's biggest strengths is **online resizing**.

```bash
# Extend a Logical Volume by 5GB
lvextend -L +5G /dev/data_vg/data_lv

# Extend and automatically resize the ext4 filesystem in one step
lvextend -r -L +5G /dev/data_vg/data_lv

# Resize filesystem manually after lvextend (ext4 example)
resize2fs /dev/data_vg/data_lv

# Resize filesystem manually (XFS example — XFS can only grow, not shrink)
xfs_growfs /data

# Shrink a Logical Volume (must shrink filesystem FIRST, then the LV — risky, ext4 only)
umount /data
e2fsck -f /dev/data_vg/data_lv
resize2fs /dev/data_vg/data_lv 20G
lvreduce -L 20G /dev/data_vg/data_lv
```

**Snapshots** — point-in-time copies of an LV, useful for backups:

```bash
# Create a snapshot of an LV (reserves 2GB for tracking changes)
lvcreate -s -L 2G -n data_snap /dev/data_vg/data_lv

# Mount snapshot to inspect/backup its point-in-time state
mkdir /mnt/snap
mount /dev/data_vg/data_snap /mnt/snap

# Revert an LV to a snapshot state
lvconvert --merge /dev/data_vg/data_snap
```

### 2.7 Advantages & Disadvantages of LVM

| ✅ Advantages | ❌ Disadvantages |
|---|---|
| Dynamic resizing of volumes without downtime | Slight performance overhead vs raw partitions |
| Combine multiple disks into one large volume | Added complexity for beginners |
| Supports snapshots for backups | Snapshots can degrade performance if space runs low |
| Easy to add new disks to expand storage | Boot-loader/rescue complexity in some setups |
| Can move data between physical disks live (`pvmove`) | Not a replacement for RAID redundancy on its own |

---

## 3. RAID — Redundant Array of Independent Disks

### 3.1 What is RAID?

**RAID (Redundant Array of Independent Disks)** is a technology that **distributes data across multiple physical disks** to achieve one or more of the following goals:

- **Redundancy** — protect against data loss if a disk fails
- **Speed** — improve read/write performance
- **Increased capacity** — combine multiple disks into a larger logical volume

RAID is managed by a **RAID controller**, which can be either:

### 3.2 RAID Controllers: Hardware vs Software

| Type | Description | Pros | Cons |
|---|---|---|---|
| **Hardware RAID** | A dedicated physical controller card manages the array, independent of the OS | Offloads CPU, often has battery-backed cache, OS-independent | Expensive; vendor lock-in; controller failure can complicate recovery |
| **Software RAID** | The OS (e.g., via `mdadm` on Linux) manages the array using the host CPU | Free, flexible, portable across hardware | Uses host CPU cycles; slightly more overhead |

### 3.3 Key RAID Concepts: Striping, Mirroring, Parity

| Concept | Meaning |
|---|---|
| **Striping** | Splitting data into blocks and writing them across multiple disks simultaneously → improves speed |
| **Mirroring** | Writing identical copies of data to two or more disks → improves redundancy |
| **Parity** | A mathematically calculated value (using XOR) stored alongside data, used to reconstruct lost data if a disk fails |

---

## 4. RAID Modes in Detail

### 4.1 RAID 0 — Striping

- **Minimum drives:** 2
- **Description:** Data is split into blocks and striped (spread) evenly across all disks in the array — **no redundancy**.
- **Comment:** Speeds up read/write performance significantly, since multiple disks work in parallel. **If any one disk fails, all data in the array is lost.**

```
Disk 0: A1  A3  A5  A7
Disk 1: A2  A4  A6  A8
```

**Use case:** Scratch space, video editing caches, temporary high-speed storage where redundancy doesn't matter.

```bash
# Create RAID 0 with mdadm using two disks
mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/sdb /dev/sdc
```

### 4.2 RAID 1 — Mirroring

- **Minimum drives:** 2
- **Description:** Data is duplicated (mirrored) identically across all disks in the array.
- **Comment:** Read speed is **n times faster** (n = number of disks, since reads can be spread across mirrors); write speed is normal. Tolerates **n−1 drive failures** (i.e., with 2 disks, 1 can fail and data survives).

```
Disk 0: A1  A2  A3  A4
Disk 1: A1  A2  A3  A4   (identical copy)
```

**Use case:** Operating system drives, critical data where uptime and safety matter more than capacity.

```bash
# Create RAID 1 with mdadm using two disks
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
```

### 4.3 RAID 5 — Striping with Distributed Parity

- **Minimum drives:** 3
- **Description:** Data and parity information are both striped across all disks — parity is **distributed**, not stored on one dedicated disk.
- **Comment:** Tolerates **1 drive failure**. Read is **n times faster**; write is **n−1 times faster** (because a parity calculation is needed on write).

```
        Disk0  Disk1  Disk2  Disk3
Row A:   A1     A2     A3     Ap    (parity)
Row B:   B1     B2     Bp     B3
Row C:   C1     Cp     C2     C3
Row D:   Dp     D1     D2     D3
```
Notice how the parity block (`p`) rotates to a different disk each row — this is what "distributed" parity means.

**Use case:** File servers, general-purpose NAS storage balancing capacity, speed, and fault tolerance.

```bash
# Create RAID 5 with mdadm using four disks
mdadm --create /dev/md0 --level=5 --raid-devices=4 /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

### 4.4 RAID 6 — Striping with Dual Distributed Parity

- **Minimum drives:** 4
- **Description:** Similar to RAID 5, but uses **two independent parity blocks** distributed across the disks instead of one.
- **Comment:** Tolerates **2 simultaneous drive failures**. Read is **n times faster**; write is **n−2 times faster** (two parity calculations needed per write).

```
        Disk0  Disk1  Disk2  Disk3  Disk4
Row A:   A1     A2     A3     Ap     Aq
Row B:   B1     B2     Bp     Bq     B3
Row C:   C1     Cp     Cq     C2     C3
Row D:   Dp     Dq     D1     D2     D3
Row E:   Eq     E1     E2     E3     Ep
```

**Use case:** Large storage arrays (e.g., NAS with many drives) where the probability of a second disk failing during a rebuild is a real risk — RAID 6 protects against that.

```bash
# Create RAID 6 with mdadm using five disks
mdadm --create /dev/md0 --level=6 --raid-devices=5 /dev/sdb /dev/sdc /dev/sdd /dev/sde /dev/sdf
```

### 4.5 Comparison Table of RAID Modes

| RAID Mode | Min Drives | Description | Fault Tolerance | Read Speed | Write Speed | Usable Capacity |
|---|---|---|---|---|---|---|
| **RAID 0** | 2 | Striping | None (0 drives) | n× faster | n× faster | 100% (n × disk size) |
| **RAID 1** | 2 | Mirroring | n−1 drive failures | n× faster | Normal | 1 disk size (50% with 2 disks) |
| **RAID 5** | 3 | Striping + distributed parity | 1 drive failure | n× faster | (n−1)× faster | (n−1) × disk size |
| **RAID 6** | 4 | Striping + dual distributed parity | 2 drive failures | n× faster | (n−2)× faster | (n−2) × disk size |

> **Key exam point:** In every redundant RAID mode, **usable capacity is always less than actual (raw) capacity** — the difference is the "cost" of redundancy.

---

## 5. Distributed Parity Explained

**Parity** is extra data calculated from the actual data blocks (typically using the **XOR** logical operation) that allows the system to **reconstruct missing data** if a disk fails.

### How XOR Parity Works (Simplified Example)

Suppose you have 3 data disks with these values (in binary, for one block):
```
Disk A: 1 0 1 1
Disk B: 0 1 1 0
Disk C: 1 1 0 0
```

Parity (XOR of A, B, C):
```
Parity = A ⊕ B ⊕ C = 1 0 1 1 ⊕ 0 1 1 0 ⊕ 1 1 0 0 = 0 0 0 1
```

If **Disk B fails**, it can be recovered:
```
B = A ⊕ C ⊕ Parity = 1 0 1 1 ⊕ 1 1 0 0 ⊕ 0 0 0 1 = 0 1 1 0  ✓ (matches original B)
```

### Why "Distributed" Parity Matters

- In older RAID designs (e.g., RAID 3/4), parity was stored on **one dedicated disk**, creating a bottleneck (every write had to touch that disk) and a single point of failure for parity itself.
- **RAID 5 and RAID 6 distribute (rotate) parity blocks across all disks** in the array, so:
  - No single disk becomes a write bottleneck.
  - Failure of any one disk is equally recoverable, since parity isn't concentrated in one place.
- **RAID 6's dual parity** (using two independent parity calculations — typically XOR + Reed-Solomon coding) allows recovery even when **two disks fail simultaneously**, which is increasingly important as disk sizes grow (larger disks = longer rebuild times = higher chance of a second failure during rebuild).

---

## 6. Software RAID with mdadm — Practical Examples

`mdadm` is the standard Linux tool for managing software RAID arrays.

```bash
# Install mdadm (Debian/Ubuntu)
sudo apt install mdadm

# Install mdadm (RHEL/CentOS)
sudo yum install mdadm

# View status of all RAID arrays
cat /proc/mdstat

# Detailed info about a specific array
mdadm --detail /dev/md0

# Create a RAID 5 array with 3 disks
mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd

# Save the RAID configuration (important for persistence across reboots)
mdadm --detail --scan >> /etc/mdadm/mdadm.conf
update-initramfs -u

# Format and mount the array
mkfs.ext4 /dev/md0
mkdir /mnt/raid
mount /dev/md0 /mnt/raid

# Simulate a disk failure (for testing)
mdadm --manage /dev/md0 --fail /dev/sdc

# Remove the failed disk
mdadm --manage /dev/md0 --remove /dev/sdc

# Add a replacement disk to rebuild the array
mdadm --manage /dev/md0 --add /dev/sde

# Stop/disassemble an array
mdadm --stop /dev/md0
```

---

## 7. LVM on Top of RAID — Combining Both

In real-world production and NAS systems (e.g., Synology, QNAP, enterprise Linux servers), **RAID and LVM are used together**:

```
┌────────────────────────────────────────┐
│      Logical Volumes (flexible)          │  ← LVM layer (resizable volumes)
├────────────────────────────────────────┤
│      Volume Group                        │
├────────────────────────────────────────┤
│      Physical Volume (on top of RAID)    │
├────────────────────────────────────────┤
│      RAID Array (/dev/md0) — redundancy  │  ← RAID layer (fault tolerance)
├────────────────────────────────────────┤
│  Disk0   Disk1   Disk2   Disk3           │  ← Physical disks
└────────────────────────────────────────┘
```

**Why combine them?**
- **RAID** provides the fault tolerance and redundancy (protects against physical disk failure).
- **LVM** provides the flexibility to resize, snapshot, and manage volumes on top of that protected storage.

```bash
# Step 1: Create a RAID 5 array
mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd

# Step 2: Turn the RAID array into a Physical Volume
pvcreate /dev/md0

# Step 3: Create a Volume Group on top of the RAID array
vgcreate raid_vg /dev/md0

# Step 4: Create Logical Volumes as needed
lvcreate -L 200G -n backups_lv raid_vg
lvcreate -L 100G -n media_lv raid_vg

# Step 5: Format and mount
mkfs.ext4 /dev/raid_vg/backups_lv
mount /dev/raid_vg/backups_lv /mnt/backups
```

This is essentially how commercial NAS devices (like the Synology unit referenced in the source material) operate under the hood.

---

## 8. Usable Capacity vs Actual Capacity

A critical exam concept: **usable capacity is always less than or equal to actual (raw) capacity** whenever redundancy is involved.

| RAID Mode | 4 × 1TB Disks — Actual Capacity | Usable Capacity | "Cost" of Redundancy |
|---|---|---|---|
| RAID 0 | 4 TB | 4 TB | 0 TB (no redundancy) |
| RAID 1 | 4 TB | ~1 TB (mirrored pairs) | High (50%+ loss) |
| RAID 5 | 4 TB | 3 TB | 1 disk worth |
| RAID 6 | 4 TB | 2 TB | 2 disks worth |

**Formula reference:**
- RAID 0: Usable = n × disk size
- RAID 1: Usable = disk size (regardless of n, assuming simple mirroring)
- RAID 5: Usable = (n − 1) × disk size
- RAID 6: Usable = (n − 2) × disk size

---

## 9. Choosing the Right RAID Level

| Priority | Recommended RAID |
|---|---|
| Maximum speed, no concern for data loss | RAID 0 |
| Maximum safety for critical/boot data, small arrays | RAID 1 |
| Balanced capacity + redundancy + speed, general NAS/file server | RAID 5 |
| Large arrays, extra protection during long rebuilds | RAID 6 |
| Flexibility to resize/manage volumes dynamically | LVM (often layered on RAID) |

---

## 10. Summary

- **LVM (Logical Volume Management)** virtualizes storage by pooling multiple physical devices into **Physical Volumes → Volume Groups → Logical Volumes**, enabling dynamic resizing, snapshots, and flexible disk management via the **lvm2** toolset (`pvcreate`, `vgcreate`, `lvcreate`, etc.).
- **RAID (Redundant Array of Independent Disks)** distributes data across multiple disks to achieve **redundancy, speed, and/or capacity**, managed by a **hardware or software RAID controller**.
- **RAID 0** stripes data for speed but offers **zero redundancy**.
- **RAID 1** mirrors data for strong redundancy at the cost of capacity.
- **RAID 5** uses **distributed parity** across a minimum of 3 disks, tolerating **1 disk failure**.
- **RAID 6** extends this with **dual distributed parity** across a minimum of 4 disks, tolerating **2 simultaneous disk failures** — crucial for large arrays with long rebuild times.
- **Distributed parity** (via XOR calculations) allows lost data to be mathematically reconstructed without dedicating a single disk to parity, avoiding bottlenecks and single points of failure.
- In all redundant RAID modes, **usable capacity < actual raw capacity** — this trade-off is the "price" of fault tolerance.
- In production systems, **LVM is frequently layered on top of a RAID array**, combining RAID's fault tolerance with LVM's flexibility — this is exactly how devices like Synology NAS units are architected internally.

---

## 11. Cheat Sheet

### 🔑 Core Definitions
| Term | Definition |
|---|---|
| LVM | Pools multiple disks into one flexible, resizable logical volume |
| PV | Physical Volume — a disk/partition prepared for LVM |
| VG | Volume Group — pool combining PVs |
| LV | Logical Volume — usable "partition" carved from a VG |
| RAID | Distributes data across disks for redundancy/speed/capacity |
| Striping | Splitting data across disks for speed (no redundancy alone) |
| Mirroring | Duplicating data across disks for redundancy |
| Parity | Calculated recovery data (XOR) allowing reconstruction after failure |

### 🔑 RAID Quick Reference
| Mode | Min Disks | Fault Tolerance | Best For |
|---|---|---|---|
| RAID 0 | 2 | None | Speed only |
| RAID 1 | 2 | n−1 disks | Critical small data |
| RAID 5 | 3 | 1 disk | Balanced general use |
| RAID 6 | 4 | 2 disks | Large/critical arrays |

### 🔑 Essential LVM Commands
```bash
pvcreate /dev/sdX       # Create Physical Volume
vgcreate vg_name /dev/sdX   # Create Volume Group
lvcreate -L 10G -n lv_name vg_name   # Create Logical Volume
lvextend -r -L +5G /dev/vg_name/lv_name  # Extend LV + filesystem
lvs / vgs / pvs         # Quick status summaries
```

### 🔑 Essential mdadm (Software RAID) Commands
```bash
mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd
cat /proc/mdstat                     # Check RAID status
mdadm --detail /dev/md0              # Detailed array info
mdadm --manage /dev/md0 --fail /dev/sdX     # Simulate/mark failure
mdadm --manage /dev/md0 --add /dev/sdY      # Add replacement disk
```

### 🔑 Capacity Formulas
```
RAID 0 usable = n × disk_size
RAID 1 usable = disk_size
RAID 5 usable = (n - 1) × disk_size
RAID 6 usable = (n - 2) × disk_size
```

### 🔑 Exam-Style Quick Facts
- RAID 0 = **speed**, zero redundancy.
- RAID 1 = **mirroring**, read is n× faster.
- RAID 5 = **1 parity block**, needs ≥3 disks, survives 1 failure.
- RAID 6 = **2 parity blocks**, needs ≥4 disks, survives 2 failures.
- Usable capacity is **always less than actual capacity** in redundant modes.
- LVM ≠ RAID: LVM is about **flexibility**, RAID is about **redundancy/performance**. They solve different problems and are often used **together**.

---

*End of study guide — good luck with your exam preparation!*
