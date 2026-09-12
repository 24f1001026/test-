# Git & GitHub — Complete Beginner's Guide (Zero to Working)

This guide assumes you know **nothing** about version control. It starts from the absolute basics and walks through everything you need to use Git and GitHub confidently.

## Table of Contents
1. [What is Version Control?](#1-what-is-version-control)
2. [Why Do We Need Version Control?](#2-why-do-we-need-version-control)
3. [Types of Version Control Systems](#3-types-of-version-control-systems)
4. [What is Git?](#4-what-is-git)
5. [What is GitHub?](#5-what-is-github)
6. [Git vs GitHub — The Key Difference](#6-git-vs-github--the-key-difference)
7. [Installing Git](#7-installing-git)
8. [Configuring Git for the First Time](#8-configuring-git-for-the-first-time)
9. [Core Git Concepts You Must Understand](#9-core-git-concepts-you-must-understand)
10. [Creating Your First Local Repository](#10-creating-your-first-local-repository)
11. [The Git Workflow: Working Directory → Staging → Repository](#11-the-git-workflow-working-directory--staging--repository)
12. [Basic Git Commands (Step by Step)](#12-basic-git-commands-step-by-step)
13. [Checking History and Status](#13-checking-history-and-status)
14. [Undoing Changes](#14-undoing-changes)
15. [Branching Explained Simply](#15-branching-explained-simply)
16. [Merging Branches](#16-merging-branches)
17. [Merge Conflicts and How to Resolve Them](#17-merge-conflicts-and-how-to-resolve-them)
18. [Creating a GitHub Account](#18-creating-a-github-account)
19. [Securing Your GitHub Account (2FA)](#19-securing-your-github-account-2fa)
20. [Creating a Repository on GitHub](#20-creating-a-repository-on-github)
21. [Repository Naming Rules](#21-repository-naming-rules)
22. [Connecting a Local Repository to GitHub](#22-connecting-a-local-repository-to-github)
23. [Authentication: Personal Access Tokens (PAT)](#23-authentication-personal-access-tokens-pat)
24. [Authentication: SSH Keys (Alternative Method)](#24-authentication-ssh-keys-alternative-method)
25. [Pushing Code to GitHub](#25-pushing-code-to-github)
26. [Cloning a Repository](#26-cloning-a-repository)
27. [Cloning a Private Repository](#27-cloning-a-private-repository)
28. [Pulling and Fetching Updates](#28-pulling-and-fetching-updates)
29. [Working with Remote Branches](#29-working-with-remote-branches)
30. [Pull Requests (PRs) — Collaboration on GitHub](#30-pull-requests-prs--collaboration-on-github)
31. [Forking a Repository](#31-forking-a-repository)
32. [.gitignore File](#32-gitignore-file)
33. [Common Errors and Troubleshooting](#33-common-errors-and-troubleshooting)
34. [Complete Command Cheat Sheet](#34-complete-command-cheat-sheet)
35. [A Typical Real-World Workflow (Putting It All Together)](#35-a-typical-real-world-workflow-putting-it-all-together)
36. [Glossary of Key Terms](#36-glossary-of-key-terms)
37. [Additional Resources & Further Reading](#37-additional-resources--further-reading)
38. [Revision Summary](#38-revision-summary)

---

## 1. What is Version Control?

Version control is a system that records changes to a file or set of files over time, so you can:
- See exactly what changed, when, and who changed it
- Go back ("revert") to an older version if something breaks
- Work on new ideas without risking the working version of your project

Think of it like an infinite "undo" button combined with a detailed diary of every change ever made to your project.


**📚 Learn more:**
- [Pro Git Book – Ch.1 About Version Control](https://git-scm.com/book/en/v2/Getting-Started-About-Version-Control) (free, official)

## 2. Why Do We Need Version Control?

Without version control, teams often end up with messy folders like:
```
project_final.zip
project_final_v2.zip
project_final_v2_ACTUAL_FINAL.zip
project_final_v2_ACTUAL_FINAL_fixed.zip
```

Version control solves this by:
- Keeping **one** organized history instead of multiple duplicate files
- Allowing **many people** to work on the same project at once without overwriting each other
- Letting you safely experiment (try new features) without breaking the main project
- Creating a permanent, searchable record of every change and why it was made


**📚 Learn more:**
- [Atlassian – Why Version Control Matters](https://www.atlassian.com/git/tutorials/what-is-version-control)

## 3. Types of Version Control Systems

### Centralized Version Control (CVCS)
- One central server holds the full history.
- Everyone connects to this single server to get updates or save changes.
- Example: SVN, CVS.
- Risk: if the central server fails, nobody can access history until it's restored.

### Distributed Version Control (DVCS)
- Every user has a **complete copy** of the project and its entire history on their own computer.
- Example: **Git**, Mercurial.
- Advantage: no single point of failure, and you can work fully offline.

Git is a **Distributed Version Control System**.


**📚 Learn more:**
- [Atlassian – Centralized vs Distributed VCS](https://www.atlassian.com/git/tutorials/what-is-version-control#distributed-vs-centralized-version-control-systems)

## 4. What is Git?

Git is free, open-source software that runs on your computer and tracks changes to your files. It is:
- **Local-first**: works entirely on your machine without needing internet access
- **Fast**: designed to handle everything from small to huge projects efficiently
- **Distributed**: every copy of a repository has the full project history

Git itself is just a tool — it doesn't require GitHub to work. You can use Git entirely on your own computer.


**📚 Learn more:**
- [Official Git Documentation](https://git-scm.com/doc)
- [Pro Git Book (free, full text)](https://git-scm.com/book/en/v2)

## 5. What is GitHub?

GitHub is a **website/cloud service** that hosts Git repositories online. It adds:
- A remote (online) place to store and back up your Git repositories
- Collaboration tools: Pull Requests, Issues, code review, project boards
- Social/discovery features: stars, forks, public profiles
- Automation tools: GitHub Actions (CI/CD)

Other similar platforms exist too, such as GitLab and Bitbucket — GitHub is simply the most popular one.


**📚 Learn more:**
- [GitHub Docs – About GitHub](https://docs.github.com/en/get-started/start-your-journey/about-github-and-git)

## 6. Git vs GitHub — The Key Difference

| | Git | GitHub |
|---|---|---|
| What it is | A version control **tool/software** | A **website** that hosts Git repositories |
| Where it runs | On your local computer | On the cloud (internet) |
| Needs internet? | No | Yes (to access/share online repos) |
| Purpose | Track changes to files | Store, share, and collaborate on Git repositories |

**In short: Git is the engine, GitHub is a garage where you can park and share your car.**


**📚 Learn more:**
- [GitHub Docs – Git and GitHub Learning Resources](https://docs.github.com/en/get-started/start-your-journey/git-and-github-learning-resources)

## 7. Installing Git

- **Windows**: Download from [git-scm.com](https://git-scm.com) and run the installer.
- **Mac**: Install via Homebrew: `brew install git`, or install Xcode Command Line Tools.
- **Linux (Debian/Ubuntu)**: `sudo apt update && sudo apt install git`

Verify installation:
```bash
git --version
```


**📚 Learn more:**
- [Official Git Downloads (all OS)](https://git-scm.com/downloads)
- [GitHub Docs – Set Up Git](https://docs.github.com/en/get-started/getting-started-with-git/set-up-git)

## 8. Configuring Git for the First Time

Before using Git, tell it who you are — this information gets attached to every change you make:
```bash
git config --global user.name "Your Name"
git config --global user.email "youremail@example.com"
```

Check your settings anytime:
```bash
git config --list
```


**📚 Learn more:**
- [GitHub Docs – Setting your username & email in Git](https://docs.github.com/en/get-started/getting-started-with-git/setting-your-username-in-git)
- [git-scm.com – git config reference](https://git-scm.com/docs/git-config)

## 9. Core Git Concepts You Must Understand

| Term | Meaning |
|---|---|
| **Repository (repo)** | A folder tracked by Git; contains your files plus their full history |
| **Commit** | A saved "snapshot" of your project at a point in time |
| **Branch** | An independent line of development |
| **Remote** | An online version of your repo (e.g., on GitHub) |
| **Clone** | Downloading a full copy of a remote repository |
| **Push** | Uploading your local commits to a remote repository |
| **Pull** | Downloading and merging changes from a remote repository |
| **Merge** | Combining changes from one branch into another |
| **Staging Area** | A "waiting area" where you prepare changes before committing them |


**📚 Learn more:**
- [Pro Git Book – Ch.2 Git Basics](https://git-scm.com/book/en/v2/Git-Basics-Getting-a-Git-Repository)
- [GitHub Docs – Glossary](https://docs.github.com/en/get-started/learning-about-github/github-glossary)

## 10. Creating Your First Local Repository

Navigate to your project folder and run:
```bash
cd my-project
git init
```

This creates a hidden `.git` folder — this is what turns a normal folder into a Git repository.


**📚 Learn more:**
- [Pro Git Book – Getting a Git Repository](https://git-scm.com/book/en/v2/Git-Basics-Getting-a-Git-Repository)
- [git-scm.com – git init reference](https://git-scm.com/docs/git-init)

## 11. The Git Workflow: Working Directory → Staging → Repository

Git has **three main areas**:

1. **Working Directory** — the actual files you're editing right now.
2. **Staging Area (Index)** — where you place changes you want to include in your next commit.
3. **Repository (.git folder)** — where committed snapshots are permanently stored.

Flow:
```
Edit file → git add (stage it) → git commit (save it permanently)
```


**📚 Learn more:**
- [Pro Git Book – Recording Changes to the Repository](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository)

## 12. Basic Git Commands (Step by Step)

```bash
git init                      # Start tracking a folder with Git
git add <filename>            # Stage a specific file
git add .                     # Stage ALL changed files
git commit -m "message"       # Save staged changes with a description
git log                       # View commit history
```

Example full sequence:
```bash
echo "Hello World" > hello.txt
git add hello.txt
git commit -m "Add hello.txt with greeting"
```


**📚 Learn more:**
- [git-scm.com – git add reference](https://git-scm.com/docs/git-add)
- [git-scm.com – git commit reference](https://git-scm.com/docs/git-commit)

## 13. Checking History and Status

```bash
git status         # Shows what's changed, staged, or untracked
git log            # Full commit history
git log --oneline  # Condensed, one line per commit
git diff           # Shows exact line-by-line changes not yet staged
```


**📚 Learn more:**
- [git-scm.com – git log reference](https://git-scm.com/docs/git-log)
- [git-scm.com – git status reference](https://git-scm.com/docs/git-status)

## 14. Undoing Changes

```bash
git restore <file>              # Discard unstaged changes in a file
git restore --staged <file>     # Unstage a file (keep the edits)
git reset --soft HEAD~1         # Undo last commit, keep changes staged
git reset --hard HEAD~1         # Undo last commit AND discard changes (careful!)
git revert <commit-hash>        # Create a new commit that undoes a specific past commit (safe for shared history)
```


**📚 Learn more:**
- [GitHub Docs – Undoing changes](https://docs.github.com/en/get-started/using-git/undoing-changes)
- [git-scm.com – git reset reference](https://git-scm.com/docs/git-reset)

## 15. Branching Explained Simply

Imagine your project as a tree. The `main` branch is the trunk — your stable, working version. A **branch** is like growing a new stem off the trunk to try something new, without affecting the trunk itself.

```bash
git branch                     # List all branches
git branch <branch-name>       # Create a new branch
git checkout <branch-name>     # Switch to that branch
git checkout -b <branch-name>  # Create AND switch in one step
git switch <branch-name>       # Modern alternative to checkout
```

Why branch?
- Build a new feature safely
- Fix a bug without touching the stable version
- Experiment freely — delete the branch if it doesn't work out


**📚 Learn more:**
- [Pro Git Book – Git Branching](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell)
- [Learn Git Branching (interactive, visual)](https://learngitbranching.js.org/)

## 16. Merging Branches

Once your work on a branch is ready, bring it back into `main`:
```bash
git checkout main
git merge <branch-name>
```

This combines the changes from your branch into `main`.


**📚 Learn more:**
- [Pro Git Book – Basic Branching and Merging](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)

## 17. Merge Conflicts and How to Resolve Them

A **conflict** happens when two branches changed the **same lines** of the **same file** differently. Git can't decide which version is correct, so it asks you to choose.

When it happens, Git marks the file like this:
```
<<<<<<< HEAD
Your current branch's version
=======
The other branch's version
>>>>>>> branch-name
```

**To resolve:**
1. Open the file and manually edit it to keep the correct content.
2. Delete the `<<<<<<<`, `=======`, and `>>>>>>>` markers.
3. Save the file, then run:
```bash
git add <file>
git commit -m "Resolve merge conflict"
```


**📚 Learn more:**
- [GitHub Docs – Resolving a merge conflict on GitHub](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/resolving-a-merge-conflict-on-github)
- [GitHub Docs – Resolving a merge conflict using the command line](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/resolving-a-merge-conflict-using-the-command-line)

## 18. Creating a GitHub Account

1. Go to [github.com](https://github.com)
2. Click **Sign Up**
3. Enter a username, email, and password
4. Verify your email address
5. Optionally verify your phone number for extra account security


**📚 Learn more:**
- [GitHub Docs – Signing up for a new GitHub account](https://docs.github.com/en/get-started/start-your-journey/creating-an-account-on-github)

## 19. Securing Your GitHub Account (2FA)

**Two-Factor Authentication (2FA)** adds an extra layer of protection beyond your password.

To enable it:
1. Go to **Settings → Password and authentication**
2. Click **Enable two-factor authentication**
3. Choose a method: authenticator app (recommended) or SMS
4. Scan the QR code with an app like Google Authenticator or Authy
5. Save your recovery codes somewhere safe

Once enabled, you'll need both your password **and** a time-based code to log in — even if your password is stolen, your account stays protected.


**📚 Learn more:**
- [GitHub Docs – Configuring two-factor authentication](https://docs.github.com/en/authentication/securing-your-account-with-two-factor-authentication-2fa/configuring-two-factor-authentication)

## 20. Creating a Repository on GitHub

1. Click the **+** icon (top right) → **New repository**
2. Enter a repository name
3. Choose **Public** (anyone can see it) or **Private** (only you/invited people can see it)
4. Optionally add a README, `.gitignore`, and license
5. Click **Create repository**


**📚 Learn more:**
- [GitHub Docs – Creating a new repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository)

## 21. Repository Naming Rules

- Avoid spaces and special characters (`#`, `%`, `&`, `@`, etc.)
- Use hyphens (`-`) or underscores (`_`) instead of spaces
- Keep it short, lowercase, and descriptive
  - Good: `weather-app`, `student_management_system`
  - Avoid: `My Project!! (final)`


**📚 Learn more:**
- [GitHub Docs – About repositories](https://docs.github.com/en/repositories/creating-and-managing-repositories/about-repositories)

## 22. Connecting a Local Repository to GitHub

If you already have a local Git repo and want to link it to a new GitHub repo:
```bash
git remote add origin https://github.com/username/repo-name.git
git remote -v          # Verify the remote was added correctly
```


**📚 Learn more:**
- [GitHub Docs – Adding a remote repository](https://docs.github.com/en/get-started/getting-started-with-git/managing-remote-repositories)

## 23. Authentication: Personal Access Tokens (PAT)

GitHub no longer accepts your account password for Git operations over HTTPS — you need a **Personal Access Token (PAT)** instead.

**To create one:**
1. GitHub → **Settings → Developer settings → Personal access tokens**
2. Click **Generate new token**
3. Select the scopes/permissions needed (e.g., `repo` for full repository access)
4. Set an expiration date
5. Copy the token immediately — GitHub will **never show it again**

When Git asks for a password during `push`/`pull`/`clone` over HTTPS, paste your PAT instead of your account password.


**📚 Learn more:**
- [GitHub Docs – Managing your personal access tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)

## 24. Authentication: SSH Keys (Alternative Method)

SSH keys let you authenticate without entering a token every time.

```bash
ssh-keygen -t ed25519 -C "youremail@example.com"   # Generate a key pair
cat ~/.ssh/id_ed25519.pub                           # Copy your public key
```
Then paste the public key into **GitHub → Settings → SSH and GPG keys → New SSH key**.

Once set up, use the SSH URL instead of HTTPS when cloning:
```bash
git clone git@github.com:username/repo-name.git
```


**📚 Learn more:**
- [GitHub Docs – Connecting to GitHub with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)

## 25. Pushing Code to GitHub

```bash
git push -u origin main       # First push (sets upstream tracking)
git push                      # Subsequent pushes
```


**📚 Learn more:**
- [git-scm.com – git push reference](https://git-scm.com/docs/git-push)
- [GitHub Docs – Pushing commits to a remote repository](https://docs.github.com/en/get-started/using-git/pushing-commits-to-a-remote-repository)

## 26. Cloning a Repository

Cloning downloads a full copy of an existing repository, including all its history:
```bash
git clone https://github.com/username/repo-name.git
```


**📚 Learn more:**
- [GitHub Docs – Cloning a repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository)

## 27. Cloning a Private Repository

Private repositories require authentication:
```bash
git clone https://github.com/username/private-repo.git
```
- You'll be prompted for a **username** and a **Personal Access Token** (not your account password).
- Alternatively, use SSH authentication (see Section 24) for a smoother experience.


**📚 Learn more:**
- [GitHub Docs – Cloning a repository (auth details)](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository)

## 28. Pulling and Fetching Updates

```bash
git fetch              # Download changes from remote WITHOUT merging them
git pull                # Download AND automatically merge changes into your current branch
git pull origin main    # Pull specifically from the main branch of "origin"
```

**Rule of thumb**: `fetch` is "look but don't touch," `pull` is "look and apply."


**📚 Learn more:**
- [git-scm.com – git pull reference](https://git-scm.com/docs/git-pull)
- [git-scm.com – git fetch reference](https://git-scm.com/docs/git-fetch)

## 29. Working with Remote Branches

```bash
git branch -r                          # List remote branches
git push origin <branch-name>          # Push a local branch to remote
git checkout -b <branch> origin/<branch>  # Create a local branch tracking a remote one
git push origin --delete <branch-name>    # Delete a remote branch
```


**📚 Learn more:**
- [GitHub Docs – About remote repositories](https://docs.github.com/en/get-started/getting-started-with-git/about-remote-repositories)

## 30. Pull Requests (PRs) — Collaboration on GitHub

A **Pull Request** is how you propose merging your branch's changes into another branch (usually `main`) on GitHub, allowing others to review before merging.

Typical flow:
1. Push your feature branch to GitHub
2. Go to the repository on GitHub → click **Compare & pull request**
3. Add a title/description explaining your changes
4. Teammates review, comment, and request changes if needed
5. Once approved, click **Merge pull request**


**📚 Learn more:**
- [GitHub Docs – Creating a pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request)
- [GitHub Docs – About pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)

## 31. Forking a Repository

**Forking** creates your own personal copy of someone else's repository on GitHub (separate from cloning, which is just a local copy).

- Useful for contributing to open-source projects you don't have write access to.
- Workflow: Fork → Clone your fork locally → Make changes → Push to your fork → Open a Pull Request to the original repository.


**📚 Learn more:**
- [GitHub Docs – Fork a repo (quickstart)](https://docs.github.com/en/get-started/quickstart/fork-a-repo)

## 32. .gitignore File

A `.gitignore` file tells Git which files/folders to **never track** (e.g., temporary files, secrets, build outputs).

Example `.gitignore`:
```
node_modules/
.env
*.log
__pycache__/
.DS_Store
```


**📚 Learn more:**
- [GitHub Docs – Ignoring files](https://docs.github.com/en/get-started/getting-started-with-git/ignoring-files)
- [github/gitignore – template collection](https://github.com/github/gitignore)

## 33. Common Errors and Troubleshooting

| Error/Issue | Likely Cause | Fix |
|---|---|---|
| `fatal: not a git repository` | Not inside a Git-tracked folder | Run `git init` or `cd` into the correct folder |
| `Permission denied (publickey)` | SSH key not set up correctly | Re-check SSH key setup (Section 24) |
| `remote: Support for password authentication was removed` | Using password instead of PAT | Use a Personal Access Token (Section 23) |
| Push rejected: `Updates were rejected` | Remote has changes you don't have locally | Run `git pull` first, then push again |
| Merge conflict markers in file | Two branches edited the same lines | Resolve manually (Section 17) |
| `git status` shows nothing but files are missing | Files may be in `.gitignore` | Check your `.gitignore` file |

General troubleshooting steps:
```bash
git status        # Always start here
git remote -v      # Confirm remote URL is correct
git fetch          # Check what's changed on remote without altering local files
```


**📚 Learn more:**
- [GitHub Docs – Troubleshooting](https://docs.github.com/en/authentication/troubleshooting-ssh)
- [GitHub Community Discussions](https://github.com/orgs/community/discussions)

## 34. Complete Command Cheat Sheet

```bash
# Setup
git init
git config --global user.name "Name"
git config --global user.email "email"

# Staging & Committing
git add <file>
git add .
git commit -m "message"

# Viewing
git status
git log
git log --oneline
git diff

# Branching
git branch
git branch <name>
git checkout <name>
git checkout -b <name>
git merge <name>

# Remote
git remote add origin <url>
git remote -v
git push -u origin main
git push
git pull
git fetch
git clone <url>

# Undoing
git restore <file>
git reset --soft HEAD~1
git reset --hard HEAD~1
git revert <commit-hash>
```


**📚 Learn more:**
- [GitHub – Git Cheat Sheet (PDF)](https://education.github.com/git-cheat-sheet-education.pdf)

## 35. A Typical Real-World Workflow (Putting It All Together)

1. `git clone <repo-url>` — get the project onto your computer
2. `git checkout -b feature/login-page` — create a branch for your task
3. Make changes to files
4. `git add .` then `git commit -m "Add login page UI"`
5. `git push origin feature/login-page` — upload your branch to GitHub
6. Open a **Pull Request** on GitHub
7. Team reviews and approves → **Merge** into `main`
8. `git checkout main` then `git pull` — update your local `main` with the merged changes
9. Delete the old feature branch (locally and remotely) once it's merged


**📚 Learn more:**
- [GitHub Docs – GitHub flow](https://docs.github.com/en/get-started/using-github/github-flow)

## 36. Glossary of Key Terms

| Term | Definition |
|---|---|
| Repository (repo) | A project folder tracked by Git |
| Commit | A saved snapshot of changes |
| Branch | An independent line of development |
| Merge | Combining changes from one branch into another |
| Clone | Downloading a full copy of a remote repository |
| Fork | Creating your own copy of someone else's repository on GitHub |
| Remote | The online version of a repository (e.g., on GitHub) |
| Push | Uploading local commits to a remote |
| Pull | Downloading and merging remote changes |
| Fetch | Downloading remote changes without merging |
| Staging Area | Where changes wait before being committed |
| PAT (Personal Access Token) | A secure token used instead of a password for Git authentication |
| SSH Key | A cryptographic key pair used for secure, passwordless authentication |
| 2FA | Two-Factor Authentication — an extra login security step |
| Merge Conflict | When two branches change the same lines differently and Git needs manual resolution |
| Pull Request (PR) | A request to merge changes from one branch into another, with review |
| .gitignore | A file listing what Git should never track |
| HEAD | A pointer to your current position (commit) in the repository |


**📚 Learn more:**
- [GitHub Docs – Glossary](https://docs.github.com/en/get-started/learning-about-github/github-glossary)

## 37. Additional Resources & Further Reading

All the best official and free resources in one place, organized by what you'll want to learn next.

**Official documentation (best starting point):**
- [git-scm.com – Official Git Documentation](https://git-scm.com/doc)
- [Pro Git Book – free, complete, official (also available in 30+ languages)](https://git-scm.com/book/en/v2)
- [GitHub Docs – the full GitHub documentation hub](https://docs.github.com/)
- [GitHub Docs – "Getting started with Git" learning path](https://docs.github.com/en/get-started/getting-started-with-git)
- [GitHub Docs – "Using Git" learning path](https://docs.github.com/en/get-started/using-git)

**Interactive / hands-on practice:**
- [Learn Git Branching – visual, interactive branching simulator](https://learngitbranching.js.org/)
- [GitHub Skills – free interactive courses run inside real GitHub repos](https://skills.github.com/)
- [Katacoda-style Git sandbox via GitHub Docs quickstart](https://docs.github.com/en/get-started/quickstart)

**Cheat sheets:**
- [GitHub's official Git Cheat Sheet (PDF)](https://education.github.com/git-cheat-sheet-education.pdf)
- [Atlassian Git Cheat Sheet](https://www.atlassian.com/git/tutorials/atlassian-git-cheatsheet)

**Deeper tutorials by topic:**
- [Atlassian Git Tutorials (topic-by-topic, well illustrated)](https://www.atlassian.com/git/tutorials)
- [GitHub Docs – GitHub flow (branching workflow used by most teams)](https://docs.github.com/en/get-started/using-github/github-flow)
- [GitHub Docs – Collaborating with pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests)
- [GitHub Docs – Connecting to GitHub with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [GitHub Docs – Keeping your account and data secure](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure)

**Useful tools:**
- [github/gitignore – official template repository for `.gitignore` files](https://github.com/github/gitignore)
- [GitHub CLI (`gh`) – manage GitHub from your terminal](https://cli.github.com/)
- [GitHub Desktop – GUI alternative to the command line](https://desktop.github.com/)

**If you get stuck:**
- [GitHub Community Discussions – ask questions, search past answers](https://github.com/orgs/community/discussions)
- [Stack Overflow – git tag](https://stackoverflow.com/questions/tagged/git)

## 38. Revision Summary

Quick recap before you start practicing:

- **Version control** tracks changes over time; Git is **distributed**, meaning every user has the full project history locally.
- **Git ≠ GitHub**: Git is the tool that runs on your computer; GitHub is the website that hosts your Git repositories online.
- **Three-stage workflow**: edit files (working directory) → `git add` (staging) → `git commit` (repository).
- **Branches** let you work safely in isolation; **merging** brings that work back together; **conflicts** must be resolved manually when the same lines were changed differently.
- **GitHub account security**: use phone verification and **2FA**.
- **Authentication for pushing/pulling**: use a **Personal Access Token** (HTTPS) or **SSH keys** — plain passwords no longer work.
- **Core remote commands**: `clone` (copy a repo), `push` (upload commits), `pull` (download + merge), `fetch` (download only).
- **Collaboration on GitHub**: use **Pull Requests** to propose and review changes; use **Forking** to contribute to projects you don't own.
- **`.gitignore`** keeps unwanted files (secrets, build files, logs) out of version control.
- When something breaks: run `git status` first, then `git remote -v`, then `git fetch` — most issues become clear from there.

**One-line takeaway:** *Git tracks your project's history locally in commits and branches; GitHub hosts that history online and adds tools (PRs, tokens, 2FA) for secure collaboration with others.*
