# Part 1: Local Version Control with Git

Welcome to the Git module. By the end of this section you will understand how Git tracks your work, why data scientists should care, and how to use it confidently from the command line. We will build up from first principles — no prior experience required.

---

## Table of Contents

- [Before You Begin: Platform Setup](#before-you-begin-platform-setup)

1. [What is Version Control?](#1-what-is-version-control)
2. [Setting Up Git](#2-setting-up-git)
3. [The Git Mental Model: Three Stages](#3-the-git-mental-model-three-stages)
4. [Creating Your First Repository](#4-creating-your-first-repository)
5. [Staging and Committing](#5-staging-and-committing)
6. [Viewing History and Changes](#6-viewing-history-and-changes)
7. [Ignoring Files with `.gitignore`](#7-ignoring-files-with-gitignore)
8. [Undoing Changes](#8-undoing-changes)
9. [Branching](#9-branching)
10. [Merging](#10-merging)
11. [Stashing](#11-stashing)
12. [Advanced: Rebase](#12-advanced-rebase)
13. [Tags](#13-tags)
14. [Useful Everyday Commands](#14-useful-everyday-commands)
15. [Summary](#summary)

---

## Before You Begin: Platform Setup

The commands in this guide are written in Unix/bash style. Read this section once so you know what to expect on your platform.

### Opening a terminal

| Platform | Terminal to use |
|---|---|
| **macOS** | Applications → Utilities → **Terminal** (or search "Terminal" in Spotlight). Default shell is **zsh** since macOS Catalina. |
| **Windows** | Install Git for Windows (see below) — it includes **Git Bash**, which gives you a Unix-style terminal. All commands in this guide work in Git Bash. |
| **Ubuntu / Linux** | Open your system terminal (GNOME Terminal, Konsole, etc.). Default shell is usually **bash**. |

### Installing Git

**macOS:**
```bash
# Option 1 — Homebrew
brew install git

# Option 2 — Xcode Command Line Tools (no Homebrew needed)
xcode-select --install
```

**Windows:** Download and run the installer from [git-scm.com/download/win](https://git-scm.com/download/win). During setup:
- Select **"Use the OpenSSH library"** for SSH key management
- On the line endings screen, select **"Checkout Windows-style, commit Unix-style line endings"**
- This installs Git Bash — use that for all commands in this guide

**Ubuntu / Linux:**
```bash
sudo apt update && sudo apt install git
```

### Which terminal should I use?

| Platform | Recommendation |
|---|---|
| macOS | Terminal or iTerm2 — all commands work as shown |
| Windows — Git Bash | All commands work as shown — use this |
| Windows — PowerShell | Git commands work, but Unix utilities like `ls`, `rm`, `cat` differ |
| Windows — CMD | Not recommended — use Git Bash instead |
| Ubuntu / Linux | Any terminal — all commands work as shown |

> **Windows users:** Use **Git Bash** for everything in this guide. It gives you the same Unix commands used on macOS and Linux.

---

## 1. What is Version Control?

Picture this: you are working on a machine learning model. You have a file called `model.py`. Over the course of a week it looks like this:

```
model.py
model_v2.py
model_v2_FINAL.py
model_v2_FINAL_really.py
model_v2_FINAL_really_fixed.py
model_v2_USE_THIS_ONE.py
```

Look familiar? This is the problem version control solves.

**Version control** is a system that records changes to your files over time so you can recall specific versions later. Think of it like the "tracked changes" feature in a Word document, but for entire projects, and infinitely more powerful.

### The Problems It Solves

| Problem | Without Git | With Git |
|---|---|---|
| "I broke something — how do I go back?" | Pray you saved a copy | `git restore` or `git revert` |
| "What did I change last Tuesday?" | No idea | `git log` + `git diff` |
| "I want to try a risky idea without breaking my working code" | Duplicate the whole folder | Create a `git branch` in seconds |
| "My colleague and I both edited the same file" | Manual copy-paste nightmare | Git handles merging |
| "My laptop died" | Data loss | `git push` to remote and recover instantly |

> **Analogy:** Imagine you are packing a suitcase for a trip. Normally, you just throw things in and zip it up (saving a file). Git is like having a personal assistant who photographs everything in the suitcase at each stage, labels each photo with a note about what changed and why, and stores every photo forever. At any point you can say "show me what the suitcase looked like on Tuesday" — and you will see it exactly as it was.

### Git: A Distributed Version Control System

Git is the most widely used version control system in the world. It was created by Linus Torvalds (the creator of Linux) in 2005.

Git is **distributed**, which means every person who works on a project has a full copy of the entire history on their own machine. There is no single point of failure. You can work completely offline and sync later.

This is different from older systems (like SVN) where there was one central server and if it went down, no one could work.

---

## 2. Setting Up Git

Before you do anything else, you need to introduce yourself to Git. Git uses your name and email to label every change you make (called a "commit"). This is how you know who changed what.

### Configuring your identity

```bash
git config --global user.name "Your Name"
git config --global user.email "your.name@example.com"
```

The `--global` flag means this setting applies to every Git project on your computer. You only need to do this once.

### Configuring line endings

Windows and macOS/Linux use different characters to mark the end of a line. This causes problems when collaborating. Tell Git how to handle them:

**macOS / Linux / Windows (Git Bash):**
```bash
git config --global core.autocrlf input
```

**Windows (PowerShell or CMD with native Git):**
```bash
git config --global core.autocrlf true
```

| Setting | What it does |
|---|---|
| `input` | Converts Windows-style line endings to Unix on commit; never converts on checkout |
| `true` | Converts to Unix on commit; converts back to Windows on checkout |

> **Tip:** If you are using Git Bash on Windows, use `input` — it behaves like macOS and Linux.

### Setting your default editor

When Git needs you to write a longer message (like during a merge), it opens a text editor. Set it to something you are comfortable with:

**macOS / Linux (Git Bash on Windows):**
```bash
# nano — simple, terminal-based, recommended for beginners
git config --global core.editor "nano"

# VS Code — works on all platforms if the `code` command is in your PATH
git config --global core.editor "code --wait"
```

**Windows (PowerShell or CMD):**
```bash
# VS Code
git config --global core.editor "code --wait"

# Notepad (built-in, simple)
git config --global core.editor "notepad"

# Notepad++ (adjust path if installed elsewhere)
git config --global core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -notabbar -nosession"
```
### Verifying your configuration

```bash
git config --list
```

Example output:

```
user.name=Your Name
user.email=your.name@example.com
core.editor=nano
core.autocrlf=input
```

> **Tip:** Your name and email will appear in the commit history of every project you contribute to. If you are planning to push code to GitHub, use the same email address you use for your GitHub account.

---

## 3. The Git Mental Model: Three Stages

This is the most important concept to understand before you write a single Git command. Git has three distinct places where your files can live:

```
+-------------------+     git add      +-------------------+    git commit    +-------------------+
|                   |  ------------->  |                   |  ------------->  |                   |
|  Working          |                  |  Staging Area     |                  |    Repository     |
|  Directory        |  <-----------    |  (Index)          |                  |   (.git folder)   |
|                   |   git restore    |                   |                  |                   |
|  Your actual      |                  |  Files you've     |                  |   Permanent       |
|  files on disk    |   git restore    |  marked "ready    |                  |   history of      |
|                   |    --staged      |  to commit"       |                  |   snapshots       |
+-------------------+                  +-------------------+                  +-------------------+
```

### The packing-a-box analogy

Imagine you are moving house. You have a room full of items (your **Working Directory**). When you decide something is ready to go, you put it in the box (the **Staging Area**). When the box is full of exactly what you want, you seal it and label it (that is a **Commit** stored in the **Repository**).

The staging area is the key insight. It lets you be deliberate: you might have changed five files, but only want to commit three of them together because they form one logical unit of work. The staging area gives you that control.

| Stage | Also called | What it means |
|---|---|---|
| Working Directory | Working Tree | Your files as they currently exist on disk |
| Staging Area | Index | A preview of what your next commit will contain |
| Repository | `.git/` folder | The permanent record of every commit ever made |

---

## 4. Creating Your First Repository

### Starting a new project

Open the terminal and navigate to a free space you have reserved for projects.

```bash
# Create a new folder for your project
mkdir my_ds_project
cd my_ds_project

# Initialise a Git repository
git init
```

Output:

```
Initialized empty Git repository in /home/name/my_ds_project/.git/
```

That is it. Your folder is now a Git repository.

### What is the `.git/` folder?

When you run `git init`, Git creates a hidden folder called `.git/` inside your project directory. This folder is where Git stores everything: the full history of every commit, your branches, your configuration. You should never manually edit or delete anything inside `.git/`. Just know it is there doing its job quietly.

Look at it by using the command:

```bash
ls -la
```
```
total 0
drwxr-xr-x  3 jane  staff   96  1 May 10:00 .
drwxr-xr-x  8 jane  staff  256  1 May 10:00 ..
drwxr-xr-x  9 jane  staff  288  1 May 10:00 .git
```

### `git status` — Your New Best Friend

Run this constantly. It tells you the state of your working directory and staging area.

```bash
git status
```
```
On branch main

No commits yet

nothing to commit (create/copy files to get started)
```

### Full Walkthrough

Let us create a real Data Science project from scratch:

```bash
mkdir sales_analysis
cd sales_analysis
git init
```

Now create a Python file:

```bash
# Create a simple analysis script
cat > analysis.py << 'EOF'
Print("Hello World!")
EOF
```
Check the status:
```bash
git status
```
```
On branch main

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	analysis.py

nothing added to commit but untracked files present (use "git add" to track)
```

Git sees the file but is not tracking it yet. Files start as **untracked**. You must explicitly tell Git which files to follow.

---

## 5. Staging and Committing

Now we will move files through the three stages.

### Adding a single file

```bash
git add analysis.py
```

Check the status again:
```bash
git status
```
```
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   analysis.py
```

The file is now in the **Staging Area** — it is ready to be committed.

### Adding everything at once

```bash
git add .
```

The `.` means "everything in this directory and below". Be careful with this — make sure you have a `.gitignore` set up (covered in Section 7) so you do not accidentally add files you do not want tracked.

### Interactive staging with `git add -p`

This is a powerful tool for the careful developer. Instead of staging entire files, you can stage individual **hunks** (sections of changes) within a file:

```bash
git add -p analysis.py
```

The options:
- `y` — stage this hunk
- `n` — skip this hunk
- `s` — split into smaller hunks
- `q` — quit

This is useful when you have made several unrelated changes to one file and want to commit them as separate logical units.

### Making your first commit

```bash
git commit -m "Add hello message"
```

Output:

```
[main (root-commit) a1b2c3d] Add hello message
 1 file changed, 1 insertion(+)
 create mode 100644 analysis.py
```

That hash (`a1b2c3d`) is the unique identifier for this commit. Every commit has one.

### Committing with a longer message

For more complex commits, drop the `-m` flag and Git will open your editor:

```bash
git commit
```

Your editor opens with a template:

```

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch main
# Changes to be committed:
#	modified:   analysis.py
#
```

Write your message, save, and close the editor. The commit is created.

### Writing good commit messages

A commit message is a note to your future self (and your colleagues). Good messages save enormous amounts of time when debugging.

**Format:**

```
Short summary line (50 characters or fewer)

Optional longer description that explains WHAT changed and WHY.
Not how — the code shows how. Focus on the reasoning.
Wrap lines at 72 characters.
```

**Examples — bad vs good:**

```
# Bad
fix stuff
update
changes
wip

# Good
Add revenue aggregation by product category
Remove outlier filter — it was excluding valid Q4 sales
Fix KeyError when sales.csv has missing 'region' column
Refactor model training loop to use DataLoader
```

> **Tip:** Write commit messages in the **imperative mood** — "Add feature", not "Added feature" or "Adds feature". This reads as an instruction: "If I apply this commit, it will Add feature." This is the convention used by Git itself and most professional teams.

### Full example walkthrough

```bash
# Start fresh
mkdir notebook_project
cd notebook_project
git init

# Create some files
touch notebook.ipynb model.py requirements.txt

# Check status
git status
# Output: 3 untracked files

# Stage only the Python files
git add model.py requirements.txt
git status
# Output: 2 files staged, notebook.ipynb still untracked

# Commit
git commit -m "Add model script and requirements"

# Now add the notebook
git add notebook.ipynb
git commit -m "Add exploratory analysis notebook"

# Check your history
git log --oneline
# a2f3b1c Add exploratory analysis notebook
# 9e1d4f2 Add model script and requirements
```

---

## 6. Viewing History and Changes

### `git log` — the full history

```bash
git log
```

```
commit a2f3b1c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0
Author: Jane Smith <jane.smith@example.com>
Date:   Mon May 1 10:30:00 2024 +0200

    Add exploratory analysis notebook

commit 9e1d4f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8
Author: Jane Smith <jane.smith@example.com>
Date:   Mon May 1 10:15:00 2024 +0200

    Add model script and requirements
```

### `git log --oneline` — compact view

```bash
git log --oneline
```

```
a2f3b1c Add exploratory analysis notebook
9e1d4f2 Add model script and requirements
```

Much easier to scan. The short hash on the left is all you need to reference a commit in most commands.

### `git log --oneline --graph --all` — visual branch graph

```bash
git log --oneline --graph --all
```

```
* f8a2d3e (HEAD -> main) Fix column name typo in analysis
* a2f3b1c Add exploratory analysis notebook
| * 7c1b9f4 (feature/visualisations) Add matplotlib plots
|/
* 9e1d4f2 Add model script and requirements
```

The `*` and `|` characters show you the branching structure visually. You will use this constantly once you start branching.

### `git diff` — what has changed but is not yet staged

Make a change to a file, then:

```bash
git diff
```

```diff
diff --git a/analysis.py b/analysis.py
index 3a4b5c6..7d8e9f0 100644
--- a/analysis.py
+++ b/analysis.py
@@ -1,4 +1,7 @@
 import pandas as pd
+import numpy as np

 df = pd.read_csv("data/sales.csv")
+df = df.dropna()
 print(df.describe())
```

Lines starting with `+` are additions. Lines starting with `-` are removals.

### `git diff --staged` — what is staged and about to be committed

```bash
git add analysis.py
git diff --staged
```

This shows the same format but only for changes you have already staged.

### `git show <commit-hash>` — inspect a specific commit

```bash
git show 9e1d4f2
```

This shows the commit metadata (author, date, message) plus the full diff of what changed in that commit.

### Filtering the log

```bash
# Show commits by a specific author
git log --author="Jane"

# Show commits from the last two weeks
git log --since="2 weeks ago"

# Combine filters
git log --author="Jane" --since="2024-01-01" --oneline

# Show commits that touched a specific file
git log --oneline -- analysis.py
```

---

## 7. Ignoring Files with `.gitignore`

Not everything in your project directory should be tracked. Some files are too large, too sensitive, or simply auto-generated noise.

### Files you almost never want to commit

| File / Pattern | Why to ignore |
|---|---|
| `.DS_Store` | macOS metadata, useless to everyone else |
| `__pycache__/` | Python bytecode cache, regenerated automatically |
| `.env` | Contains secrets like API keys and passwords |
| `*.csv`, `*.parquet` | Large data files — use DVC or a data store instead |
| `venv/`, `.venv/` | Virtual environment — recreated from `requirements.txt` |
| `.ipynb_checkpoints/` | Jupyter auto-save files |
| `*.log` | Log files that grow over time |
| `models/*.pkl` | Large serialised model files |

> **Warning:** Never commit a `.env` file. If your API keys or database passwords end up in a public GitHub repository, they can be found by bots within seconds and abused. A `.gitignore` is your first line of defence.

### Creating a `.gitignore`

Create a file called `.gitignore` at the root of your project:

```bash
touch .gitignore
```

Open it in your editor and add patterns — one per line.

### Pattern syntax

```
# This is a comment

# Ignore a specific file
.env

# Ignore all files with this extension
*.csv
*.log
*.pkl

# Ignore an entire directory
data/raw/
venv/
__pycache__/
.ipynb_checkpoints/

# Ignore all .txt files in any subdirectory
**/*.txt

# Exception: do NOT ignore this specific file even though *.csv is ignored
!data/sample_small.csv

# Ignore files only at the root level (not in subdirectories)
/config.local.py
```

### A practical `.gitignore` for a Python/DS project

```gitignore
# === Python ===
__pycache__/
*.py[cod]
*.pyo
*.pyd
.Python

# === Virtual Environments ===
venv/
.venv/
env/
.env/

# === Environment variables / secrets ===
.env
.env.local
.env.*.local
secrets.json

# === Jupyter ===
.ipynb_checkpoints/
*.ipynb_checkpoints

# === Data files (track with DVC instead) ===
data/raw/
data/processed/
*.csv
*.parquet
*.feather
*.h5
*.hdf5

# === Model artefacts ===
models/*.pkl
models/*.joblib
models/*.pt
models/*.h5

# === Logs and outputs ===
logs/
*.log
outputs/
reports/figures/

# === OS and editor clutter ===
.DS_Store
Thumbs.db
.idea/
.vscode/
*.swp
```

Once you save this file, Git will start ignoring the listed patterns. You should commit your `.gitignore`:

```bash
git add .gitignore
git commit -m "Add .gitignore for Python/DS project"
```

### Debugging ignored files

If a file is being ignored and you are not sure why:

```bash
git check-ignore -v data/sales.csv
```

```
.gitignore:14:*.csv	data/sales.csv
```

This tells you exactly which line in which `.gitignore` file is responsible for ignoring it.

---

## 8. Undoing Changes

This is where many beginners feel nervous. Let us demystify it completely. There are several ways to undo things in Git, and choosing the right one depends on exactly what state you are in.

### Situation 1: You changed a file but have NOT staged it yet

You edited `model.py`, made a mess, and want to go back to the last committed version:

```bash
git restore model.py
```

> **Warning:** This permanently discards your uncommitted changes to that file. There is no undo for this. Make sure you really want to throw away those changes before running it.

### Situation 2: You staged a file but have NOT committed yet

You ran `git add model.py` but now want to unstage it (keep your changes, just remove it from staging):

```bash
git restore --staged model.py
```

The file goes back to the working directory with your changes intact. Nothing is lost — it is just no longer queued for the next commit.

### Situation 3: You want to fix your last commit message (or add a forgotten file)

You made a typo in your commit message, or forgot to include a file:

```bash
# Fix the message
git commit --amend -m "Correct commit message here"

# Or open the editor to rewrite it
git commit --amend

# Add a forgotten file to the last commit
git add forgotten_file.py
git commit --amend --no-edit
```

> **Warning:** Only use `--amend` on commits you have NOT yet pushed to a shared repository. Amending rewrites history. If someone else has already pulled that commit, amending will cause conflicts.

### Situation 4: You want to undo a commit safely (after possibly pushing)

`git revert` creates a **new commit** that undoes the changes from a previous one. It does not rewrite history, so it is safe to use even on shared branches.

```bash
# Find the hash of the commit you want to undo
git log --oneline
# a2f3b1c Bad analysis — wrong filter applied
# 9e1d4f2 Add model script and requirements

# Revert it
git revert a2f3b1c
```

Git will open your editor to write the revert commit message (or you can use `-m`). A new commit is created that undoes everything from `a2f3b1c`. The original commit stays in the history.

### Situation 5: `git reset` — the powerful (and dangerous) undo

`git reset` moves the branch pointer backwards. Use this for local work only.

```bash
# Undo the last commit, but keep the changes STAGED
git reset --soft HEAD~1

# Undo the last commit, keep the changes in the working directory (UNSTAGED)
git reset --mixed HEAD~1    # this is the default if you omit the flag

# Undo the last commit AND discard all the changes completely
git reset --hard HEAD~1
```

`HEAD~1` means "one commit before the current position". You can also use a commit hash directly.

> **Warning:** `git reset --hard` is the most dangerous Git command for beginners. It discards your work permanently (unless you have it in `git reflog` — see Section 14). Do not use it unless you are certain you want to throw away those changes.

### Quick reference table

| Situation | Command | Safe? | Rewrites history? |
|---|---|---|---|
| Discard unstaged file changes | `git restore <file>` | Irreversible locally | No |
| Unstage a file | `git restore --staged <file>` | Yes — changes preserved | No |
| Fix last commit | `git commit --amend` | Only before pushing | Yes |
| Undo a commit (keep history clean) | `git revert <hash>` | Yes — always safe | No |
| Undo commit, keep staged | `git reset --soft HEAD~1` | Only local | Yes |
| Undo commit, keep unstaged | `git reset --mixed HEAD~1` | Only local | Yes |
| Undo commit, discard everything | `git reset --hard HEAD~1` | Dangerous | Yes |

---

## 9. Branching

A branch is simply a pointer to a specific commit. Creating a branch means you can develop a new idea, experiment, or feature in complete isolation from your main working code. If the experiment fails, delete the branch. If it succeeds, merge it in.

This is one of the most powerful ideas in Git. In a Data Science context: try that new feature engineering approach on a branch. If it does not improve the model, delete the branch and your `main` is untouched.

### Listing branches

```bash
git branch
```

```
* main
```

The `*` shows your current branch.

### Creating a new branch

```bash
git branch feature/add-visualisations
```

This creates the branch but does NOT switch to it.

### Switching to a branch

```bash
git switch feature/add-visualisations
```

```
Switched to branch 'feature/add-visualisations'
```

> **Note:** You may also see the older `git checkout feature/add-visualisations` in documentation and tutorials. `git switch` was introduced in Git 2.23 as a cleaner, less confusing alternative. Prefer `git switch` in new work.

### Create and switch in one step

```bash
git switch -c experiment/new-model-architecture
```

This is the shortcut you will use most often.

### Seeing all branches with their latest commits

```bash
git branch -v
```

```
* main                        9e1d4f2 Add model script and requirements
  feature/add-visualisations  a2f3b1c Add exploratory analysis notebook
```

### Deleting a branch

Once you have merged a branch, clean it up:

```bash
# Safe delete — only works if the branch has been merged
git branch -d feature/add-visualisations

# Force delete — works even if the branch has not been merged
git branch -D experiment/failed-approach
```

### Naming conventions

Good branch names make it clear what the branch is for at a glance:

| Prefix | Used for | Example |
|---|---|---|
| `feature/` | New functionality | `feature/add-shap-values` |
| `fix/` | Bug fixes | `fix/nan-handling-in-preprocess` |
| `experiment/` | Exploratory ideas | `experiment/lstm-baseline` |
| `chore/` | Maintenance tasks | `chore/update-requirements` |
| `docs/` | Documentation only | `docs/add-readme` |

Use lowercase letters and hyphens. Avoid spaces and special characters.

---

## 10. Merging

Once your branch work is complete, you bring it back into `main` (or whichever branch you want to incorporate the work into) using `git merge`.

The general workflow is:

```bash
# 1. Switch to the branch you want to merge INTO
git switch main

# 2. Merge the other branch
git merge feature/add-visualisations
```

### Fast-forward merge

A fast-forward merge happens when the branch being merged is directly ahead of the current branch — there is a straight line of commits with no divergence.

```
Before merge:

main:    A --- B
                \
feature:         C --- D

After fast-forward merge:

main:    A --- B --- C --- D
```

```bash
git switch main
git merge feature/add-visualisations
```

```
Updating 9e1d4f2..a2f3b1c
Fast-forward
 analysis.py | 5 +++++
 1 file changed, 5 insertions(+)
```

Git simply moves the `main` pointer forward. No new commit is created.

### Three-way merge

A three-way merge happens when both branches have diverged — each has commits the other does not have.

```
Before merge:

main:    A --- B --- E
                \
feature:         C --- D

After three-way merge:

main:    A --- B --- E --- M   (M is the merge commit)
                \         /
feature:         C --- D
```

```bash
git switch main
git merge feature/new-features
```

```
Merge made by the 'ort' strategy.
 model.py | 12 ++++++++++++
 1 file changed, 12 insertions(+)
```

Git creates a new **merge commit** that has two parents.

### Merge conflicts

A conflict occurs when both branches changed the same part of the same file, and Git cannot figure out which version to keep. Git stops and asks you to decide.

```bash
git merge feature/add-visualisations
```

```
Auto-merging analysis.py
CONFLICT (content): Merge conflict in analysis.py
Automatic merge failed; fix conflicts and then commit the result.
```

Open `analysis.py` and you will see conflict markers:

```python
import pandas as pd
import numpy as np

df = pd.read_csv("data/sales.csv")

<<<<<<< HEAD
df = df.dropna(subset=['revenue'])
print(df.describe())
=======
df = df.fillna(0)
df['revenue'].plot()
>>>>>>> feature/add-visualisations
```

Reading the conflict:
- `<<<<<<< HEAD` — start of your current branch's version
- `=======` — divider between the two versions
- `>>>>>>> feature/add-visualisations` — end of the incoming branch's version

### Resolving a conflict step by step

1. Open the conflicted file in your editor
2. Decide what the final version should look like (you can keep one side, the other, or combine both)
3. Delete the conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`)
4. Save the file

The resolved version might look like:

```python
import pandas as pd
import numpy as np

df = pd.read_csv("data/sales.csv")

# Keep NaN rows with revenue=0 but drop rows with no revenue column at all
df = df.dropna(subset=['revenue'])
df = df.fillna(0)
df['revenue'].plot()
print(df.describe())
```

5. Stage the resolved file and commit:

```bash
git add analysis.py
git commit -m "Merge feature/add-visualisations — combine NaN handling approaches"
```

### Aborting a merge

If you get into a conflict and want to start over:

```bash
git merge --abort
```

This puts everything back to how it was before you started the merge.

> **Tip:** Conflicts are less scary than they look. They are just Git asking you "I found two different versions of this — which do you want?" Take your time, read both sides carefully, and decide.

---

## 11. Stashing

Stashing is a way to temporarily save your uncommitted changes so you can switch context — for example, to fix an urgent bug on another branch — without committing half-finished work.

### The scenario

You are halfway through building a new feature on `feature/add-charts` when a colleague asks you to urgently check something on `main`. Your working directory has changes you are not ready to commit yet.

```bash
git status
```

```
On branch feature/add-charts
Changes not staged for commit:
	modified:   analysis.py
	modified:   model.py
```

### Saving your work with `git stash`

```bash
git stash
```

```
Saved working directory and index state WIP on feature/add-charts: a2f3b1c Add notebook
```

Your working directory is now clean. You can switch branches freely.

You can also give your stash a descriptive name:

```bash
git stash push -m "WIP: adding revenue chart to analysis"
```

### Listing stashes

```bash
git stash list
```

```
stash@{0}: WIP on feature/add-charts: a2f3b1c Add notebook
stash@{1}: On main: quick experiment with log transform
```

Stashes are numbered from newest (`{0}`) to oldest.

### Restoring your work

```bash
# Apply the most recent stash and remove it from the stash list
git stash pop

# Apply a specific stash (keeps it in the list)
git stash apply stash@{1}
```

`pop` = apply + delete. `apply` = apply but keep the stash around (useful if you want to apply the same stash to multiple branches).

### Deleting a stash

```bash
git stash drop stash@{0}
```

### Creating a branch from a stash

If your stashed work grows into something bigger, you can turn it directly into a branch:

```bash
git stash branch experiment/log-transform stash@{1}
```

This creates a new branch at the commit where you created the stash, applies the stash to it, and checks it out — all in one step.

> **Tip:** Do not accumulate dozens of stashes. If a piece of work is worth saving, it is worth committing (even as a work-in-progress commit with a message like "WIP: rough draft of feature X"). Stash is for short-lived context switches, not long-term storage.

---

## 12. Advanced: Rebase

Rebasing is a more advanced technique for incorporating changes from one branch into another. It gives you a cleaner, more linear history than merging — but comes with important rules.

### What is rebasing?

When you rebase branch A onto branch B, Git takes all the commits unique to branch A and **replays** them, one by one, on top of branch B. The result looks like your branch was always based on the latest version of B.

**Merging** (creates a merge commit, preserves exact history):
```
main:    A --- B --- E --- M
                \         /
feature:         C --- D
```

**Rebasing** (linear history, looks cleaner):
```
main:    A --- B --- E
                          \
feature:                   C' --- D'
```

The commits `C'` and `D'` are new commits with the same changes as `C` and `D`, but with different parent commits (and therefore different hashes).

### Basic rebase

```bash
# You are on feature/new-model, and main has moved ahead
git switch feature/new-model
git rebase main
```

This replays your feature branch commits on top of the current tip of `main`.

If there are conflicts, Git will pause and let you resolve them one commit at a time:

```bash
# After resolving conflicts in the file:
git add model.py
git rebase --continue

# Or if you want to give up entirely:
git rebase --abort
```

### Interactive rebase — cleaning up your history

Interactive rebase is one of the most useful Git tools for keeping a tidy, readable history. It lets you edit, reorder, squash, or drop commits before you share them with others.

```bash
# Interactively edit the last 3 commits
git rebase -i HEAD~3
```

Your editor opens with a list like this:

```
pick 7a1b2c3 Add data loading function
pick 4d5e6f7 Fix typo in column name
pick 8a9b0c1 Add model training loop

# Rebase 9e1d4f2..8a9b0c1 onto 9e1d4f2 (3 commands)
#
# Commands:
# p, pick   = use commit
# r, reword = use commit, but edit the commit message
# s, squash = melt into previous commit, combining messages
# f, fixup  = like squash, but discard this commit's message
# d, drop   = remove commit entirely
```

**Common use cases:**

**Squash several messy commits into one clean one:**
```
pick 7a1b2c3 Add data loading function
squash 4d5e6f7 Fix typo in column name
squash 8a9b0c1 Actually fix the right typo
```

This turns three commits into one, combining their messages (which you can edit).

**Reword a commit message:**
```
reword 7a1b2c3 Add data loading function
pick 4d5e6f7 Fix typo in column name
```

**Drop a commit entirely:**
```
pick 7a1b2c3 Add data loading function
drop 4d5e6f7 Debugging print statement — remove
pick 8a9b0c1 Add model training loop
```

Save and close the editor, and Git executes your instructions.

### The golden rule of rebasing

> **Warning — the golden rule:** Never rebase commits that have already been pushed to a shared branch. Rebasing rewrites commit hashes. If your colleagues have already pulled those commits and you rebase and force-push, their history becomes incompatible with the remote. This causes serious headaches. Rebase is for cleaning up **local, private** history before you share it.

A safe workflow: rebase your feature branch onto `main` locally (to incorporate the latest changes), then merge or open a pull request. Never rebase `main` itself.

---

## 13. Tags

Tags are like permanent bookmarks on specific commits. Unlike branches, tags do not move as you make new commits. They are used to mark significant points in history — typically releases or milestones.

### Lightweight tag

A lightweight tag is just a pointer to a commit, with no extra information:

```bash
git tag v1.0
```

### Annotated tag

An annotated tag is a full Git object with its own message, author, and timestamp — preferred for releases:

```bash
git tag -a v1.0 -m "First stable release of the sales analysis pipeline"
```

### Listing tags

```bash
git tag
```

```
v0.1
v0.2
v1.0
```

You can filter by pattern:

```bash
git tag -l "v1.*"
```

### Inspecting a tag

```bash
git show v1.0
```

```
tag v1.0
Tagger: Jane Smith <jane.smith@example.com>
Date:   Fri May 3 14:00:00 2024 +0200

First stable release of the sales analysis pipeline

commit f8a2d3e...
Author: Jane Smith <jane.smith@example.com>
Date:   Fri May 3 13:55:00 2024 +0200

    Finalise model evaluation metrics
...
```

### Tagging a past commit

```bash
# Tag an older commit using its hash
git tag -a v0.9 9e1d4f2 -m "Pre-release version"
```

### When to use tags

- Marking a model version that was used for a specific report or presentation
- Marking the state of code when you deployed to production
- Marking milestones in a long project (`v1.0-baseline`, `v2.0-with-nlp`)
- Anywhere you need a permanent, human-readable reference to a specific point in history

> **Tip:** Tags are not automatically pushed to a remote repository. When you push to GitHub or GitLab, use `git push origin v1.0` or `git push origin --tags` to push all tags.

---

## 14. Useful Everyday Commands

These are the commands you will reach for regularly once the basics feel natural.

### Full visual history

```bash
git log --oneline --graph --all --decorate
```

```
* f8a2d3e (HEAD -> main, tag: v1.0) Finalise model evaluation metrics
* a2f3b1c Add exploratory analysis notebook
| * 7c1b9f4 (feature/visualisations) Add matplotlib plots
|/
* 9e1d4f2 Add model script and requirements
```

Alias this to something short — you will use it constantly:

```bash
git config --global alias.lg "log --oneline --graph --all --decorate"
# Now you can just type:
git lg
```

### `git blame` — see who changed each line

Want to know why a specific line of code exists, or when it was last changed?

```bash
git blame analysis.py
```

```
9e1d4f2 (Jane Smith  2024-05-01 10:15:00 +0200 1) import pandas as pd
a2f3b1c (Jane Smith  2024-05-01 10:30:00 +0200 2) import numpy as np
7c1b9f4 (Bob Chen    2024-05-02 14:22:00 +0200 3)
7c1b9f4 (Bob Chen    2024-05-02 14:22:00 +0200 4) df = pd.read_csv("data/sales.csv")
```

Each line shows: commit hash | author | timestamp | line content.

Great for understanding context around a piece of code before you change it.

### `git grep` — search tracked files

Search for a string across all files tracked by Git:

```bash
git grep "dropna"
```

```
analysis.py:df = df.dropna(subset=['revenue'])
preprocess.py:    df = df.dropna()
```

Unlike plain `grep`, this automatically searches only tracked files, respects `.gitignore`, and can search through specific branches or commits:

```bash
# Search in a specific branch
git grep "dropna" feature/add-visualisations

# Search at a specific commit
git grep "dropna" HEAD~5
```

### `git shortlog` — contribution summary

```bash
git shortlog -sn
```

```
    34  Jane Smith
    12  Bob Chen
     5  Alice Wong
```

Shows how many commits each author has made. Useful for understanding who owns which parts of a project.

### `git reflog` — your safety net

The reflog is a local log of every time `HEAD` moved — every commit, merge, rebase, reset, checkout. It is your ultimate undo tool.

```bash
git reflog
```

```
f8a2d3e (HEAD -> main) HEAD@{0}: commit: Finalise model evaluation metrics
a2f3b1c HEAD@{1}: commit: Add exploratory analysis notebook
9e1d4f2 HEAD@{2}: reset: moving to HEAD~1
b7c3d1f HEAD@{3}: commit: Oops — experiment gone wrong
...
```

Imagine you ran `git reset --hard HEAD~1` and lost a commit. The reflog shows you that the lost commit was `b7c3d1f`. You can recover it:

```bash
git switch -c recovery-branch b7c3d1f
```

> **Tip:** The reflog is local only — it is not pushed to remotes. It is retained for 90 days by default. Within that window, almost nothing in Git is truly lost.

### `git bisect` — find which commit introduced a bug

`git bisect` performs a **binary search** through your commit history to find exactly which commit introduced a bug. This is astonishingly useful when you know something broke somewhere in the last 200 commits but have no idea where.

```bash
# Start bisecting
git bisect start

# Tell Git: the current state is broken
git bisect bad

# Tell Git: this old commit was working fine
git bisect good 9e1d4f2
```

Git checks out the commit halfway between good and bad. You test your code and tell Git the result:

```bash
# If this commit is broken:
git bisect bad

# If this commit is fine:
git bisect good
```

Git keeps halving the range until it narrows down to the exact commit that introduced the problem. With 200 commits, it takes at most 8 steps.

```
Bisecting: 4 revisions left to test after this (roughly 2 steps)
[a4b5c6d] Add feature engineering for time variables
```

When done:

```bash
# Reset back to normal
git bisect reset
```

You can even automate it by giving `git bisect` a test script — it will run the script at each step and determine good/bad automatically. But even used manually, it is a powerful debugging tool.

---

## Summary

Here is a quick reference of the commands covered in this module:

```bash
# Setup
git config --global user.name "Name"
git config --global user.email "email"

# Starting a repository
git init
git status

# Staging and committing
git add <file>
git add .
git add -p
git commit -m "message"

# Viewing history
git log --oneline --graph --all
git diff
git diff --staged
git show <hash>

# Ignoring files
# (edit .gitignore)
git check-ignore -v <file>

# Undoing changes
git restore <file>
git restore --staged <file>
git commit --amend
git revert <hash>
git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1

# Branching
git branch
git switch -c <name>
git branch -d <name>

# Merging
git merge <branch>
git merge --abort

# Stashing
git stash
git stash list
git stash pop
git stash drop stash@{n}

# Rebase
git rebase <branch>
git rebase -i HEAD~n
git rebase --abort
git rebase --continue

# Tags
git tag -a v1.0 -m "message"
git tag
git show v1.0

# Everyday utilities
git blame <file>
git grep "term"
git reflog
git bisect start
```

You now have everything you need for solid, professional local version control. In the next module we will connect your local repository to a remote (GitHub) and learn how to collaborate with others.
