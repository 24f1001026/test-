# Week 2 Summary — Editors, Networking & SSH (Quick Revision)

Covers: Lectures 1–3 (Command Line Editors), Lecture 4 (Networking & SSH)

---

## 1. Categories of Text Editors

| Category | Description | Examples |
|---|---|---|
| Line Editors | One line at a time, no screen view | `ed` |
| Terminal Editors | Full-screen, interactive, inside a terminal | `vi`, `emacs`, `nano`, `pico` |
| GUI Editors | Mouse-driven, graphical | gedit, Sublime, Atom, kate |
| IDE | Full dev environment (edit+debug+build) | Eclipse, NetBeans |

⚠️ As you go Line → Terminal → GUI → IDE, tools get more visual but **line/terminal editors stay essential for SSH sessions**, where no GUI is available — this is why `vi`/`nano` matter even though they look "outdated."

**Common features across almost all editors:** scrolling/navigation, insert/replace/delete, cut-copy-paste, search-replace, syntax highlighting, key-maps/macros, plugins.

---

## 2. `ed` — The Original Line Editor

| Concept | Detail |
|---|---|
| What it is | Ancestor of `vi`, `ex`, `sed` — edits **one line at a time**, no visual display |
| Buffer | Temporary in-memory copy of the file being edited |
| Syntax | `[address[,address]] command [parameters]` |

**Address commands:**

| Symbol | Meaning |
|---|---|
| `.` | Current line |
| `$` | Last line |
| `%` | All lines |
| `+` / `-` | Next / previous line |
| `,` / `;` | Range separators |
| `/RE/` | Line matching a regex |

**Editing & file commands:**

| Command | Meaning |
|---|---|
| `p` | Print current line |
| `a` / `i` | Append after / insert before current line |
| `c` | Change (replace) current line |
| `d` | Delete current line |
| `s` | Search/replace with regex |
| `w` | Write buffer to file |
| `q` | Quit |
| `!command` | Run a shell command |

⚠️ Ending insert mode is easy to forget: a lone **`.` on its own line** ends append/insert — not `Esc` (that's a `vi` habit that doesn't apply here).

---

## 3. `ex` — Line Editor Behind `vi`

| Concept | Detail |
|---|---|
| Relation to `ed` | Very similar command set, but more powerful for scripting/batch edits |
| Relation to `vi` | **Every `:` command inside `vi` is actually an `ex` command** — this is the key exam link between the two |

| Command | Meaning |
|---|---|
| `:1,10d` | Delete lines 1–10 |
| `:%s/foo/bar/g` | Replace all `foo` with `bar` in the whole file |
| `:w` | Save |
| `:q` | Quit |

---

## 4. `nano` — Beginner-Friendly Full-Screen Editor

| Concept | Detail |
|---|---|
| Style | Full-screen, shows a help/shortcut bar at the bottom — no modes to remember |
| Open | `nano filename` |

| Category | Key Shortcuts |
|---|---|
| File handling | `Ctrl+S` save, `Ctrl+O` save-as, `Ctrl+X` exit |
| Editing | `Ctrl+K` cut line, `Ctrl+U` paste, `Alt+U`/`Alt+E` undo/redo |
| Search & replace | `Ctrl+W` forward search, `Ctrl+Q` backward search, `Alt+R` replace |
| Deletion | `Ctrl+H` char before, `Ctrl+D` char under cursor |
| Movement | `Ctrl+A`/`Ctrl+E` start/end of line, `Ctrl+P`/`Ctrl+N` line up/down |
| Info | `Ctrl+C` cursor position, `Ctrl+G` help |

⚠️ Note the **save vs exit split**: `Ctrl+O` writes the file but does **not** close nano; `Ctrl+X` closes nano but will prompt you to save first if there are unsaved changes — don't assume one key does both, unlike `vi`'s `:x`.

---

## 5. `vi` — The Modal Editor

| Concept | Detail |
|---|---|
| Key idea | **Modal** — same keys do different things depending on mode. This is the single biggest conceptual difference from `nano` (which has no modes) |

**Modes:**

| Mode | How to Enter | Purpose |
|---|---|---|
| Command Mode | Default, or press `Esc` from anywhere | Navigate, delete, copy, run commands |
| Insert Mode | `i` / `o` / `a` / `I` / `O` / `A` from Command Mode | Type new text |
| Ex (Last-line) Mode | `:` from Command Mode | Save/quit/search-replace (this is literally `ex`, see §3) |

⚠️ **The #1 beginner mistake**: typing normal text while accidentally in Command Mode (or vice versa) — always check your mode, and `Esc` is your safe "return to Command Mode" reflex before running any `:` command.

**Exiting vi:**

| Command | Effect |
|---|---|
| `:w` | Save only |
| `:q` | Quit — **fails if unsaved changes exist** |
| `:wq` / `:x` | Save and quit |
| `:q!` | Discard changes and quit forcibly |

⚠️ `:q` refusing to quit when you've made edits is expected behavior, not a bug — use `:q!` if you want to discard, or `:wq` to save.

**Movement & editing quick table:**

| Task | Keys | Note |
|---|---|---|
| Move by char | `h j k l` | Left/down/up/right — no mouse needed |
| Line start/end | `0` / `$` | — |
| Go to line N | `:n` or `nG` | `G` alone = last line |
| Replace 1 char | `r` | Stays in Command Mode |
| Replace until Esc | `R` | Switches to a replace-insert mode |
| Change word/line | `cw` / `cc` | Deletes then drops you into Insert Mode |
| Delete char/word/line | `x` / `dw` / `dd` | `Ndd` deletes N lines |
| Copy (yank) / paste | `yy` / `p` | Yank does **not** delete, unlike `dd` |
| Search | `/pattern` / `?pattern` | Forward / backward; `n`/`N` repeat |

⚠️ `dd` and `yy` both fill the **same buffer** — yanking after deleting overwrites what you cut, so paste immediately after `dd` if you want it back.

---

## 6. `emacs` — Non-Modal, Extensible Editor

| Concept | Detail |
|---|---|
| Key difference from `vi` | **Non-modal** — you type directly; commands are triggered by key combos, not by switching modes |
| Notation | `C-x` = Ctrl+x, `M-x` = Alt+x ("Meta") |

| Task | Keys |
|---|---|
| Move up/down/left/right | `C-p` / `C-n` / `C-b` / `C-f` |
| Line start/end | `C-a` / `C-e` |
| Save | `C-x C-s` |
| Suspend (keep running) | `C-z` |
| Quit fully | `C-x C-c` |
| Cut to end of line | `C-k` |
| Paste (yank) | `C-y` |
| Search forward/backward | `C-s` / `C-r` |

⚠️ `C-z` **suspends** emacs (it keeps running in the background) rather than closing it — easy to mistake for "exit," and you'll need `fg` in the shell to get back to it, unlike `C-x C-c` which actually terminates it.

---

## 7. Editor Comparison at a Glance

| Feature | `ed` | `ex` | `nano` | `vi` | `emacs` |
|---|---|---|---|---|---|
| Type | Line | Line | Terminal | Terminal | Terminal |
| Modal? | No | No | No | **Yes** | No |
| Beginner-friendly | ✗ | ✗ | ✅ | ✗ | ✗ |
| On-screen help | ✗ | ✗ | ✅ bottom bar | ✗ (`:help`) | ✅ `C-h` |
| Save | `w` | `:w` | `Ctrl+O` | `:w` | `C-x C-s` |
| Quit | `q` | `:q` | `Ctrl+X` | `:q`/`:q!` | `C-x C-c` |
| Best for | Scripted edits | Backbone of `vi` | Quick beginner edits | Fast editing once learned; near-universal on servers | Deep customization, programming |

⚠️ Exam angle: `ed`/`ex` look "useless" today but matter because **`vi`'s `:` commands are literally `ex` commands** — understanding one explains the other.

---

## 8. Public vs Private Networks

| Term | Meaning |
|---|---|
| Public Network | The open internet — globally routable IPs |
| Private Network | Internal/local network (e.g., LAN) — not directly routable on the internet |
| Gateway | Device (router) connecting a private network to another network |

⚠️ Private networks can be **nested** — a private network can contain smaller private networks, each behind its own gateway (e.g., a company gateway → department gateway → employee machines). Every device only reaches the internet **through its gateway**, never directly.

---

## 9. IPv4 Addressing

| Range Type | CIDR / Range | Notes |
|---|---|---|
| Localhost | `127.0.0.0/8` (e.g. `127.0.0.1`) | A machine referring to itself (loopback) |
| Private — Class A | `10.0.0.0/8` (`10.0.0.0–10.255.255.255`) | ~16.7M addresses; large orgs/ISPs |
| Private — Class B | `172.16.0.0/12` (`172.16.0.0–172.31.255.255`) | ~1M addresses; medium orgs |
| Private — Class C | `192.168.0.0/16` (`192.168.0.0–192.168.255.255`) | ~65K addresses; home/small office routers |
| Public | Globally unique, ICANN/IANA-assigned | Routable across the internet |

⚠️ `172.16.x.x` is Class B — people often misremember it as `172.x.x.x` for any second octet, but the valid Class B private range is strictly **172.16.0.0 to 172.31.255.255**, not all of `172.*`.

---

## 10. Ports

A **port + IP address** together uniquely identify a service on a machine. One IP can have many open ports simultaneously, each for a different service.

| Port | Service | Purpose |
|---|---|---|
| 21 | FTP | File transfer |
| 22 | SSH | Secure remote login |
| 25 | SMTP | Sending email between servers |
| 80 | HTTP | Unencrypted web traffic |
| 443 | HTTPS | Encrypted (TLS/SSL) web traffic |
| 631 | CUPS | Network printing |
| 3306 | MySQL | Database connections |

⚠️ Classic exam trap: matching port → service backwards (e.g., confusing 80 vs 443, or 21 vs 22). Memorize them as pairs, not just numbers.

---

## 11. Ways to Gain Remote Access

| Method | Best For | Example Tools |
|---|---|---|
| VPN | Accessing an entire private network securely | OpenVPN, Cisco AnyConnect |
| SSH Tunneling | Securely accessing one specific service | `ssh -L`, `ssh -R` |
| Remote Desktop | Full graphical session needed | x2go, RDP, PCoIP |
| Desktop over Browser | No client software allowed/available | Apache Guacamole |
| Commercial tools | Quick ad-hoc support sessions | TeamViewer, AnyDesk, Zoho Assist |

⚠️ Know **when** to use which — VPN gives access to a whole network, SSH tunneling to one service only; picking the wrong one is a common conceptual exam question.

---

## 12. SSH — Secure Shell

| Concept | Detail |
|---|---|
| What | Cryptographic protocol for secure remote login & file transfer |
| Default port | **22** |
| Syntax | `ssh [options] username@hostname_or_IP` |

**Common options:**

| Option | Meaning |
|---|---|
| `-p <port>` | Connect on a non-default port |
| `-v` | Verbose (debugging) |
| `-i <keyfile>` | Use a specific private key |
| `-X` | X11 forwarding (run remote GUI apps) |
| `-L` / `-R` | Local / remote port forwarding (tunneling) |

**Key-based authentication (avoids typing password):**

| Step | Command |
|---|---|
| 1. Generate key pair | `ssh-keygen -t rsa -b 4096 -C "email"` |
| 2. Copy public key to server | `ssh-copy-id user@host` |
| 3. Login (now passwordless) | `ssh user@host` |

⚠️ `ssh-keygen` creates **two** files — `id_rsa` (private, never share) and `id_rsa.pub` (public, safe to distribute). Only the `.pub` file goes to the remote server via `ssh-copy-id`.

**SCP (secure copy):**

| Direction | Example |
|---|---|
| Local → Remote | `scp file.pdf user@host:/path/` |
| Remote → Local | `scp user@host:/path/file.csv ./` |
| Directory (recursive) | `scp -r ./folder user@host:/path/` |

⚠️ Just like `cp`, `scp` needs `-r` for directories — forgetting it is the same classic mistake as with `cp`.

**SSH Tunneling (port forwarding):**

| Type | Syntax | Use case |
|---|---|---|
| Local (`-L`) | `ssh -L local_port:target_host:target_port user@server` | Reach a remote-only service (e.g., a DB) via a local port |
| Remote (`-R`) | `ssh -R remote_port:local_host:local_port user@server` | Expose a local service to the remote server |

⚠️ `-L` and `-R` are opposite directions — `-L` forwards **your local port outward**, `-R` forwards **the remote server's port back to your machine**. Easy to swap them by mistake.

---

## 13. Firewalls & Server Protection

| Firewall Check Point | What to Verify |
|---|---|
| Local machine | Are the needed ports open/listening? |
| Remote machine | Is the target port open and the service running? |
| Every hop in between | Each router/gateway can apply its own rules |

⚠️ A connection fails if **any single hop** blocks the port — troubleshooting means checking firewalls at multiple levels, not just your own machine or just the server.

**Layered server protection (defense-in-depth):**

```
Anonymous Users → Network Firewall → Web Application Filter (WAF) → Server
```

| Layer | Role |
|---|---|
| Network Firewall | Blocks unwanted ports/IPs/protocols |
| WAF | Blocks malicious app-layer traffic (SQLi, XSS) |
| Server | Hosts the actual application |

⚠️ Order matters — traffic hits the **network firewall first**, then the WAF, and only then the server; don't reverse this in an exam diagram.

---

## 14. SELinux (Security-Enhanced Linux)

| Concept | Detail |
|---|---|
| What | Kernel module adding access control **beyond** standard Unix permissions |
| Where | Standard on CentOS, Fedora, RHEL, SuSE; available on Ubuntu |
| Model | Role-Based Access Control (RBAC) |

**RBAC components:**

| Component | Example | Meaning |
|---|---|---|
| User | `unconfined_u` | SELinux user identity |
| Role | `object_r` | What the user/process may do |
| Type | `user_home_t` | Core access-control mechanism (Type Enforcement) |
| Level | `s0` | Sensitivity level (Multi-Level Security) |

**Modes:**

| Mode | Behavior |
|---|---|
| Disabled | No enforcement at all |
| Enforcing | Denies **and** logs unauthorized actions |
| Permissive | Only **logs** violations, doesn't block — used for testing |

⚠️ **Permissive ≠ Disabled** — permissive mode still records violations in logs even though it doesn't block them; it's a testing/diagnostic mode, not "off."

| Command | Purpose |
|---|---|
| `ls -lZ` | Show SELinux context of files |
| `ps -eZ` | Show SELinux context of processes |
| `getenforce` | Check current mode |
| `setenforce 0` / `1` | Set permissive / enforcing |
| `semanage` | Manage SELinux policy |
| `restorecon` | Restore default context after moving/copying a file |

⚠️ Moving/copying files can leave them with the **wrong SELinux context**, silently breaking access even though normal Unix permissions look fine — always `restorecon` after relocating files into a service directory (e.g., `/var/www/html`).

---

## 15. Network Diagnostic Tools

| Tool | Purpose | Example |
|---|---|---|
| `ping` | Check if a host is reachable | `ping -c 4 192.168.1.10` |
| `traceroute` | Hop-by-hop path timing | `traceroute -n 8.8.8.8` |
| `nslookup` | IP ↔ domain resolution | `nslookup google.com` |
| `dig` | Detailed DNS lookup | `dig example.com MX` |
| `netstat` | Show active connections | `netstat -tulpn` |
| `whois` | Domain ownership info | `whois example.com` |
| `nmap` ⚠️ | Port scanner | `nmap -p 1-1000 192.168.1.10` |
| `wireshark` ⚠️ | Packet capture/analysis (GUI) | `wireshark` |
| mxtoolbox.com | Web-based public accessibility/DNS check | Browser only, no CLI |

⚠️ `nmap` and `wireshark` are flagged specifically because **unauthorized scanning or packet sniffing can be illegal** — only ever use them on systems/networks you own or are explicitly authorized to test. This is a direct exam point, not just a technical caveat.

---

## 16. High Performance Computing (HPC) Access

| Point | Detail |
|---|---|
| Access method | Almost always via **SSH** |
| Long jobs | Submitted to a **job scheduler** (e.g., SLURM, PBS) — never run interactively |
| Large data | Processed **remotely on the cluster** first, only final results transferred locally |
| Why | Bandwidth/data-transfer costs and limits |
| Interface | Mostly command-line only — no GUI on most clusters |

⚠️ The recurring exam trap: assuming you SSH in and run your job directly — in HPC, you SSH into a **login node** and then **submit** the job to a scheduler; running heavy jobs directly on the login node is bad practice.

Example flow: `ssh` in → `sbatch job.sh` (submit) → `squeue -u user` (check status) → `scp` only the final results back.

---

## 17. Master Quick-Reference Table

| Topic | Key Fact | Watch out for |
|---|---|---|
| Editor categories | Line → Terminal → GUI → IDE | Line/terminal editors stay essential for SSH sessions |
| `ed` | One line at a time, `[addr]cmd[params]` | Insert mode ends with a lone `.`, not `Esc` |
| `ex` | Backbone of `vi`'s `:` commands | Same command names as `ed` but more scripting power |
| `nano` | Beginner-friendly, all keys shown | `Ctrl+O` saves but doesn't exit; `Ctrl+X` exits |
| `vi` | Modal: Command / Insert / Ex | `:q` fails with unsaved changes — use `:q!` or `:wq` |
| `emacs` | Non-modal, `C-x`/`M-x` combos | `C-z` suspends, doesn't quit — `C-x C-c` does |
| Public vs Private network | Private connects via gateway | Networks can nest with multiple gateways |
| IPv4 private ranges | 10.x (A), 172.16–172.31.x (B), 192.168.x (C) | Class B is NOT all of `172.*` |
| Ports | 21 FTP, 22 SSH, 25 SMTP, 80 HTTP, 443 HTTPS, 3306 MySQL | Common port↔service mix-ups |
| Remote access methods | VPN (whole network) vs SSH tunnel (one service) | Pick based on scope needed |
| SSH keys | `id_rsa` private, `id_rsa.pub` public | Only share the `.pub` file |
| `scp -r` | Needed for directories | Same rule as `cp -r` |
| `-L` vs `-R` forwarding | Local forwards out, Remote forwards back | Easy to swap by mistake |
| Firewall | Checked at every hop | One blocking hop = failed connection |
| Server protection order | Firewall → WAF → Server | Don't reverse the order |
| SELinux modes | Disabled / Permissive / Enforcing | Permissive still logs, isn't "off" |
| `restorecon` | Fixes SELinux context after moving files | Unix perms can look fine while SELinux still blocks |
| `nmap`/`wireshark` | Powerful diagnostic tools | Only use with authorization — legal risk otherwise |
| HPC | SSH → submit to scheduler → don't run heavy jobs on login node | Process large data remotely first |

---

*Prepared as a consolidated, table-based revision summary for Week 2 (Lectures 1–4).*
