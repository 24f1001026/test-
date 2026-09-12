# Version Control, Git & GitHub — Complete Notes

## Table of Contents
1. [Introduction to Version Control](#1-introduction-to-version-control)
2. [Overview of Version Control Systems (VCS)](#2-overview-of-version-control-systems-vcs)
3. [Importance of Preventing Hardware Failures in Storage Systems](#3-importance-of-preventing-hardware-failures-in-storage-systems)
4. [RAID (Redundant Array of Independent Disks) — Overview](#4-raid-redundant-array-of-independent-disks--overview)
5. [The Git Protocol and Remote Synchronization](#5-the-git-protocol-and-remote-synchronization)
6. [Security Measures: Account Verification & Two-Factor Authentication (2FA)](#6-security-measures-account-verification--two-factor-authentication-2fa)
7. [Getting Started with Git](#7-getting-started-with-git)
8. [Personal Access Tokens (PAT), Repositories & Authentication](#8-personal-access-tokens-pat-repositories--authentication)
9. [Setting Up a Remote Server and Git Accounts](#9-setting-up-a-remote-server-and-git-accounts)
10. [Setting Up a GitHub Account & Two-Factor Authentication](#10-setting-up-a-github-account--two-factor-authentication)
11. [Branches — Managing and Merging Changes](#11-branches--managing-and-merging-changes)
12. [Setting Up a New Repository — Naming Conventions](#12-setting-up-a-new-repository--naming-conventions)
13. [Cloning Repositories](#13-cloning-repositories)
14. [Git Configuration for a New Repository](#14-git-configuration-for-a-new-repository)
15. [Troubleshooting Network Issues & Preparing to Push](#15-troubleshooting-network-issues--preparing-to-push)
16. [Core Git Workflow Commands (Summary)](#16-core-git-workflow-commands-summary)
17. [Branching and Merging in Version Control Systems (Recap)](#17-branching-and-merging-in-version-control-systems-recap)
18. [Quick Reference: Key Terms](#18-quick-reference-key-terms)
19. [Revision Summary](#19-revision-summary)

## 1. Introduction to Version Control

**Version Control** is the practice of tracking and managing changes to files (especially source code) over time. It allows multiple people to collaborate on a project without overwriting each other's work, and it keeps a complete history of every change made.

Key benefits of version control:
- Tracks who changed what, when, and why
- Allows reverting to previous versions if something breaks
- Enables multiple developers to work on the same project simultaneously
- Provides a safety net against accidental data loss
- Supports parallel development through branching

## 2. Overview of Version Control Systems (VCS)

There are two broad categories of VCS:

### a) Centralized Version Control Systems (CVCS)
- A single central server stores all versioned files.
- Clients "check out" files from this central place.
- Example: SVN (Subversion), CVS.
- Drawback: single point of failure — if the central server goes down, no one can collaborate or save version history.

### b) Distributed Version Control Systems (DVCS)
- Every client has a **full copy (mirror)** of the entire repository, including its complete history.
- Example: **Git**, Mercurial.
- Advantage: no single point of failure; anyone can restore the entire repository from their local copy.

## 3. Importance of Preventing Hardware Failures in Storage Systems

Hardware (disks) can fail unexpectedly, causing data loss. Preventing such failures is critical for:
- Business continuity
- Protecting valuable source code / data
- Avoiding downtime and costly recovery efforts

This is why storage redundancy technologies like **RAID** are used alongside version control practices — VCS protects code history, while RAID protects the physical storage layer.

## 4. RAID (Redundant Array of Independent Disks) — Overview

RAID is a data storage technology that combines multiple physical disk drives into a single logical unit to improve **speed**, **reliability**, and **fault tolerance** through techniques like **mirroring** and **striping**.

- **Mirroring**: Data is duplicated (copied) across two or more disks. If one disk fails, the data still exists safely on the other disk(s).
- **Striping**: Data is split into blocks and spread across multiple disks so read/write operations happen in parallel, improving speed.

### Common RAID Configurations
- **RAID 0 (Striping)**: Splits data across multiple disks for higher speed, but offers **no redundancy** — if one disk fails, all data is lost.
- **RAID 1 (Mirroring)**: Duplicates the same data across two disks. If one disk fails, the other still has a complete copy — improves safety but not capacity.
- **RAID 5 / distributed parity setups**: Data (and parity/check information) is distributed across multiple disks so that if one disk fails, the missing data can be reconstructed from the parity information on the remaining disks — balancing speed, storage efficiency, and fault tolerance.

The general goal of RAID configurations is: **improve performance (speed)** and/or **improve fault tolerance (safety)**, often by distributing or duplicating data across multiple physical disks.

## 5. The Git Protocol and Remote Synchronization

Git uses a **distributed model**, meaning:
- Every developer has a full local copy of the repository (including history).
- Changes are synced between the **local repository** and a **remote repository** (e.g., hosted on GitHub) using commands like `git push`, `git pull`, and `git fetch`.
- Git can communicate with remotes over several protocols:
  - **HTTPS** — most common, uses a username + password or Personal Access Token (PAT) for authentication.
  - **SSH** — uses SSH keys for secure, passwordless authentication.
  - **Git protocol** — a lightweight, fast protocol (mostly used for public, read-only access).

## 6. Security Measures: Account Verification & Two-Factor Authentication (2FA)

To protect accounts from identity theft and unauthorized access, platforms like GitHub require additional security layers:

- **Phone number verification**: Used to confirm the account holder's identity and add a recovery/verification method.
- **Two-Factor Authentication (2FA)**: Adds a second layer of security beyond just a password — typically a time-based code from an authenticator app or SMS. Even if a password is compromised, 2FA helps prevent unauthorized login.

Setting up 2FA is a recommended best practice when creating and securing a GitHub account.

## 7. Getting Started with Git

- Git can be used via the **command line (CLI)** or through **GUI tools** that let you visually navigate repositories, branches, and history.
- Basic setup typically involves installing Git and configuring your identity:
  ```bash
  git config --global user.name "Your Name"
  git config --global user.email "your.email@example.com"
  ```
- Screen-sharing/guided walkthroughs are often used in tutorials to demonstrate navigating Git's command options step-by-step.

## 8. Personal Access Tokens (PAT), Repositories & Authentication

### What is a Personal Access Token (PAT)?
A PAT is a secure, generated string of characters that acts as an alternative to a password when authenticating with GitHub over HTTPS (especially since GitHub deprecated plain password authentication for Git operations).

- PATs can be scoped with specific permissions (e.g., repo access, workflow access).
- They are used when:
  - Pushing/pulling code over HTTPS
  - Cloning private repositories
  - Authenticating with third-party tools or scripts

### Creating a Repository
- A **repository (repo)** is a storage space for a project, containing all files, folders, and the complete version history.
- Repositories can be created on GitHub (remote) or initialized locally with:
  ```bash
  git init
  ```

## 9. Setting Up a Remote Server and Git Accounts

- A **remote** is a version of your repository hosted elsewhere (e.g., on GitHub, GitLab, Bitbucket).
- To connect a local repository to a remote:
  ```bash
  git remote add origin <repository-url>
  ```
- Working with Git accounts involves:
  - Creating an account on the hosting platform (e.g., GitHub)
  - Authenticating via PAT or SSH keys
  - Linking local repositories to remote ones for synchronization

## 10. Setting Up a GitHub Account & Two-Factor Authentication

Steps typically covered:
1. Create a GitHub account (username, email, password).
2. Verify identity (email/phone verification).
3. Enable Two-Factor Authentication (2FA) for added security.
4. Generate a Personal Access Token if needed for command-line authentication.

## 11. Branches — Managing and Merging Changes

### What is a Branch?
A **branch** is an independent line of development. It allows you to work on new features, fixes, or experiments without affecting the main codebase (often called `main` or `master`).

Common branch commands:
```bash
git branch <branch-name>        # create a new branch
git checkout <branch-name>      # switch to a branch
git checkout -b <branch-name>   # create and switch in one step
git branch                      # list branches
```

### Merging
**Merging** combines changes from one branch into another (commonly merging a feature branch back into `main`).
```bash
git checkout main
git merge <branch-name>
```

- If both branches changed the same lines of code, a **merge conflict** occurs, which must be resolved manually by editing the conflicting files and then committing the resolution.

## 12. Setting Up a New Repository — Naming Conventions

Best practices when naming a repository:
- Avoid special characters (spaces, `#`, `%`, `&`, etc.)
- Use hyphens (`-`) or underscores (`_`) instead of spaces
- Keep names short, descriptive, and lowercase for consistency
- Avoid starting names with numbers or symbols

## 13. Cloning Repositories

**Cloning** creates a local copy of an existing remote repository:
```bash
git clone <repository-url>
```

- **Cloning a private repository** requires authentication — you'll be prompted for a username and password (or PAT), since private repos aren't publicly accessible.
- Public repositories can typically be cloned without authentication.

## 14. Git Configuration for a New Repository

After cloning or initializing a repository, it's common to configure:
```bash
git config user.name "Your Name"
git config user.email "your.email@example.com"
git config --list          # view current configuration
```

This ensures commits are properly attributed to the correct author.

## 15. Troubleshooting Network Issues & Preparing to Push

Common troubleshooting steps before pushing changes:
```bash
git status          # check current state of the working directory
git remote -v        # verify remote connections
git fetch            # check for updates without merging
```

- Network issues (e.g., failed authentication, connectivity problems, DNS errors) can prevent pushing/pulling. Verifying remote URLs, tokens, and internet connectivity are common first troubleshooting steps.

## 16. Core Git Workflow Commands (Summary)

```bash
git init                     # initialize a new local repository
git clone <url>               # copy a remote repository locally
git status                    # see current changes/status
git add <file>                 # stage changes
git commit -m "message"       # save staged changes with a message
git push origin <branch>      # upload local commits to remote
git pull origin <branch>      # download and merge remote changes
git fetch                     # download remote changes without merging
git branch                    # list/manage branches
git checkout <branch>         # switch branches
git merge <branch>             # merge a branch into current branch
```

## 17. Branching and Merging in Version Control Systems (Recap)

- **Branching** allows isolated development — experiment safely without breaking the main project.
- **Merging** brings changes back together once work is tested and ready.
- Using the command line, merging is done by switching to the target branch and running `git merge`, followed by resolving any conflicts and committing the final merged result.
- A healthy Git workflow often includes:
  1. Creating a feature branch
  2. Making and committing changes
  3. Pushing the branch to the remote
  4. Opening a Pull Request (PR) on GitHub
  5. Reviewing and merging the PR into `main`

## 18. Quick Reference: Key Terms

| Term | Meaning |
|---|---|
| Repository (repo) | Storage location for a project's files and history |
| Commit | A saved snapshot of changes |
| Branch | An independent line of development |
| Merge | Combining changes from one branch into another |
| Clone | Creating a local copy of a remote repository |
| Remote | A version of the repository hosted on another server (e.g., GitHub) |
| Push | Sending local commits to a remote repository |
| Pull | Fetching and merging changes from a remote repository |
| PAT (Personal Access Token) | A secure token used instead of a password for authentication |
| 2FA | Two-Factor Authentication — an extra security layer for account login |
| Merge Conflict | Occurs when changes in two branches overlap and must be manually resolved |

## 19. Revision Summary

A condensed recap for quick revision before an exam, interview, or practical session:

- **Version Control** = tracking changes to files over time; **Centralized** (single server, e.g. SVN) vs **Distributed** (every user has full history, e.g. Git).
- **RAID** protects physical storage: **striping** (speed, RAID 0, no redundancy), **mirroring** (duplication, RAID 1, safe but no extra capacity), **parity-based** (RAID 5, balances speed + fault tolerance).
- **Git** is distributed — local repo ↔ remote repo, synced via `push`/`pull`/`fetch`, over **HTTPS** (PAT) or **SSH** (keys).
- **Security**: GitHub uses phone verification + **2FA** to protect accounts; **PAT** replaces password for HTTPS-based Git authentication.
- **Repository basics**: `git init` (new local repo), `git clone <url>` (copy remote repo locally — private repos need authentication), `git remote add origin <url>` (link local to remote).
- **Naming rule**: no special characters in repo names; use hyphens/underscores, lowercase, descriptive.
- **Config**: always set `user.name` and `user.email` before committing.
- **Everyday commands**: `status`, `add`, `commit -m`, `push`, `pull`, `fetch` — check `status` and `remote -v` first when troubleshooting network/push issues.
- **Branching & Merging**: `branch` creates an isolated line of work; `checkout` switches; `merge` combines changes back — conflicts must be resolved manually. Typical flow: feature branch → commit → push → Pull Request → review → merge into `main`.

**One-line takeaway:** *Git/GitHub give you safe, distributed version history with branching and merging, while RAID and account security (PAT, 2FA) protect the underlying storage and access layer.*

---

*These notes consolidate concepts on version control fundamentals, RAID storage technology, Git protocol/workflow, and GitHub account setup, security, and collaboration practices.*
