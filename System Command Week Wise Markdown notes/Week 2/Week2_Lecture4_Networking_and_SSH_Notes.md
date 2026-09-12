# Week 2 — Lecture 4: Networking Commands & SSH
### Accessing Remote Machines on the Command Line

> Exam Preparation Notes — Professional Edition
> Source: *Networking Commands and SSH* (Week 2, Lecture 4)

---

## Table of Contents

1. [Introduction to Networking](#1-introduction-to-networking)
2. [Public Network vs Private Network](#2-public-network-vs-private-network)
   - 2.1 [Concept](#21-concept)
   - 2.2 [Multiple Private Networks & Nested Gateways](#22-multiple-private-networks--nested-gateways)
3. [IPv4 Addressing](#3-ipv4-addressing)
   - 3.1 [IPv4 Address Range Classification](#31-ipv4-address-range-classification)
   - 3.2 [Private Network Address Table (Expanded)](#32-private-network-address-table-expanded)
4. [Ports and Connections](#4-ports-and-connections)
   - 4.1 [Concept of a Port](#41-concept-of-a-port)
   - 4.2 [Important Ports Table (Expanded)](#42-important-ports-table-expanded)
5. [Ways to Gain Remote Access](#5-ways-to-gain-remote-access)
6. [SSH — Secure Shell (Concept & Syntax)](#6-ssh--secure-shell-concept--syntax)
   - 6.1 [What is SSH?](#61-what-is-ssh)
   - 6.2 [Basic SSH Syntax](#62-basic-ssh-syntax)
   - 6.3 [SSH Key-Based Authentication](#63-ssh-key-based-authentication)
   - 6.4 [SCP — Secure Copy](#64-scp--secure-copy)
   - 6.5 [SSH Tunneling / Port Forwarding](#65-ssh-tunneling--port-forwarding)
7. [Firewall Concepts](#7-firewall-concepts)
8. [Protecting a Server](#8-protecting-a-server)
9. [SELinux — Security Enhanced Linux](#9-selinux--security-enhanced-linux)
   - 9.1 [Concept](#91-concept)
   - 9.2 [RBAC Components & Modes (Expanded)](#92-rbac-components--modes-expanded)
   - 9.3 [SELinux Syntax & Commands](#93-selinux-syntax--commands)
10. [Network Diagnostic Tools](#10-network-diagnostic-tools)
    - 10.1 [Tools Table (Expanded with Syntax & Examples)](#101-tools-table-expanded-with-syntax--examples)
11. [High Performance Computing (HPC)](#11-high-performance-computing-hpc)
12. [Summary](#12-summary)

---

## 1. Introduction to Networking

This lecture covers the fundamentals of **computer networking** required to access and manage **remote machines** using the **command line**, with a focus on **SSH (Secure Shell)** as the primary secure access mechanism.

**Key learning areas:**
- Public vs Private networks and gateways
- IPv4 addressing and address classes
- Ports and network connections
- Methods of remote access
- SSH concepts and command syntax
- Firewalls and server protection
- SELinux for access control
- Diagnostic networking tools
- Basics of High Performance Computing (HPC) access

---

## 2. Public Network vs Private Network

### 2.1 Concept

A network is broadly divided into:

| Term | Definition |
|---|---|
| **Public Network** | The open internet; machines here are reachable globally via public IP addresses. |
| **Private Network** | An internal/local network (e.g., LAN) where machines use private IP addresses not directly routable on the internet. |
| **Gateway** | A device (usually a router) that connects a private network to the public network (or to another network), enabling communication between them. |

**Diagram Concept (from slide):**
- Two **Private Networks** (Blue and Orange) are each connected to the **Public Network** (Yellow) through their own **gateway**.
- Every device inside a private network reaches the internet *only* through its gateway.

**Example:**
- Your home Wi-Fi router acts as the **gateway**. Your laptop and phone are on the **private network** (e.g., `192.168.1.x`), and the router connects you to the **public network** (the internet) using a single public IP address.

### 2.2 Multiple Private Networks & Nested Gateways

Private networks can be **nested** — a private network can itself contain smaller private networks, each connected via its own gateway.

**Example structure (from slide):**
- **Private Network #1** (outer network) contains:
  - **Private Network #2** — connected via **Gateway 1-2**
  - **Private Network #3** — connected via **Gateway 2-3** (which connects Network #2 and Network #3)

**Real-world example:**
A large organization's network:
```
Internet
   |
Organization Gateway (Private Network #1)
   |
   ├── Department A Gateway (Private Network #2)
   |         └── Employee machines
   |
   └── Department B Gateway (Private Network #3)
             └── Server machines
```
This layered gateway structure is common in enterprise networks and data centers, where each layer adds a level of routing and firewall control.

---

## 3. IPv4 Addressing

### 3.1 IPv4 Address Range Classification

IPv4 addresses are divided into three broad categories:

```
IPv4 Address Range
├── Localhost        → 127.0.0.0/8
├── Private Network   → Class A, B, C (see table below)
└── Public Network    → Globally routable addresses (🌐)
```

- **Localhost (`127.0.0.0/8`)**: Reserved for a machine to refer to itself (loopback). Example: `127.0.0.1` (commonly called "localhost").
- **Private Network**: Reserved ranges used only within internal networks (not routed on the public internet).
- **Public Network**: Globally unique addresses assigned by internet authorities (ICANN/IANA) and routable across the internet.

### 3.2 Private Network Address Table (Expanded)

Since the private network range contains **three distinct classes**, each is expanded below for clarity:

| Class | CIDR Notation | Address Range | Total Usable Addresses |
|---|---|---|---|
| **Class A** | `10.0.0.0/8` | `10.0.0.0 – 10.255.255.255` | 16,777,216 |
| **Class B** | `172.16.0.0/12` | `172.16.0.0 – 172.31.255.255` | 1,048,576 |
| **Class C** | `192.168.0.0/16` | `192.168.0.0 – 192.168.255.255` | 65,536 |

**Notes:**
- **Class A** ranges are typically used by **large organizations/ISPs** (huge address pool).
- **Class B** ranges are typically used by **medium-sized organizations**.
- **Class C** ranges (`192.168.x.x`) are the most common in **home routers and small office networks**.

**Example:**
- A home router assigning addresses like `192.168.1.2`, `192.168.1.3` to connected devices is using the **Class C** private range.
- A large corporate network might use `10.20.5.100` — a **Class A** private address.

---

## 4. Ports and Connections

### 4.1 Concept of a Port

A **port** is a logical endpoint of communication that, combined with an **IP address**, uniquely identifies a specific service/process on a machine for **routing** a **connection**.

**Concept flow (from slide):**
```
Machine A (IP + Port) ---[connection]---> Machine B (IP + Port)
             \                                  /
              \----------[routing]-------------/
```

- A single IP address can have **multiple open ports**, each corresponding to a different running service (e.g., web server, mail server, SSH daemon).
- A **connection** is established between a source (IP + port) and a destination (IP + port), and **routing** determines the network path the data takes.

**Example:**
- When you visit a website, your browser connects to the server's IP address on **port 443** (HTTPS). Meanwhile, the same server might also be listening on **port 22** (SSH) for administrators to log in remotely — both use the same IP but different ports.

### 4.2 Important Ports Table (Expanded)

The lecture lists several **important/well-known ports**. Each is expanded below with its protocol name, description, and a practical example:

| Port | Service | Full Name | Purpose | Example Usage |
|---|---|---|---|---|
| **21** | ftp | File Transfer Protocol | Transferring files between client and server | `ftp ftp.example.com` |
| **22** | ssh | Secure Shell | Secure remote login and command execution | `ssh user@192.168.1.10` |
| **25** | smtp | Simple Mail Transfer Protocol | Sending email between mail servers | Used internally by mail servers like Postfix |
| **80** | http | Hypertext Transfer Protocol | Unencrypted web traffic | `http://example.com` |
| **443** | https | Secure Hypertext Transfer Protocol | Encrypted (TLS/SSL) web traffic | `https://example.com` |
| **631** | cups | Common Unix Printing System | Network printing service | Used by Linux print servers |
| **3306** | mysql | MySQL Database | Database connections | `mysql -h 192.168.1.10 -P 3306 -u root -p` |

> **Exam Tip:** Remember port numbers with their protocols — a very common exam/interview question is to match port number → service name (e.g., "Which port does SSH use?" → **22**).

---

## 5. Ways to Gain Remote Access

The lecture lists several methods to remotely access a machine:

| Method | Description | Example / Tool |
|---|---|---|
| **VPN access** | Creates a secure, encrypted tunnel into a private network as if you were physically on it | Corporate VPN clients (OpenVPN, Cisco AnyConnect) |
| **SSH Tunneling** | Uses SSH to securely forward traffic through an encrypted channel | `ssh -L`, `ssh -R` (see [Section 6.5](#65-ssh-tunneling--port-forwarding)) |
| **Remote Desktop** | Provides a full graphical desktop session over the network | x2go, RDP (Remote Desktop Protocol), PCoIP |
| **Desktop over Browser** | Access a remote desktop directly through a web browser, no client install needed | Apache Guacamole |
| **Commercial (over internet)** | Third-party proprietary remote access services, often used for support | TeamViewer, AnyDesk, Zoho Assist |

**Explanation of each:**
- **VPN Access**: Best for accessing an *entire private network* securely (e.g., connecting to office resources from home).
- **SSH Tunneling**: Best for securely accessing a *specific service* on a remote machine, or forwarding traffic through an intermediate secure host.
- **Remote Desktop (x2go, RDP, PCoIP)**: Best when a **graphical interface** is required, not just command-line access.
- **Desktop over Browser (Apache Guacamole)**: Useful when you cannot install client software (e.g., accessing from a restricted/public computer).
- **Commercial tools (TeamViewer, AnyDesk, Zoho Assist)**: Common for quick, ad-hoc remote support sessions, especially for non-technical end users.

---

## 6. SSH — Secure Shell (Concept & Syntax)

### 6.1 What is SSH?

**SSH (Secure Shell)** is a cryptographic network protocol used to securely operate network services over an unsecured network — most commonly for **remote command-line login** and **secure file transfer**. It operates by default on **TCP port 22**.

**Why SSH is important (from lecture context):**
- It is the primary method for **accessing remote machines on the command line**.
- Required for accessing **High Performance Computing (HPC)** clusters (see [Section 11](#11-high-performance-computing-hpc)).
- Forms the basis of secure **tunneling** for remote access ([Section 5](#5-ways-to-gain-remote-access)).

### 6.2 Basic SSH Syntax

**General Syntax:**
```bash
ssh [options] username@hostname_or_IP
```

**Common Examples:**
```bash
# Connect to a remote server using username and IP address
ssh john@192.168.1.10

# Connect using a domain name
ssh john@remote-server.example.com

# Connect on a non-default port (e.g., port 2222)
ssh -p 2222 john@192.168.1.10

# Connect and immediately run a single remote command
ssh john@192.168.1.10 "ls -l /home/john"

# Enable verbose mode for debugging connection issues
ssh -v john@192.168.1.10
```

**Common Options:**

| Option | Meaning |
|---|---|
| `-p <port>` | Connect on a custom port instead of default 22 |
| `-v` | Verbose mode (useful for troubleshooting) |
| `-i <keyfile>` | Specify a private key file for authentication |
| `-X` | Enable X11 forwarding (to run GUI apps remotely) |
| `-L` | Local port forwarding (tunneling) |
| `-R` | Remote port forwarding (tunneling) |

### 6.3 SSH Key-Based Authentication

Instead of typing a password each time, SSH supports secure **public/private key authentication**.

**Syntax to generate a key pair:**
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

**Copy the public key to a remote server:**
```bash
ssh-copy-id username@remote-host
```

**Example workflow:**
```bash
# Step 1: Generate a key pair (creates id_rsa and id_rsa.pub)
ssh-keygen -t rsa -b 4096

# Step 2: Copy the public key to the server
ssh-copy-id john@192.168.1.10

# Step 3: Now login without a password
ssh john@192.168.1.10
```

### 6.4 SCP — Secure Copy

**SCP** uses the SSH protocol to securely transfer files between machines.

**Syntax:**
```bash
scp [options] source destination
```

**Examples:**
```bash
# Copy a local file to a remote server
scp report.pdf john@192.168.1.10:/home/john/documents/

# Copy a file FROM a remote server to your local machine
scp john@192.168.1.10:/home/john/data.csv ./local-folder/

# Copy an entire directory recursively
scp -r ./project_folder john@192.168.1.10:/home/john/
```

### 6.5 SSH Tunneling / Port Forwarding

SSH tunneling securely forwards network traffic through an encrypted SSH connection — directly relevant to the lecture's "Ways to Gain Remote Access" ([Section 5](#5-ways-to-gain-remote-access)).

**Local Port Forwarding Syntax:**
```bash
ssh -L local_port:target_host:target_port username@ssh_server
```
**Example:** Access a remote MySQL database (port 3306) securely through an SSH gateway:
```bash
ssh -L 3307:localhost:3306 john@192.168.1.10
# Now connect locally to: mysql -h 127.0.0.1 -P 3307
```

**Remote Port Forwarding Syntax:**
```bash
ssh -R remote_port:local_host:local_port username@ssh_server
```
**Example:** Expose a locally running web app (port 8080) to a remote server:
```bash
ssh -R 9000:localhost:8080 john@192.168.1.10
```

---

## 7. Firewall Concepts

A **firewall** controls what network traffic is allowed to pass, based on rules applied at multiple points.

**Key firewall considerations (from lecture):**
- **Ports open on my machine** — which local ports are listening/accepting connections.
- **Ports needed to be accessed on remote machine** — which ports must be reachable on the target server.
- **Network routing over the port** — the path traffic takes to reach the destination port.
- **Firewall controls at each hop** — every intermediate device (router, gateway) may apply its own firewall rules.

**Example:**
If you want to SSH into a remote server, **port 22 must be**:
1. Open on the remote machine's local firewall,
2. Allowed by any network firewall between you and the server,
3. Properly routed through all intermediate gateways/hops.

If any single hop blocks port 22, the connection will fail — this is why diagnosing connectivity issues often requires checking firewalls at multiple levels.

---

## 8. Protecting a Server

A public-facing server is typically protected using **layered security**, as shown in the lecture diagram:

```
Server (public service) ⇄ Web Application Filter ⇄ Network Firewall ⇄ 🌐 Anonymous Users
```

| Layer | Role |
|---|---|
| **Server with a public service** | The actual machine hosting the application (e.g., a web app) |
| **Web Application Filter (WAF)** | Filters malicious application-layer traffic (e.g., SQL injection, XSS attempts) before it reaches the server |
| **Network Firewall** | Filters traffic at the network/transport layer (blocks unwanted ports, IPs, protocols) |
| **Anonymous Users** | External/public internet users trying to reach the service |

**Explanation:**
This is a defense-in-depth approach: traffic from anonymous users first passes through a **network firewall** (blocking unauthorized ports/protocols), then through a **Web Application Filter** (blocking malicious application-level requests), before finally reaching the **server**.

---

## 9. SELinux — Security Enhanced Linux

### 9.1 Concept

**SELinux (Security-Enhanced Linux)** is a Linux kernel security module providing an **additional layer of access control**, beyond standard Unix permissions.

**Key points:**
- Available on **Ubuntu**, and standard on server-grade distributions like **CentOS, Fedora, RHEL, SuSE Linux**.
- Adds **access control on files and services**.
- Implements **Role-Based Access Control (RBAC)**.
- Provides **process sandboxing** and **least privilege access** for subjects (processes/users).
- SELinux is **recommended for all publicly visible servers**.

### 9.2 RBAC Components & Modes (Expanded)

Since SELinux includes **multiple RBAC components and multiple modes**, each is expanded into its own table below:

**A. RBAC Components:**

| Component | Example Value | Description |
|---|---|---|
| **User** | `unconfined_u` | The SELinux user identity mapped to the Linux user |
| **Role** | `object_r` | Defines what a user/process is allowed to do (used for RBAC) |
| **Type** | `user_home_t` | The SELinux "type" — the core mechanism controlling access (Type Enforcement) |
| **Level** | `s0` | The sensitivity/security level (used in Multi-Level Security setups) |

**B. SELinux Modes:**

| Mode | Description |
|---|---|
| **Disabled** | SELinux is completely turned off; no policy enforcement |
| **Enforcing** | SELinux actively enforces policy — denies and logs unauthorized actions |
| **Permissive** | SELinux logs violations but does **not** block/deny them (useful for testing policies) |

### 9.3 SELinux Syntax & Commands

**Checking SELinux context of files:**
```bash
ls -lZ
```
**Example output interpretation:**
```
-rw-r--r--. john john unconfined_u:object_r:user_home_t:s0 report.txt
```

**Checking SELinux context of processes:**
```bash
ps -eZ
```

**Key management tools:**

| Command | Purpose | Example |
|---|---|---|
| `semanage` | Manage SELinux policy (ports, file contexts, booleans, etc.) | `semanage port -a -t http_port_t -p tcp 8080` |
| `restorecon` | Restore default SELinux security context of files | `restorecon -v /var/www/html/index.html` |

**Example workflow:**
```bash
# Check current SELinux mode
getenforce

# Temporarily set SELinux to permissive mode
setenforce 0

# View file security context
ls -lZ /var/www/html/

# Restore correct SELinux context after moving a file
restorecon -Rv /var/www/html/
```

---

## 10. Network Diagnostic Tools

### 10.1 Tools Table (Expanded with Syntax & Examples)

The lecture lists multiple diagnostic tools. Since this list contains many distinct tools, each is expanded below with **syntax and example usage** for clarity.

| Tool | Purpose |
|---|---|
| `ping` | Check if a remote machine is reachable/up |
| `traceroute` | Diagnose hop-by-hop timing to a remote machine |
| `nslookup` | Convert an IP address to a domain name (or vice versa) |
| `dig` | DNS lookup utility (more detailed than nslookup) |
| `netstat` | Print current network connections |
| `mxtoolbox.com` | Online tool to check accessibility from the public network |
| `whois` lookup | Find out who owns a particular domain name |
| `nmap` ⚠️ | Network port scanner (use carefully — only on authorized systems) |
| `wireshark` ⚠️ | Network protocol analyzer (use carefully — captures live traffic) |

**Detailed syntax and examples for each tool:**

**1. `ping`** — Check if a machine is up
```bash
ping google.com
ping -c 4 192.168.1.10     # Send only 4 packets
```

**2. `traceroute`** — Diagnose the network path/hop timings
```bash
traceroute google.com
traceroute -n 8.8.8.8       # Do not resolve hostnames (faster)
```

**3. `nslookup`** — Resolve IP address to name (or name to IP)
```bash
nslookup google.com
nslookup 8.8.8.8
```

**4. `dig`** — Detailed DNS lookup
```bash
dig example.com
dig example.com MX          # Look up mail server records
```

**5. `netstat`** — Print active network connections
```bash
netstat -tulpn               # Show listening TCP/UDP ports with process names
netstat -a                   # Show all connections
```

**6. `mxtoolbox.com`**
Web-based tool — no command line syntax; used by visiting the website and entering a domain/IP to check DNS, blacklist status, and public accessibility.

**7. `whois`** — Find domain ownership information
```bash
whois example.com
```

**8. `nmap`** ⚠️ *(Use only on systems you are authorized to scan)*
```bash
nmap 192.168.1.10                  # Basic scan
nmap -p 1-1000 192.168.1.10        # Scan a specific port range
nmap -sV 192.168.1.10              # Detect service versions
```

**9. `wireshark`** ⚠️ *(Use only with proper authorization)*
A GUI-based network protocol analyzer used to capture and inspect live network packets. Typically launched simply as:
```bash
wireshark
```
Then a network interface is selected within the GUI to begin capturing traffic.

> **⚠️ Important Exam Note:** Both `nmap` and `wireshark` are flagged with "careful" in the original lecture — they should only be used on networks/systems you **own or are explicitly authorized to test**, as unauthorized scanning or packet sniffing can be illegal.

---

## 11. High Performance Computing (HPC)

**Key points from the lecture:**
- Refer to **[www.top500.org](http://www.top500.org)** for statistics on the world's most powerful supercomputers.
- Accessing a remote **HPC machine is usually done over SSH**.
- **Long-duration jobs** are not run interactively — they are **submitted to a job scheduler** for execution (e.g., SLURM, PBS).
- **Large raw datasets** should be **processed remotely** (on the HPC cluster) before being transferred to your local machine, due to **network bandwidth and data transfer costs/limits**.
- **Comfort with the command line is essential** for working with HPC systems, since most clusters do not provide a graphical interface.

**Example conceptual workflow for HPC access:**
```bash
# Step 1: SSH into the HPC login node
ssh username@hpc-cluster.university.edu

# Step 2: Submit a long-running job to the scheduler (example: SLURM)
sbatch my_job_script.sh

# Step 3: Check job status
squeue -u username

# Step 4: Once processing is done, transfer only the final results
scp username@hpc-cluster.university.edu:/results/output.csv ./
```

---

## 12. Summary

| Topic | Key Takeaway |
|---|---|
| **Public vs Private Networks** | Private networks connect to the public internet via a **gateway**; networks can be nested with multiple gateways. |
| **IPv4 Addressing** | Addresses fall into **Localhost** (`127.0.0.0/8`), **Private** (Class A/B/C), or **Public** ranges. |
| **Ports** | A **port + IP address** uniquely identifies a service; common ports include **22 (SSH), 80 (HTTP), 443 (HTTPS)**. |
| **Remote Access Methods** | VPN, SSH tunneling, Remote Desktop (x2go/RDP/PCoIP), browser-based (Guacamole), and commercial tools (TeamViewer, AnyDesk). |
| **SSH** | The primary secure protocol (port 22) for remote command-line access, key-based authentication, file transfer (`scp`), and tunneling. |
| **Firewalls** | Must be checked at **every hop** — local machine, remote machine, and all intermediate network devices. |
| **Protecting a Server** | Layered defense: **Network Firewall → Web Application Filter → Server**. |
| **SELinux** | Adds RBAC-based access control (**user, role, type, level**) with three modes: **disabled, permissive, enforcing** — recommended for public servers. |
| **Diagnostic Tools** | `ping`, `traceroute`, `nslookup`, `dig`, `netstat`, `whois`, `nmap`, `wireshark` — each serves a distinct diagnostic purpose; `nmap`/`wireshark` require caution and authorization. |
| **HPC Access** | Almost always via **SSH**; long jobs go through a **job scheduler**; process large data remotely before transferring locally. |

### Final Exam-Focused Checklist
- [ ] Know the difference between **public** and **private** networks and the role of a **gateway**.
- [ ] Memorize IPv4 private ranges: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`.
- [ ] Memorize key ports: **21, 22, 25, 80, 443, 631, 3306**.
- [ ] Understand all **5 methods** of remote access and when to use each.
- [ ] Be able to write **basic SSH, SCP, and SSH tunneling commands** from memory.
- [ ] Understand **firewall checks at each hop**.
- [ ] Understand the **layered server protection model** (Firewall → WAF → Server).
- [ ] Know **SELinux RBAC components** (user, role, type, level) and **three modes**.
- [ ] Be able to name each **diagnostic tool** and its exact purpose.
- [ ] Understand why **HPC access uses SSH + job schedulers**, and why data should be processed remotely first.

---
*End of Notes — Week 2, Lecture 4: Networking & SSH*
