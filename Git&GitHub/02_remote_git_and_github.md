# Part 2: Remote Version Control with GitHub

Welcome back! In Part 1 you built a solid foundation with local Git — initializing repositories, staging changes, committing, branching, and merging. Now it's time to take your work online.

In this module you will learn how to connect your local repository to GitHub, collaborate with teammates, contribute to open-source projects, and handle the real-world complexities that come with multiple people working on the same codebase. By the end you will feel confident with the full remote workflow that professional data science teams use every day.

Let's go!

---

## Table of Contents

1. [Introduction to GitHub](#1-introduction-to-github)
2. [Connecting to GitHub via SSH](#2-connecting-to-github-via-ssh)
3. [Creating a Remote Repository](#3-creating-a-remote-repository)
4. [Cloning a Repository](#4-cloning-a-repository)
5. [Fetching, Pulling, and Pushing](#5-fetching-pulling-and-pushing)
6. [Remote Branches and Tracking](#6-remote-branches-and-tracking)
7. [Pull Requests (PRs)](#7-pull-requests-prs)
8. [Forking](#8-forking)
9. [Team Collaboration Workflows](#9-team-collaboration-workflows)
10. [Handling Conflicts on Remote Branches](#10-handling-conflicts-on-remote-branches)
11. [Advanced Remote Operations](#11-advanced-remote-operations)
12. [Quick Reference: Common Remote Workflows](#12-quick-reference-common-remote-workflows)
13. [Wrapping Up](#wrapping-up)

---

## 1. Introduction to GitHub

### What Is GitHub?

GitHub is a **web-based hosting platform for Git repositories**. It gives your local Git history a home on the internet so you can:

- Back up your work in the cloud
- Share projects with colleagues and the public
- Collaborate with others without emailing zip files back and forth
- Review each other's code before it goes into the main codebase
- Track bugs, ideas, and tasks through Issues

> **Note:** GitHub is not Git. Git is the version control tool that runs on your machine. GitHub is a service that hosts Git repositories and adds collaboration features on top. You could use Git without GitHub — but GitHub without Git is just a website.

### Git vs. GitHub — Side by Side

| | Git | GitHub |
|---|---|---|
| **Type** | Command-line tool | Web service |
| **Runs on** | Your local machine | Anthropic's servers (cloud) |
| **Created by** | Linus Torvalds (2005) | Tom Preston-Werner et al. (2008) |
| **Core job** | Track changes in files | Host repos + collaboration features |
| **Cost** | Free, open source | Free tier + paid plans |

### Other Platforms

GitHub is the most popular platform, but it is not the only one. You might encounter:

- **GitLab** — popular in enterprise environments; can be self-hosted
- **Bitbucket** — popular with teams already using Atlassian tools (Jira, Confluence)

All three speak the same underlying Git protocol, so the commands you learn here transfer directly. We focus on GitHub because it hosts the vast majority of open-source data science libraries you will use (pandas, scikit-learn, PyTorch, and so on).

### Key Concepts

**Repository (repo):** A project tracked by Git. On GitHub, each repo lives at a URL like `github.com/username/repo-name`.

**Organization:** A shared account for a team or company. Members can have different access levels (read, write, admin). You will often work inside an organization like `neuefische`.

**Profile:** Your personal GitHub account — your public portfolio. Employers look at these! A green contribution graph and well-documented repos make a great impression.

**Stars:** A way to bookmark or appreciate a repository (like a GitHub "like").

**Fork:** Your personal copy of someone else's repository (we will cover this in detail in Section 8).

### A Quick Tour of the GitHub Interface

When you open any repository on GitHub, you will see several tabs:

| Tab | What you find there |
|---|---|
| **Code** | All files and folders, the README, branch selector, recent commits |
| **Issues** | Bug reports, feature requests, discussion threads |
| **Pull Requests** | Proposed code changes awaiting review |
| **Actions** | Automated workflows (CI/CD pipelines, tests, deployments) |
| **Projects** | Kanban-style project boards |
| **Wiki** | Documentation pages |
| **Settings** | Repo configuration, collaborators, branch protection |

For now, focus on **Code**, **Issues**, and **Pull Requests** — those are the three you will visit every day.

---

## 2. Connecting to GitHub via SSH

### SSH Keys Are Already Configured

We assume your SSH key is already set up and added to your GitHub account. If you are unsure, the quick check below will tell you immediately.

### Quick Check: Test Your SSH Connection

Run this command in your terminal:

```bash
ssh -T git@github.com
```

A successful response looks like this:

```
Hi your-username! You've successfully authenticated, but GitHub does not provide shell access.
```

If you see that message — you are good to go. If you see a permission error, reach out to your instructor to get SSH configured before continuing.

### Platform notes: SSH agent startup

The SSH agent holds your unlocked key in memory so you do not have to enter a passphrase repeatedly. How it starts differs by platform:

**macOS:** The SSH agent starts automatically in the background. To persist your key across reboots, add it to the macOS Keychain:
```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

**Ubuntu / Linux:** The agent does not always start automatically. Start it for your current session:
```bash
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_ed25519
```
To start it automatically on login, add both lines to your `~/.bashrc` or `~/.zshrc`.

**Windows (Git Bash):** Start the agent for your current session:
```bash
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_ed25519
```
To avoid doing this every time, add both lines to `~/.bashrc` in your Git Bash home directory.

**Windows (PowerShell):** Run once as Administrator to enable the agent service:
```powershell
Set-Service -Name ssh-agent -StartupType Automatic
Start-Service ssh-agent
ssh-add $HOME\.ssh\id_ed25519
```

### SSH vs. HTTPS

GitHub supports two ways to authenticate: **SSH** and **HTTPS**.

| | SSH | HTTPS |
|---|---|---|
| **Authentication** | SSH key pair (no password) | Username + Personal Access Token |
| **URL format** | `git@github.com:user/repo.git` | `https://github.com/user/repo.git` |
| **Password prompt?** | Never (once keys are set up) | Every push/pull (unless cached) |
| **Recommended?** | Yes | Only if SSH is unavailable |

**We use SSH** because it is frictionless: once your key is configured, you never type a password again. Every `git push` and `git pull` just works.

### Understanding SSH Remote URLs

An SSH remote URL has this structure:

```
git@github.com:username/repo-name.git
```

Breaking it down:

- `git` — the user on the remote server
- `@github.com` — the server
- `:username/repo-name.git` — the path to the repository

For example, if your GitHub username is `data-anna` and your repo is `customer-churn`, the URL would be:

```
git@github.com:data-anna/customer-churn.git
```

---

## 3. Creating a Remote Repository

### Step 1: Create the Repo on GitHub

1. Go to [github.com](https://github.com) and sign in.
2. Click the **+** button in the top-right corner → **New repository**.
3. Fill in the form:
   - **Repository name:** `customer-churn` (use hyphens, not spaces)
   - **Description:** (optional but recommended — e.g. "Analysis of customer churn for telecom dataset")
   - **Visibility:** Public or Private (see below)
   - **Initialize this repository:** Leave all checkboxes **unchecked** if you already have a local repo. If this is a brand-new project with no local files yet, you can tick "Add a README".
4. Click **Create repository**.

<!-- SCREENSHOT: Creating a new repository on GitHub — the "New repository" form filled out with name, description, and visibility settings before clicking "Create repository" -->

> 📸 **Screenshot:** The GitHub "New repository" page with the repository name, description, visibility selector (Public/Private), and the "Create repository" button visible.

### Public vs. Private

**Public** — anyone on the internet can see the code. Great for:
- Open-source projects
- Portfolio work you want employers to see
- Sharing analysis notebooks with the data science community

**Private** — only you and explicitly added collaborators can see it. Use for:
- Client work
- Proprietary models or datasets
- Work-in-progress you are not ready to share

> **Tip:** Your neuefische bootcamp projects can start private and be made public later. You control this any time in the repo's Settings.

### Connecting Your Existing Local Repo to GitHub

After creating the repo on GitHub, it will show you a page with setup instructions. For an **existing local repo**, use this sequence:

```bash
# Tell Git where the remote is (we name it "origin" by convention)
git remote add origin git@github.com:your-username/customer-churn.git

# Rename your current branch to "main" (if it is still called "master")
git branch -M main

# Push your commits to GitHub, and set "origin/main" as the upstream
git push -u origin main
```

You should see output like:

```
Enumerating objects: 12, done.
Counting objects: 100% (12/12), done.
Delta compression using up to 8 threads
Compressing objects: 100% (9/9), done.
Writing objects: 100% (12/12), 4.23 KiB | 4.23 MiB/s, done.
Total 12 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (2/2), done.
To git@github.com:your-username/customer-churn.git
 * [new branch]      main -> main
Branch 'main' set up to track remote branch 'main' from 'origin'.
```

Refresh GitHub — your files are there!

### The `-u` Flag Explained

The `-u` flag (short for `--set-upstream`) does two things in one go:

1. Pushes your commits to the remote branch.
2. Links your local `main` branch to `origin/main` as its **tracking branch**.

After this, you can just type `git push` or `git pull` without specifying the remote and branch every time — Git already knows where to go.

### Verifying Your Remote Connections

```bash
git remote -v
```

Expected output:

```
origin  git@github.com:your-username/customer-churn.git (fetch)
origin  git@github.com:your-username/customer-churn.git (push)
```

The `(fetch)` and `(push)` entries can technically point to different URLs, but in practice they are almost always the same.

### Managing Remotes

**Rename a remote:**

```bash
git remote rename origin upstream
```

This is commonly used in the fork workflow (see Section 8) where you rename `origin` to `upstream` to clarify it is the original source repo.

**Remove a remote:**

```bash
git remote remove old-remote
```

This only removes the reference in your local config — it does nothing to the actual repository on GitHub.

**Change a remote's URL** (e.g. if you renamed your GitHub repo):

```bash
git remote set-url origin git@github.com:your-username/new-repo-name.git
```

---

## 4. Cloning a Repository

Cloning means downloading a complete copy of a repository — all files, all history, all branches — to your local machine.

### Basic Clone

```bash
git clone git@github.com:username/repo.git
```

This creates a new folder named `repo` in your current directory and downloads everything into it.

### Clone Into a Custom Folder Name

```bash
git clone git@github.com:neuefische/ds-ml-project.git my-ml-project
```

This clones the repo into a folder called `my-ml-project` instead of `ds-ml-project`. Useful when the repo name is long or conflicts with something you already have locally.

### What Cloning Does

When you run `git clone`:

1. Creates a new directory with the repo name (or your custom name).
2. Initializes a Git repo inside it (`.git` folder).
3. Downloads all commits, branches, and tags.
4. Sets up `origin` pointing to the URL you cloned from — automatically, no `git remote add` needed.
5. Checks out the default branch (usually `main`).

In other words, cloning is the "get a copy of someone else's project and start working" command.

### Clone vs. Fork — A Preview

You will hear both terms often. Here is the quick distinction:

| | Clone | Fork |
|---|---|---|
| **What it is** | Local copy of a repo on your machine | Server-side copy of a repo on YOUR GitHub account |
| **Where it lives** | Your laptop | GitHub servers |
| **Write access to original?** | Depends on your permissions | No (unless you open a PR and it gets merged) |
| **Used when** | You have write access to the repo | You want to contribute to a repo you do not own |

We cover forks in full detail in Section 8.

---

## 5. Fetching, Pulling, and Pushing

This section covers the three commands that move data between your local repo and the remote. Understanding the difference between `fetch` and `pull` is one of the most important things in this module.

### git fetch — Download Without Merging

`git fetch` downloads new commits and branches from the remote **but does not touch your working files or your local branches**. It is purely an information-gathering step.

**Fetch everything from origin:**

```bash
git fetch origin
```

Output:

```
remote: Enumerating objects: 7, done.
remote: Counting objects: 100% (7/7), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 4 (delta 2), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (4/4), done.
From github.com:your-username/customer-churn
   a1b2c3d..e4f5g6h  main       -> origin/main
 * [new branch]      feature/visualization -> origin/feature/visualization
```

**Fetch a specific branch:**

```bash
git fetch origin main
```

**See what changed after fetching:**

```bash
git diff main origin/main
```

This shows you exactly what commits are on the remote `main` that you do not have locally yet — without changing anything. You are in full control.

**Why fetch before merging?**

Fetching first lets you inspect incoming changes before they land in your local branch. This is especially useful on a team where you want to check if there are conflicts or breaking changes before integrating them.

```bash
git fetch origin
git log main..origin/main --oneline   # see commits on remote but not local
git diff main origin/main             # see the actual file differences
git merge origin/main                 # merge when you are ready
```

### git pull — Fetch + Merge in One Step

`git pull` is a convenience command that runs `git fetch` followed by `git merge` (or rebase, depending on config).

**Pull the main branch:**

```bash
git pull origin main
```

**Pull the currently tracked branch (after upstream is set):**

```bash
git pull
```

**Pull with rebase instead of merge:**

```bash
git pull --rebase
```

`--rebase` replays your local commits on top of the incoming commits instead of creating a merge commit. The result is a cleaner, linear history. This is generally preferred in teams that care about a readable git log.

> **Tip:** If you want `--rebase` to be the default behavior for every `git pull`, run:
> ```bash
> git config --global pull.rebase true
> ```

**Fast-forward vs. merge commit:**

- **Fast-forward:** Happens when the remote has new commits but you have none locally since you last synced. Git simply moves your branch pointer forward — no merge commit needed. Clean!
- **Merge commit:** Happens when both you and the remote have commits since the last sync. Git creates an extra "Merge branch..." commit to join the two histories.

```
Fast-forward:              With diverging histories:
                          
A---B---C  (local main)   A---B---C  (local main)
         \                         \
          D---E (origin/main)       D---E (origin/main)
                                           \
After pull:                         A---B---C---M  (after merge)
A---B---C---D---E                            \ /
                                              D---E
```

### git push — Send Your Commits to GitHub

**Push to a specific remote and branch:**

```bash
git push origin main
```

**Push the currently tracked branch:**

```bash
git push
```

**Push a new local branch to GitHub and set tracking:**

```bash
git push -u origin feature/add-preprocessing
```

After this, future `git push` and `git pull` on that branch will automatically target `origin/feature/add-preprocessing`.

**Delete a remote branch:**

```bash
git push origin --delete feature/old-experiment
```

> **Warning:** Deleting a remote branch cannot be undone easily. Make sure the work is merged or no longer needed before running this.

### The Typical Daily Workflow

Here is what a normal working day looks like:

```bash
# 1. Start your day: get the latest changes
git fetch origin
git diff main origin/main        # check what came in
git merge origin/main            # or: git pull

# 2. Work: edit files, run notebooks, train models
# ... edit analysis.py, notebook.ipynb, model.py ...

# 3. Stage and commit your changes
git add analysis.py notebook.ipynb
git commit -m "feat: add feature engineering for age groups"

# 4. Push your work
git push
```

Rinse and repeat. The key habit is: **pull before you start, push when you are done**.

---

## 6. Remote Branches and Tracking

### What Are Remote-Tracking Branches?

When you fetch or clone, Git creates **remote-tracking branches** — local references that reflect the last known state of branches on the remote. They look like this:

```
origin/main
origin/feature/visualization
origin/feature/model-tuning
```

These are read-only snapshots. You cannot commit directly to `origin/main`. They update whenever you run `git fetch` or `git pull`.

Think of them as a cached copy of what GitHub looked like the last time you synced.

### Listing Branches

**List remote-tracking branches:**

```bash
git branch -r
```

Output:

```
  origin/HEAD -> origin/main
  origin/feature/visualization
  origin/feature/model-tuning
  origin/main
```

**List all branches (local + remote):**

```bash
git branch -a
```

Output:

```
* main
  feature/preprocessing
  remotes/origin/HEAD -> origin/main
  remotes/origin/feature/visualization
  remotes/origin/feature/model-tuning
  remotes/origin/main
```

The `*` marks your currently checked-out branch.

### Checking Out a Remote Branch Locally

If a teammate pushed a branch and you want to work on it:

```bash
git switch -c feature/visualization origin/feature/visualization
```

This creates a new local branch `feature/visualization` that tracks `origin/feature/visualization`. Now you can work on it, commit, and push.

> **Note:** The older equivalent is `git checkout -b feature/visualization origin/feature/visualization`. Both work; `git switch` is the newer, cleaner command.

### Understanding Upstream Tracking

Every local branch can have an **upstream** (also called a tracking branch) — a remote branch it is linked to. This is what makes `git push` and `git pull` work without specifying the remote and branch name each time.

**See tracking relationships for all branches:**

```bash
git branch -vv
```

Output:

```
* feature/preprocessing  3a4b5c6 [origin/feature/preprocessing: ahead 2] feat: add outlier removal
  main                   1a2b3c4 [origin/main] chore: update requirements.txt
```

Reading the output:
- `[origin/feature/preprocessing: ahead 2]` — your local branch is 2 commits ahead of the remote (you have unpushed work).
- `[origin/main]` — in sync with remote.
- If you see `behind 3`, you have 3 commits to pull in.
- If you see `ahead 2, behind 1`, your histories have diverged.

### HEAD and What It Points To

`HEAD` is a special pointer that indicates which branch you are currently on (and thus which commit you are working from). When you push and pull, Git uses `HEAD` to know which local branch to sync.

```bash
git log --oneline --graph --all
```

Example output:

```
* e4f5g6h (HEAD -> feature/preprocessing, origin/feature/preprocessing) feat: add outlier removal
* d3e4f5g feat: add age group bins
| * c2d3e4f (origin/main, main) docs: update README with dataset description
|/
* b1c2d3e feat: load telecom churn dataset
* a0b1c2d Initial commit
```

This graph shows you the full picture: where your branches are, where the remote branches are, and how they relate.

---

## 7. Pull Requests (PRs)

Pull Requests are the heart of GitHub collaboration. A PR is a **proposal to merge a branch into another branch**, wrapped in a discussion thread where your team can review, comment, and approve (or request changes) before anything lands in `main`.

> **Note:** The name "Pull Request" comes from asking the maintainer to *pull* your changes. GitLab calls them "Merge Requests" which is arguably more descriptive — but they are the same thing.

### The Full PR Workflow

Here is the complete lifecycle of a feature from idea to merged code:

**1. Create a branch locally**

```bash
git switch main
git pull                              # always start from latest main
git switch -c feature/churn-model
```

**2. Make your commits**

```bash
# Work on your analysis
# ... edit model.py, notebook.ipynb ...

git add model.py
git commit -m "feat: train logistic regression baseline"

git add notebook.ipynb
git commit -m "feat: add confusion matrix visualization"
```

**3. Push your branch to GitHub**

```bash
git push -u origin feature/churn-model
```

**4. Open a PR on GitHub**

- Go to your repository on GitHub.
- You will usually see a yellow banner: _"Your recently pushed branch: feature/churn-model — Compare & pull request"_ — click it.
- Alternatively, go to the **Pull Requests** tab → **New pull request** → select your branch.
- Fill in the PR form (see "Writing a Good PR Description" below).
- Click **Create pull request**.

<!-- SCREENSHOT: Creating a new pull request — the "Open a pull request" page on GitHub showing the branch comparison, title field, description editor, and the "Create pull request" button -->

> 📸 **Screenshot:** The GitHub "Open a pull request" page showing the base branch (`main`) and compare branch (`feature/churn-model`), with the PR title, description, and reviewer fields visible.

**5. Team reviews and comments**

Your teammates are notified. They look at the diff, leave comments, ask questions, suggest improvements.

<!-- SCREENSHOT: The PR review/diff view — showing the "Files changed" tab with line-by-line diff, inline comment threads, and the overall review summary panel -->

> 📸 **Screenshot:** The "Files changed" tab of a pull request, showing the diff of `model.py` with green additions and red deletions, and an inline comment thread on a specific line.

<!-- SCREENSHOT: Leaving a comment on a specific line — the comment box appearing on hover over a diff line, with a partially typed review comment -->

> 📸 **Screenshot:** A reviewer clicking the `+` icon on a specific line of the diff to leave an inline comment, showing the comment editor with the line highlighted.

**6. Address feedback with new commits**

You do not need to close and reopen the PR. Just push new commits to the same branch:

```bash
# Address reviewer feedback
git add model.py
git commit -m "refactor: use Pipeline to prevent data leakage"
git push
```

The PR automatically updates with your new commits.

**7. PR is approved**

Once reviewers are happy, they click **Approve**.

<!-- SCREENSHOT: Approving a PR — the review submission dialog with "Approve" selected and a positive review comment written -->

> 📸 **Screenshot:** The GitHub "Review changes" dialog with the "Approve" option selected, a summary comment written, and the "Submit review" button highlighted.

**8. Merge the PR**

The PR author (or a maintainer) clicks **Merge pull request**.

<!-- SCREENSHOT: Merging a PR and deleting the branch — the merge confirmation dialog on GitHub, followed by the "Delete branch" button that appears after successful merge -->

> 📸 **Screenshot:** The GitHub merge confirmation page showing the merge button, a confirmation message after merging, and the "Delete branch" button to clean up the feature branch.

**9. Clean up locally**

```bash
git switch main
git pull                              # get the merge commit
git branch -d feature/churn-model    # delete local branch
```

### Writing a Good PR Description

A good PR description saves your teammates time and makes reviews more productive. Include:

- **What changed:** Briefly describe the feature or fix. ("Adds a logistic regression baseline model for churn prediction.")
- **Why:** What problem does it solve? ("Provides a benchmark we can compare against more complex models.")
- **How to test:** Tell reviewers how to verify the change works. ("Run `python model.py` — you should see AUC of ~0.78 printed to console. Open `notebook.ipynb` to see the confusion matrix.")
- **Screenshots or output:** If you changed a visualization or model output, paste a screenshot or copy-paste the relevant output.
- **Related issues:** Link to the Issue this closes with `Closes #42` — GitHub will automatically close Issue #42 when the PR is merged.

A template to get you started:

```markdown
## What changed
Trained a logistic regression baseline model on the telecom churn dataset.

## Why
We need a benchmark model before experimenting with more complex approaches.

## How to test
1. Run `python model.py` — should print AUC ~0.78
2. Open `notebook.ipynb` — run all cells, check the confusion matrix at the bottom

## Related
Closes #12
```

### PR Review: Your Role as a Reviewer

When reviewing someone else's PR:

- **Read the description first** — understand the intent before diving into code.
- **Look at the big picture** in the "Files changed" tab — are there any files that should not have changed?
- **Leave specific, actionable comments** — not "this is wrong" but "consider using `pd.get_dummies()` here to avoid manual encoding — it handles edge cases better."
- **Distinguish blocking issues from suggestions** — use "nit:" prefix for minor style preferences that do not block the merge.
- **Approve when you are satisfied** — do not leave PRs hanging.

### Merging Strategies on GitHub

When you click "Merge pull request", GitHub gives you three options:

**1. Merge commit (default)**

```
main:    A---B---M
                  \
feature:    C---D--'
```

Creates a merge commit `M` that joins the two histories. The full history of every commit on the feature branch is preserved. Use this when you want the complete audit trail.

```bash
# Equivalent local command
git merge --no-ff feature/churn-model
```

**2. Squash and merge**

```
main:    A---B---E
```

All commits from the feature branch are **squashed into a single commit** `E` before merging. The feature branch history is compressed into one tidy commit on `main`. The PR description becomes the commit message. Great for keeping `main` clean and readable.

```bash
# Equivalent local command
git merge --squash feature/churn-model
git commit -m "feat: add logistic regression churn model (#15)"
```

> **Tip:** Squash and merge is very popular in data science teams. It keeps `git log --oneline` on `main` clean and readable — one entry per feature/fix.

**3. Rebase and merge**

```
main:    A---B---C---D
```

Each commit from the feature branch is replayed onto `main` individually, creating a perfectly linear history. No merge commit. Commits are rewritten (new SHA hashes). Use when you want linear history but want to preserve individual commits.

```bash
# Equivalent local command
git rebase main feature/churn-model
git switch main
git merge --ff-only feature/churn-model
```

> **Warning:** Rebase and merge rewrites commit SHAs. If others have based work on those commits, they will have a bad time. Only use on short-lived feature branches.

**Which should you use?**

Most data science teams default to **Squash and merge** for its clean history, unless the team has a strong preference for full commit logs. Talk to your team and agree on one strategy — consistency matters more than which one you pick.

---

## 8. Forking

### What Is a Fork?

A **fork** is your own personal copy of someone else's repository, stored on **your GitHub account**. It is entirely independent from the original — you can commit freely, create branches, break things, and experiment without affecting the original project.

Forking is the foundation of open-source collaboration. It is how thousands of people contribute to projects like pandas, scikit-learn, and Matplotlib every day.

### When to Fork

- **Contributing to open source:** You do not have write access to the original repo, so you fork, make changes, and open a PR back.
- **Projects you cannot write to directly:** Your instructor's starter repo, a public dataset repo, someone else's tool you want to customize.
- **Experimenting with a project:** You want to try a major refactor without risking the original.

If you have direct write access to a repo (e.g. you are a team member), you do **not** need to fork — just create a branch directly.

### The Fork Workflow

**Step 1: Fork on GitHub**

Click the **Fork** button at the top-right of any repository page. GitHub creates a copy at `github.com/your-username/repo-name`.

<!-- SCREENSHOT: Forking a repository — the fork button at the top of a GitHub repository page, and the fork dialog showing the owner and repository name before confirming -->

> 📸 **Screenshot:** The top-right corner of a GitHub repository page showing the "Fork" button with a fork count, and the dialog that appears asking where to fork the repo (your personal account or an org).

**Step 2: Clone your fork**

```bash
git clone git@github.com:your-username/original-repo.git
cd original-repo
```

Note: you are cloning **your fork**, not the original. Your fork's URL has your username in it.

**Step 3: Add the original repo as `upstream`**

```bash
git remote add upstream git@github.com:original-owner/original-repo.git
```

Now verify you have two remotes:

```bash
git remote -v
```

```
origin    git@github.com:your-username/original-repo.git (fetch)
origin    git@github.com:your-username/original-repo.git (push)
upstream  git@github.com:original-owner/original-repo.git (fetch)
upstream  git@github.com:original-owner/original-repo.git (push)
```

- **`origin`** = your fork on GitHub (you have write access)
- **`upstream`** = the original repo (read access only, unless your PR is accepted)

**Step 4: Keep your fork in sync with the original**

The original repo keeps getting new commits from other contributors. You need to pull those in regularly:

```bash
git fetch upstream                  # download new commits from original
git switch main
git merge upstream/main             # apply them to your local main
git push origin main                # push updated main to your fork
```

Or in one line:

```bash
git pull upstream main
```

**Step 5: Create a branch, make changes, and open a PR**

```bash
git switch -c fix/correct-column-names
# ... make changes to analysis.py ...
git add analysis.py
git commit -m "fix: rename 'custID' column to 'customer_id' for consistency"
git push -u origin fix/correct-column-names
```

Then go to GitHub and open a PR from `your-username:fix/correct-column-names` **to** `original-owner:main`.

> **Note:** The PR goes from **your fork** to the **original repo** — not to your own `main`. GitHub makes this easy: when you push to your fork and open a PR, it automatically suggests the original repo as the target.

**Full picture of the fork workflow:**

```
Original Repo (upstream)
        ↑ PR
Your Fork (origin)
        ↑ push
Your Local Clone
        → branch, commit, push
```

---

## 9. Team Collaboration Workflows

### The Feature Branch Workflow

The **Feature Branch Workflow** is the most widely used collaboration pattern in professional data science teams. The rules are simple:

1. `main` is **always stable and deployable** — no one commits directly to it.
2. Every piece of work (feature, fix, experiment) lives on its own branch.
3. Work flows: branch → commit → PR → review → merge.
4. Short-lived branches are better than long-lived ones.

This keeps the main branch clean and ensures that every change gets a second pair of eyes before going in.

### Full Example Walkthrough with Two Developers

**Setup:** Anna and Ben are collaborating on a churn prediction project.

**Anna starts a new feature:**

```bash
# Anna: make sure her main is up to date
git switch main
git pull origin main

# Create a branch for her feature
git switch -c feature/eda-visualizations

# Work...
git add notebook.ipynb
git commit -m "feat: add distribution plots for numerical features"
git add notebook.ipynb
git commit -m "feat: add correlation heatmap"

# Push and open a PR
git push -u origin feature/eda-visualizations
# → Anna opens a PR on GitHub titled "Add EDA visualizations"
```

**Ben reviews Anna's PR:**

- Ben opens the PR on GitHub
- Reviews the diff in "Files changed"
- Leaves a comment: "The heatmap looks great! Can you also add a title to the correlation matrix?"
- Requests changes

**Anna addresses the feedback:**

```bash
# Still on her branch
git add notebook.ipynb
git commit -m "style: add title to correlation heatmap (PR feedback)"
git push
```

**Ben approves and the PR is merged.**

**Meanwhile, Ben was working on a different branch:**

```bash
git switch main
git pull                            # Ben had already been working on feature/model-baseline
git switch feature/model-baseline
git merge main                      # bring in Anna's merged EDA work
# Resolve any conflicts if needed
git push
# → Ben opens his PR when ready
```

This pattern scales from 2 people to 50. Every developer works in isolation on their branch, integrating via PRs.

### Protecting the Main Branch

On real projects, you should **protect** the `main` branch so that no one can push to it directly or merge without review.

**To set up branch protection:**

1. Go to your repository → **Settings** → **Branches**.
2. Click **Add branch ruleset** (or **Add rule**).
3. Enter `main` as the branch name pattern.
4. Enable the protections you want:

| Protection | What it does |
|---|---|
| **Require a pull request before merging** | No direct pushes to main |
| **Require approvals** | Set minimum number of reviewers (e.g. 1 or 2) |
| **Require status checks to pass** | Tests must be green before merge |
| **Require branches to be up to date** | Branch must be current with main before merge |
| **Do not allow bypassing** | Even admins must follow the rules |

<!-- SCREENSHOT: GitHub branch protection rules settings page — showing the branch name pattern field set to "main" and the various protection options (require PR, require approvals, require status checks) with checkboxes -->

> 📸 **Screenshot:** The GitHub repository Settings → Branches page, showing a branch protection rule for `main` with "Require a pull request before merging" and "Require approvals: 1" enabled.

### Dealing with an Outdated Branch

You have been working on `feature/model-tuning` for a few days. Meanwhile, three other PRs have been merged into `main`. Your branch is now "behind" and needs to be updated. You have two options:

**Option A: Merge main into your branch**

```bash
git switch feature/model-tuning
git fetch origin
git merge origin/main
# Resolve conflicts if any, then:
git add .
git commit -m "merge: bring in latest main changes"
git push
```

This creates a merge commit on your feature branch. The history shows exactly when you synced. Safe and straightforward.

**Option B: Rebase your branch onto main**

```bash
git switch feature/model-tuning
git fetch origin
git rebase origin/main
# Resolve conflicts if any (per-commit, not all at once)
# After resolving each conflict:
# git add <resolved-file>
# git rebase --continue
# If you need to bail out:
# git rebase --abort

git push --force-with-lease         # required because rebase rewrites history
```

This replays your commits on top of the current `main`, creating a clean linear history. Your branch looks like it was started fresh from today's `main`.

**When to use which:**

| | Merge | Rebase |
|---|---|---|
| **History style** | Non-linear (merge commits) | Linear (clean) |
| **Safety** | Always safe | Rewrites commit SHAs |
| **Force push needed?** | No | Yes |
| **Best for** | Shared branches, quick sync | Your own feature branches before PR |
| **Conflict resolution** | Once | Once per replayed commit |

> **Warning:** Never rebase a branch that other people are also working on. Rewriting history confuses everyone and causes them pain. Only rebase your own, unshared feature branches.

---

## 10. Handling Conflicts on Remote Branches

### Why Remote Conflicts Happen

Remote conflicts are fundamentally the same as local merge conflicts — two people edited the same part of the same file. The only difference is that one set of changes is on the remote (pushed by someone else) and the other is local (your uncommitted or committed work).

The most common scenario: you and a teammate both modified `model.py` on different branches, and now those branches need to be integrated.

### Resolving a Pull Conflict Locally

This is the safest and most flexible approach:

```bash
# Step 1: Fetch the latest remote state (no changes to local files yet)
git fetch origin

# Step 2: Merge the remote main into your local branch
git switch feature/model-tuning
git merge origin/main
```

Git will report a conflict:

```
Auto-merging model.py
CONFLICT (content): Merge conflict in model.py
Automatic merge failed; fix conflicts then commit the result.
```

```bash
# Step 3: Open model.py and resolve the conflict
# Look for the conflict markers:
```

```python
<<<<<<< HEAD
# Your version
model = LogisticRegression(C=0.1, max_iter=200)
=======
# Their version (from origin/main)
model = LogisticRegression(C=1.0, max_iter=500, class_weight='balanced')
>>>>>>> origin/main
```

Edit the file to the correct final version (keeping one, combining both, or writing something new):

```python
# Resolved: use balanced class weights with the tuned C from main
model = LogisticRegression(C=0.1, max_iter=500, class_weight='balanced')
```

```bash
# Step 4: Stage the resolved file and commit
git add model.py
git commit -m "merge: resolve conflict in model.py - keep balanced class weights"

# Step 5: Push your updated branch
git push
```

### Resolving a PR Conflict on GitHub

When a PR has conflicts with the base branch, GitHub shows a warning: **"This branch has conflicts that must be resolved."**

**GitHub's built-in conflict editor:**

Click **Resolve conflicts** to open GitHub's web editor. It shows the conflict markers in the file. Edit them, mark as resolved, and commit.

<!-- SCREENSHOT: GitHub's built-in conflict editor — showing a file with conflict markers (<<<<<<<, =======, >>>>>>>) in the web editor, with the "Mark as resolved" button visible -->

> 📸 **Screenshot:** The GitHub conflict resolution editor showing a file (`model.py`) with conflict markers highlighted, the edit cursor active, and the "Mark as resolved" button in the top-right corner of the editor.

**When to use the web editor vs. resolving locally:**

| Situation | Use... |
|---|---|
| Simple conflict, one file, obvious resolution | GitHub's web editor (quick) |
| Complex conflict, multiple files | Resolve locally (more control) |
| Need to run tests to verify the resolution | Resolve locally |
| Conflict in a Jupyter notebook (`.ipynb`) | Resolve locally (JSON conflicts are messy in web editor) |

> **Tip:** Jupyter notebooks (`.ipynb`) are JSON files under the hood. Conflicts in notebooks are notoriously ugly. When you see one, resolve it locally and consider using tools like `nbstripout` to strip cell output before committing — this reduces merge conflicts dramatically.

### Best Practices to Minimize Conflicts

1. **Communicate:** If you and a teammate are both about to edit `model.py`, coordinate who does what first.
2. **Keep PRs small:** Smaller, focused PRs reduce the surface area for conflicts and are reviewed faster.
3. **Pull often:** Start every session with `git pull`. The longer you go without syncing, the bigger the eventual conflict.
4. **Break files apart:** `analysis.py` that does everything is a conflict magnet. Split into `preprocessing.py`, `features.py`, `model.py`, `evaluation.py` — different people can work on different files without stepping on each other.
5. **Merge into your branch before you push:** Run `git fetch && git merge origin/main` before opening a PR. It is better to resolve conflicts locally than to have the PR blocked.

---

## 11. Advanced Remote Operations

### Force Push — Use with Extreme Caution

A normal `git push` will be rejected if the remote has commits you do not have locally (Git protects you from overwriting someone else's work). A force push overrides this protection and **rewrites the remote history**.

**The safer way to force push:**

```bash
git push --force-with-lease
```

`--force-with-lease` is safer than `--force` because it will **refuse to push** if someone else has pushed to the branch since you last fetched. It protects you from accidentally overwriting a teammate's work.

**When force push is acceptable:**

- You have rebased your own feature branch to clean up commits before a PR.
- You used `git commit --amend` on your feature branch.
- You are the only one working on that branch.

```bash
# Example: clean up commits before opening PR
git rebase -i origin/main           # squash/reword your commits
git push --force-with-lease         # update the remote branch
```

**When force push is never acceptable:**

- On `main`, `dev`, or any shared branch.
- If someone else has cloned or branched from the branch you are rewriting.

> **Warning:** `git push --force` on a shared branch is one of the fastest ways to destroy team trust and lose work. When in doubt, do not force push. Ask a senior developer first.

### Tracking Multiple Remotes

You can have more than two remotes. This is useful when:
- You forked a repo and want to pull from the upstream **and** a colleague's fork.
- You are deploying to multiple environments (GitHub + a private server).

```bash
# Add a colleague's fork as another remote
git remote add colleague git@github.com:colleague-username/customer-churn.git

# Fetch from their fork to see their branches
git fetch colleague

# Check out their experimental branch to test it
git switch -c test/colleague-experiment colleague/feature/new-model
```

```bash
git remote -v
```

```
colleague  git@github.com:colleague-username/customer-churn.git (fetch)
colleague  git@github.com:colleague-username/customer-churn.git (push)
origin     git@github.com:your-username/customer-churn.git (fetch)
origin     git@github.com:your-username/customer-churn.git (push)
upstream   git@github.com:neuefische/customer-churn.git (fetch)
upstream   git@github.com:neuefische/customer-churn.git (push)
```

### git cherry-pick Across Branches

`git cherry-pick` lets you apply a specific commit from one branch onto another, without merging the whole branch. This is covered in the local Git module, but it is worth noting that it works just as well with commits from remote branches:

```bash
# Fetch the latest remote state
git fetch origin

# Apply a specific commit from a remote branch to your current branch
git cherry-pick a1b2c3d             # use the commit SHA from git log
```

Useful when a critical bug fix lives on a feature branch that is not ready to merge, but you need that fix in `main` today.

### GitHub CLI (`gh`)

The GitHub CLI brings GitHub's web interface to your terminal. It is a huge productivity booster once you are comfortable with it.

**Install** (macOS):

```bash
brew install gh
gh auth login        # authenticate once
```

**Key commands:**

```bash
# Clone a repo (same as git clone but you can search by name)
gh repo clone username/repo-name

# Create a PR from the current branch (interactive)
gh pr create

# Create a PR with title and body inline
gh pr create --title "feat: add churn model" --body "Adds logistic regression baseline. Closes #12."

# List open PRs
gh pr list

# Check out a PR by number (creates a local branch automatically)
gh pr checkout 15

# Merge a PR
gh pr merge 15 --squash --delete-branch

# View PR status (checks, reviews)
gh pr status

# View a specific PR in the browser
gh pr view 15 --web
```

**Why `gh` is useful:**

- Create PRs without leaving your terminal.
- Check out any PR in one command (`gh pr checkout` beats the multi-step manual process).
- Automate repetitive GitHub tasks in scripts.
- Great for CI/CD and automation workflows.

---

## 12. Quick Reference: Common Remote Workflows

This section is your cheat sheet. Bookmark it. Come back to it whenever you are mid-task and need the exact command sequence.

---

### 1. Start a New Feature

```bash
# Make sure you are on main and up to date
git switch main
git pull

# Create and switch to a new feature branch
git switch -c feature/your-feature-name

# Do your work...
# ... edit analysis.py, model.py, notebook.ipynb ...

# Stage and commit
git add analysis.py model.py
git commit -m "feat: describe what you did"

# Push and set upstream tracking
git push -u origin feature/your-feature-name

# → Open a Pull Request on GitHub
```

---

### 2. Update Your Branch with Latest Main

Use this when `main` has moved forward while you were working on your feature branch.

**Option A: Merge (safe, preserves history)**

```bash
git switch feature/your-feature-name
git fetch origin
git merge origin/main
# Resolve conflicts if any, then:
git add <resolved-files>
git commit
git push
```

**Option B: Rebase (clean linear history)**

```bash
git switch feature/your-feature-name
git fetch origin
git rebase origin/main
# Resolve conflicts per commit (git add + git rebase --continue)
git push --force-with-lease        # required after rebase
```

---

### 3. Contribute to an Open-Source Project (Fork Workflow)

```bash
# Step 1: Fork the repo on GitHub (click the Fork button)

# Step 2: Clone your fork
git clone git@github.com:your-username/original-repo.git
cd original-repo

# Step 3: Add the original as upstream
git remote add upstream git@github.com:original-owner/original-repo.git

# Step 4: Verify both remotes
git remote -v

# Step 5: Create a branch and do your work
git switch -c fix/describe-your-fix
# ... make changes ...
git add changed-file.py
git commit -m "fix: describe what you fixed"

# Step 6: Push to your fork
git push -u origin fix/describe-your-fix

# Step 7: Open a PR on GitHub from your fork to the original repo

# === Keeping your fork in sync (run periodically) ===
git fetch upstream
git switch main
git merge upstream/main
git push origin main
```

---

### 4. Fix a Bug Urgently on Main (Hotfix)

When something is broken in production and you need to fix it now:

```bash
# Start from main (latest)
git switch main
git pull

# Create a hotfix branch
git switch -c hotfix/fix-data-pipeline-crash

# Make the minimal fix
git add pipeline.py
git commit -m "fix: handle missing values in customer_id column"

# Push and open a PR — get a quick review
git push -u origin hotfix/fix-data-pipeline-crash

# After PR is reviewed and merged:
git switch main
git pull

# Clean up the hotfix branch
git branch -d hotfix/fix-data-pipeline-crash
```

> **Note:** Even in an emergency, open a PR. A 5-minute review is always worth it. "Move fast and break things" sounds good until you break the production data pipeline on a Friday at 5pm.

---

### 5. Delete a Stale Branch Locally and Remotely

After a branch is merged, clean it up:

```bash
# Switch away from the branch you want to delete
git switch main
git pull

# Delete the remote branch
git push origin --delete feature/old-experiment

# Delete the local branch
git branch -d feature/old-experiment

# If the branch was not merged and you still want to force-delete it:
git branch -D feature/abandoned-experiment   # capital D = force

# Clean up stale remote-tracking references (optional but tidy)
git fetch --prune
```

`git fetch --prune` removes local references to remote branches that no longer exist on GitHub. Run this occasionally to keep `git branch -a` output tidy.

---

## Wrapping Up

You have covered the full arc of remote Git and GitHub collaboration:

- Connecting your local repo to GitHub via SSH
- Creating, cloning, fetching, pulling, and pushing
- Understanding remote-tracking branches and upstream tracking
- Opening, reviewing, and merging Pull Requests
- Contributing to projects you do not own via forks
- Running a clean Feature Branch Workflow with branch protection
- Resolving conflicts that happen on remote branches
- Advanced operations: force push, multiple remotes, cherry-pick, GitHub CLI
- A cheat sheet of the five workflows you will use most

The best way to internalize all of this is to **practice on a real project**. Push every exercise, every notebook, every experiment to GitHub. Open PRs even when you are working alone — reviewing your own diffs before merging is a great habit. And when you work in a team, embrace the PR process: the code review is where you will learn the most.

Happy collaborating!

---

*Part of the neuefische Data Science Bootcamp — Command Line & Version Control Module*
