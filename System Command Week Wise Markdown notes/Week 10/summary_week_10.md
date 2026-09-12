# Week 10 Summary — Quick Revision Notes

Covers: **Git & Version Control** · **GitHub Workflow** · **Linux Hardware Inspection** · **LVM & RAID** · **Bash/Python Prompt Strings**

---

## 1. Version Control & Git Basics

**Version control** = tracking changes to files over time so you can see who changed what, revert bad changes, and let many people work on the same project.

| Type | How it works | Example | Weak point |
|---|---|---|---|
| Centralized (CVCS) | One central server holds all history; clients check out files | SVN, CVS | Single point of failure — server down = no history access |
| Distributed (DVCS) | Every user has a **full local copy** of the entire history | **Git**, Mercurial | None really — you can restore everything from any clone; works offline |

**Git vs GitHub — don't mix these up (common confusion):**

| | Git | GitHub |
|---|---|---|
| What it is | Version control software | Website hosting Git repos |
| Runs where | Your computer | The cloud |
| Needs internet | No | Yes |
| Adds | Nothing extra | Pull Requests, Issues, Actions (CI/CD), forks |

👉 *Edge case to remember:* Git works **completely offline** — GitHub is optional. Don't assume you need GitHub to use version control.

### First-time setup
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --list          # verify settings
```
⚠️ If you skip this, commits get attributed to a generic/blank identity — always configure identity **before** your first commit.

---

## 2. The Three-Stage Git Workflow

```
Working Directory  →  git add  →  Staging Area  →  git commit  →  Repository (.git)
```

| Command | What it does |
|---|---|
| `git init` | Turns a folder into a Git repo (creates hidden `.git/`) |
| `git status` | Shows changed/staged/untracked files — **always run this first** when confused |
| `git add <file>` / `git add .` | Stages changes |
| `git commit -m "msg"` | Saves a permanent snapshot |
| `git log` / `git log --oneline` | View history |
| `git diff` | Line-by-line unstaged changes |

### Undoing changes — pick the right one (edge case zone ⚠️)

| Command | Effect | Risk |
|---|---|---|
| `git restore <file>` | Discards unstaged changes | Destructive to uncommitted edits |
| `git restore --staged <file>` | Unstages a file, keeps the edits | Safe |
| `git reset --soft HEAD~1` | Undoes last commit, keeps changes staged | Safe |
| `git reset --hard HEAD~1` | Undoes last commit **and deletes the changes** | ⚠️ Destructive — no undo |
| `git revert <hash>` | Creates a *new* commit that reverses an old one | Safe for shared/pushed history |

👉 *Remember:* `reset --hard` rewrites history locally — never do this on commits already pushed and shared with others; use `revert` instead in that case.

---

## 3. Branching & Merging

A **branch** = an independent line of development (feature/fix) that doesn't touch `main` until merged.

```bash
git branch <name>          # create
git checkout <name>        # switch
git checkout -b <name>     # create + switch in one step
git switch <name>          # modern alternative to checkout
git branch                 # list branches
git branch -r               # list remote branches
```

**Merging:**
```bash
git checkout main
git merge <branch-name>
```

### Merge conflicts (the classic exam/edge case)
Happens only when **two branches changed the same lines of the same file**. Git can't auto-decide, so it inserts markers:
```
<<<<<<< HEAD
your version
=======
their version
>>>>>>> branch-name
```
**Fix:** manually edit the file → delete the `<<<<<<<`, `=======`, `>>>>>>>` markers → `git add <file>` → `git commit`.

👉 *Remember:* a conflict is a **content** collision, not a Git error — it's expected behavior, not something broken.

---

## 4. Remotes, Cloning & Syncing

| Term | Meaning |
|---|---|
| Remote | An online copy of the repo (e.g. on GitHub) |
| Clone | Full local copy of a remote repo, **including history** |
| Push | Upload local commits to remote |
| Pull | Download **and merge** remote changes |
| Fetch | Download changes **without** merging — "look but don't touch" |

```bash
git remote add origin <url>
git remote -v                 # verify remote URL
git clone <url>
git push -u origin main       # first push (sets upstream)
git push
git pull origin main
git fetch
```

👉 *Edge case:* **Cloning a private repo** requires authentication (PAT or SSH) — public repos clone with no login needed. `git push` failing with "Updates were rejected" usually means the remote has commits you don't have locally → run `git pull` first, then push again.

---

## 5. Authentication: PAT vs SSH

GitHub **no longer accepts plain account passwords** for Git operations over HTTPS — this is a major edge case people get tripped up on.

| Method | How | Use when |
|---|---|---|
| **PAT (Personal Access Token)** | GitHub → Settings → Developer settings → generate token, scope it, **copy it immediately (shown only once)** | HTTPS push/pull/clone |
| **SSH keys** | `ssh-keygen -t ed25519 -C "email"`, add public key to GitHub → Settings → SSH keys | Passwordless workflow, `git clone git@github.com:user/repo.git` |

Error `remote: Support for password authentication was removed` → you tried to use a password instead of a PAT.

**2FA (Two-Factor Authentication):** adds a code (authenticator app or SMS) on top of your password. Save recovery codes somewhere safe — losing both password and 2FA device can lock you out.

---

## 6. Repos, Naming Rules & Collaboration

**Repo naming rules:** lowercase, use `-` or `_` instead of spaces, no special characters (`#`, `%`, `&`, `@`), keep it short and descriptive. ✅ `weather-app` ❌ `My Project!! (final)`

| Term | Meaning |
|---|---|
| Fork | Your own copy of *someone else's* repo (for contributing to projects you don't own) |
| Pull Request (PR) | Proposal to merge your branch into another (usually `main`), with review |
| `.gitignore` | Lists files Git should never track (`.env`, `node_modules/`, `*.log`) |

**Typical real workflow:**
1. `git clone <url>`
2. `git checkout -b feature/x`
3. Edit → `git add .` → `git commit -m "..."`
4. `git push origin feature/x`
5. Open PR on GitHub → review → merge into `main`
6. `git checkout main && git pull` → delete old branch

👉 *Remember:* **Fork ≠ Clone.** Clone = local copy of a repo you already have access to. Fork = your own *remote* copy on GitHub of someone else's repo, used when you lack write access.

### Troubleshooting checklist (always in this order)
```bash
git status
git remote -v
git fetch
```

| Error | Cause | Fix |
|---|---|---|
| `fatal: not a git repository` | Not inside a tracked folder | `git init` or `cd` into the right folder |
| `Permission denied (publickey)` | SSH key issue | Recheck SSH setup |
| `git status` shows nothing but files missing | File matched in `.gitignore` | Check `.gitignore` |

---

## 7. Linux Hardware Inspection

Two data layers: **`/proc` & `/sys`** (live kernel data, no install needed) vs **userspace tools** (`lshw`, `hwinfo`, `dmidecode`) that read DMI/SMBIOS firmware tables for structured reports.

> ⚠️ **Golden rule:** Firmware/hardware-level tools (`lshw`, `dmidecode`, `hdparm`) need **`sudo`**; `/proc`/`/sys`-based commands (`cat /proc/cpuinfo`, `free`, `lsblk`) generally **don't**.

### Master command table

| Category | Command | Root? | Key takeaway / edge case |
|---|---|:---:|---|
| CPU | `hwinfo --cpu` | Recommended | Probes hardware **even without a loaded driver** |
| CPU | `sudo lshw -class cpu` | ✅ | Structured tree, exportable HTML/XML |
| CPU | `cat /proc/cpuinfo` | ❌ | Lists **logical** cores — a 6-core/12-thread CPU shows **12** entries, not 6 |
| CPU | `lscpu` | ❌ | Clean parsed summary of the above |
| Graphics | `sudo lshw -c display` | ✅ | Shows the bound driver (`amdgpu`, `nvidia`…); hybrid-graphics laptops show **two** display blocks |
| Graphics | `clinfo -l` | ❌ | "0 platforms" = missing GPU driver/ICD loader (common trap) |
| Storage | `cat /proc/partitions` | ❌ | Raw block devices only — **no filesystem info** |
| Storage | `lsblk -f` | ❌ | Adds filesystem type + UUID — the modern go-to |
| Storage | `df -h` | ❌ | Space per *filesystem*; `du` measures *files* — don't confuse the two |
| Storage | `df -i` | ❌ | Inode usage — a disk can show free space but still be "full" on inodes |
| Storage | `sudo hdparm -Tt` | ✅ | `-T` = cached (RAM) read speed, `-t` = real disk speed; **SATA/PATA only**, use `nvme-cli` for NVMe |
| Storage | `iostat -dx` | ❌ | `%util` near 100% = bottleneck; **ignore the first sample** (it's since-boot average) |
| Memory | `free -h` | ❌ | Trust **`available`**, not `free` — Linux uses spare RAM as reclaimable cache |
| Memory | `sudo dmidecode --type memory` | ✅ | Per-DIMM detail; often **broken/unreliable inside VMs/containers** (no real SMBIOS table) |
| Network | `ip addr` / `ip route` | ❌ (config: ✅) | Modern replacement for `ifconfig` |
| Network | `ifconfig` | ❌ | Legacy; often **not installed by default** on modern distros (`command not found`) |
| Battery | `upower -i <device>` | ❌ | Health = `energy-full ÷ energy-full-design × 100` |

### Package-to-command mapping (favorite trick question)

| Command | Package |
|---|---|
| `lspci` | pciutils |
| `lsblk`, `fdisk` | util-linux |
| `iostat` | sysstat |
| `ifconfig` | net-tools (legacy) |
| `ip` | iproute2 (usually pre-installed) |

### Common errors to remember
- `dmidecode: /dev/mem: Permission denied` → forgot `sudo`.
- `SMBIOS entry point missing/broken` inside a VM → not a bug, just a hypervisor limitation; rely on `free -h` instead.
- `ifconfig: command not found` → install `net-tools`, or better, just use `ip addr show`.

---

## 8. LVM & RAID

Two **complementary** (not competing) technologies: LVM = flexibility, RAID = redundancy/speed. In real deployments (NAS boxes, servers), **LVM is layered on top of a RAID array**.

### LVM — three-layer model
```
Physical Volume (PV)  →  Volume Group (VG)  →  Logical Volume (LV)
   (real disks)             (storage pool)        (usable "partition")
```

| Layer | Command to create | Notes |
|---|---|---|
| PV | `pvcreate /dev/sdb1` | Prepares a disk/partition for LVM |
| VG | `vgcreate my_vg /dev/sdb1 /dev/sdc1` | Pools PVs together |
| LV | `lvcreate -L 10G -n my_lv my_vg` | Carves out usable space |

Resize live: `lvextend -r -L +5G /dev/vg/lv` (the `-r` also resizes the filesystem in one step).
⚠️ **Shrinking is riskier** — must shrink the filesystem *first*, then the LV, and **XFS can only grow, never shrink**.

### RAID levels — the core comparison table

| RAID | Min disks | Redundancy | Read speed | Write speed | Usable capacity (n disks) |
|---|:---:|---|---|---|---|
| **RAID 0** (striping) | 2 | None — 1 disk failure = **all data lost** | n× | n× | 100% (n × size) |
| **RAID 1** (mirroring) | 2 | Survives n−1 failures | n× | Normal | 1 disk size only |
| **RAID 5** (distributed parity) | 3 | Survives 1 failure | n× | (n−1)× | (n−1) × size |
| **RAID 6** (dual distributed parity) | 4 | Survives 2 failures | n× | (n−2)× | (n−2) × size |

👉 *Key exam edge case:* **usable capacity is always ≤ raw capacity** whenever redundancy is involved — that gap is "the cost of safety." RAID 0 is the only mode with zero cost and zero protection.

**Parity** (RAID 5/6) is calculated with **XOR**, and is *distributed* (rotated) across all disks rather than stored on one dedicated disk — this avoids a write bottleneck and a single point of failure for the parity data itself. RAID 6's second parity block matters most on **large arrays**, where rebuild time after one failure is long enough that a second disk failing mid-rebuild becomes a real risk.

### Software RAID with `mdadm`
```bash
mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd
cat /proc/mdstat                 # check status
mdadm --detail /dev/md0
mdadm --manage /dev/md0 --fail /dev/sdc     # simulate failure
mdadm --manage /dev/md0 --add /dev/sde      # rebuild with replacement
```
⚠️ Don't forget `mdadm --detail --scan >> /etc/mdadm/mdadm.conf` — without saving the config, the array **won't reassemble automatically after reboot**.

### Choosing a RAID level

| Priority | Best fit |
|---|---|
| Max speed, don't care about data loss | RAID 0 |
| Max safety, small critical data (OS drive) | RAID 1 |
| Balanced general file server / NAS | RAID 5 |
| Large arrays, long rebuild windows | RAID 6 |
| Flexible resizing/snapshots | LVM (usually on top of RAID) |

---

## 9. Prompt Strings (Bash & Python)

A **prompt string** is a *dynamic template*, not static text — it's re-evaluated live every time the shell/interpreter redraws it.

### The four bash prompt variables

| Variable | Triggered when | Default |
|---|---|---|
| **PS1** | Ready for a new top-level command | `$` (or `\u@\h:\w\$`) |
| **PS2** | Command is syntactically incomplete (unclosed quote/bracket) | `>` |
| **PS3** | Inside a `select` menu loop | `#?` |
| **PS4** | Each traced line during `set -x` debugging | `+` |

👉 *Edge case:* `PS3` **cannot be read back with `echo $PS3`** the way the others can, and only applies *inside* a `select` loop.

### Most exam-relevant escape sequences

| Escape | Meaning | Escape | Meaning |
|---|---|---|---|
| `\u` | username | `\h` / `\H` | hostname (short/full) |
| `\w` / `\W` | full path / basename only | `\s` | shell name |
| `\t` / `\T` | time 24h / 12h `HH:MM:SS` | `\A` | time 24h `HH:MM` |
| `\d` | date "Day Mon DD" | `\D{fmt}` | custom `strftime` date |
| `\#` | command # **this session only** | `\!` | command # from **persistent history file** |
| `\$` | `#` if root, else `$` | `\[ \] ` | wrap non-printing chars (e.g. color codes) |

👉 *Most confused pair:* `\#` resets each new terminal session; `\!` keeps counting across sessions forever. Always wrap ANSI color codes in `\[...\]` or the terminal miscounts line length and cursor position glitches.

### Making changes permanent — login vs non-login shells (classic trap)

| Shell type | Occurs when | Reads |
|---|---|---|
| Login shell | SSH login, `su -`, console login | `/etc/profile` → `~/.bash_profile` (or `.bash_login`/`.profile`) |
| Non-login interactive shell | Opening a new GUI terminal tab | `~/.bashrc` |

👉 *Edge case:* editing `PS1` in `.bash_profile` often "doesn't work" in a new terminal tab because tabs usually open a **non-login shell**, which reads `.bashrc` instead. Fix: have `.bash_profile` `source ~/.bashrc`, or just put prompt customizations directly in `.bashrc`, then run `source ~/.bashrc` to apply without restarting.

### Python prompts (`sys.ps1` / `sys.ps2`)
- Only **2** variables (vs bash's 4), and **no built-in escape sequences**.
- Only apply in **interactive REPL mode** — `sys.ps1` has zero effect when running `python script.py`.
- To make them dynamic (e.g. a live counter), assign an **object with a `__str__` method**, not a plain string — Python calls `str()` on it every redraw.
```python
class DynamicPrompt:
    def __str__(self):
        return "computed-each-time> "
sys.ps1 = DynamicPrompt()
```
- Persisting customizations across sessions uses `PYTHONSTARTUP` (Python's rough equivalent of `.bashrc`).

### Common prompt-string errors

| Symptom | Cause | Fix |
|---|---|---|
| Prompt shows literal `\u@\h` text | Escapes not expanded — shell isn't bash, or wrong quoting | Use **single quotes**: `PS1='\u@\h:\w\$ '` |
| Line-wrapping glitches after adding colors | Missing `\[ \]` around ANSI codes | Wrap every color sequence |
| PS1 change vanishes after closing terminal | Only set at the command line, not saved | Add to `~/.bashrc`, then `source ~/.bashrc` |
| `sys.ps1` seems to do nothing | Running as a `.py` script, not interactively | Run `python` or `python -i file.py` |

---

## 10. One-Line Takeaways

| Topic | Takeaway |
|---|---|
| Git/GitHub | Git tracks history locally in commits/branches; GitHub hosts that online + adds PRs, tokens, 2FA for secure collaboration |
| Hardware | `/proc`/`/sys` = live no-install data; `lshw`/`dmidecode`/`hdparm` need `sudo` for firmware-level detail |
| LVM/RAID | RAID = redundancy/speed via disk-level tricks (striping/mirroring/parity); LVM = flexible resizing on top of any storage, often stacked on RAID |
| Prompt Strings | Bash has 4 dynamic template variables (PS1–4) with rich escape sequences; Python has 2, with no built-ins — dynamism needs `__str__` |
