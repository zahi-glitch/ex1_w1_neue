# Part 3: Troubleshooting & Common Errors

Even experienced developers run into Git errors regularly. The good news: almost every error message contains the clue you need to fix it. This guide collects the most common problems you will encounter when cloning, pushing, opening pull requests, managing SSH keys, and authenticating with Personal Access Tokens — with plain-English explanations and step-by-step fixes.

---

## Table of Contents

1. [How to Read a Git Error Message](#1-how-to-read-a-git-error-message)
2. [SSH Key Problems](#2-ssh-key-problems)
3. [HTTPS vs SSH Cloning](#3-https-vs-ssh-cloning)
4. [Personal Access Tokens](#4-personal-access-tokens)
5. [Cloning Errors](#5-cloning-errors)
6. [Push & Pull Errors](#6-push--pull-errors)
7. [Pull Request Problems](#7-pull-request-problems)
8. [Permissions & Access Errors](#8-permissions--access-errors)
9. [Quick Diagnostic Checklist](#9-quick-diagnostic-checklist)
10. [Best Practices Summary & Further Reading](#10-best-practices-summary--further-reading)

---

## 1. How to Read a Git Error Message

Git errors follow a consistent pattern once you know what to look for:

```
ERROR: <what went wrong>
fatal: <more detail>
hint: <what to try next>
```

Always read the **hint** line first — Git often tells you exactly what command to run. If there is no hint, focus on the `fatal:` line and search for that exact phrase online.

> **Tip:** Copy the error message exactly and paste it into a search engine. Git error messages are precise and standardized — someone has almost certainly solved your exact problem before.

---

## 2. SSH Key Problems

SSH key errors are the most common source of confusion for beginners. They almost always fall into one of three categories: the key does not exist, it is not loaded, or it is not registered with GitHub.

### Checking your SSH connection first

Before anything else, run:

```bash
ssh -T git@github.com
```

| Response | Meaning |
|---|---|
| `Hi username! You've successfully authenticated...` | SSH is working — look elsewhere for the problem |
| `Permission denied (publickey)` | SSH key issue — continue reading this section |
| `ssh: connect to host github.com port 22: Connection refused` | Network or firewall issue — see [port 22 blocked](#error-port-22-blocked) |

---

### Error: `Permission denied (publickey)`

**Full message:**
```
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.
```

**What it means:** GitHub received your connection request but could not match it to any registered SSH key.

**Fix — Step 1:** Check whether you have an SSH key at all.

**macOS / Linux / Windows (Git Bash):**
```bash
ls ~/.ssh
```
**Windows (PowerShell):**
```powershell
ls $HOME\.ssh
```

You are looking for a pair of files like `id_ed25519` and `id_ed25519.pub` (or `id_rsa` / `id_rsa.pub`). If the directory is empty or does not exist, you need to generate a key:

```bash
ssh-keygen -t ed25519 -C "your.email@example.com"
```

Press Enter at every prompt to accept the defaults.

**Fix — Step 2:** Check whether your key is loaded in the SSH agent, and start the agent if needed.

```bash
ssh-add -l
```

If you see `The agent has no identities`, the agent may not be running or your key is not loaded. Start the agent and add your key based on your platform:

**macOS:**
```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

**Ubuntu / Linux:**
```bash
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_ed25519
```

**Windows (Git Bash):**
```bash
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_ed25519
```

**Windows (PowerShell — run once as Administrator to enable the service):**
```powershell
Set-Service -Name ssh-agent -StartupType Automatic
Start-Service ssh-agent
ssh-add $HOME\.ssh\id_ed25519
```

**Fix — Step 3:** Check whether your public key is registered on GitHub.

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the entire output. Then on GitHub: **Settings → SSH and GPG keys → New SSH key**. Paste the key and save.

Test again:

```bash
ssh -T git@github.com
```

---

### Error: Port 22 blocked

**Full message:**
```
ssh: connect to host github.com port 22: Connection refused
```

**What it means:** Your network (often a corporate firewall or some university WiFi networks) blocks outbound connections on port 22, which SSH uses by default.

**Fix:** Tell SSH to connect to GitHub over port 443 (HTTPS port), which is almost never blocked. Create or edit `~/.ssh/config`:

```bash
# macOS / Linux / Windows (Git Bash)
nano ~/.ssh/config

# Windows (PowerShell) — open in Notepad
notepad $HOME\.ssh\config
```

Add these lines:

```
Host github.com
    Hostname ssh.github.com
    Port 443
    User git
```

Save and test:

```bash
ssh -T git@github.com
```

---

### Error: Wrong key loaded / multiple GitHub accounts

**Symptom:** `ssh -T git@github.com` shows the wrong username, or you get permission errors on a repo you know you should have access to.

**What it means:** Your machine has multiple SSH keys, and the wrong one is being offered to GitHub.

**Fix:** Specify which key to use for GitHub in `~/.ssh/config`:

```
Host github.com
    IdentityFile ~/.ssh/id_ed25519_personal
    User git
```

If you have two GitHub accounts (personal and work), you need two host aliases:

```
Host github-personal
    Hostname github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal

Host github-work
    Hostname github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work
```

Then clone using the alias instead of the real hostname:

```bash
# Instead of: git clone git@github.com:your-org/repo.git
git clone git@github-work:your-org/repo.git
```

---

### Error: SSH key passphrase asked every time

**Symptom:** You set a passphrase on your SSH key and Git asks for it on every `push` or `pull`.

**Fix:** Add your key to the SSH agent so it is unlocked for the session. The command differs by platform:

**macOS** — persists the passphrase in Keychain across reboots:
```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

**Ubuntu / Linux** — unlocked for the current session only:
```bash
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_ed25519
```
To avoid repeating this on every login, add both lines to your `~/.bashrc` or `~/.zshrc`.

**Windows (Git Bash)** — unlocked for the current session:
```bash
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_ed25519
```
Add both lines to `~/.bashrc` in your Git Bash home directory to run automatically.

**Windows (PowerShell)** — once the ssh-agent service is enabled (see Step 2 above):
```powershell
ssh-add $HOME\.ssh\id_ed25519
```

---

## 3. HTTPS vs SSH Cloning

This is one of the most common points of confusion for beginners. When you click the green **Code** button on GitHub, you are given a choice between two URLs. Picking the wrong one — or mixing them up — causes a whole class of frustrating errors.

### The two URL formats

```
# SSH
git@github.com:username/repo-name.git

# HTTPS
https://github.com/username/repo-name.git
```

They point to the same repository but use completely different authentication methods.

| | SSH | HTTPS |
|---|---|---|
| **Looks like** | `git@github.com:user/repo.git` | `https://github.com/user/repo.git` |
| **Authentication** | SSH key pair | Personal Access Token (or browser login) |
| **Password prompt?** | Never (once set up) | Yes — on every push/pull unless cached |
| **Works behind strict firewalls?** | Sometimes blocked on port 22 | Almost always allowed |
| **Recommended for daily use?** | Yes | Only when SSH is unavailable |

**We use SSH on this course.** If you find yourself being asked for a username and password on every push, you are almost certainly using HTTPS by mistake.

---

### How to tell which one you are using

```bash
git remote -v
```

The output will look like one of these:

```
# SSH (correct)
origin  git@github.com:your-username/repo-name.git (fetch)
origin  git@github.com:your-username/repo-name.git (push)

# HTTPS (will cause credential prompts)
origin  https://github.com/your-username/repo-name.git (fetch)
origin  https://github.com/your-username/repo-name.git (push)
```

---

### Switching from HTTPS to SSH

If you accidentally cloned with HTTPS, you do not need to re-clone. Just update the remote URL:

```bash
git remote set-url origin git@github.com:your-username/repo-name.git

# Verify the change
git remote -v
```

---

### Switching from SSH to HTTPS

If SSH is unavailable (e.g. a locked-down network), switch to HTTPS. You will need a Personal Access Token for authentication — see [§4](#4-personal-access-tokens).

```bash
git remote set-url origin https://github.com/your-username/repo-name.git
```

---

### Error: `Support for password authentication was removed`

**Full message:**
```
remote: Support for password authentication was removed on August 13, 2021.
fatal: Authentication failed for 'https://github.com/...'
```

**What it means:** GitHub no longer accepts your GitHub account password over HTTPS. You must use a Personal Access Token instead — or switch to SSH.

**Fix option 1 (recommended):** Switch to SSH.

```bash
git remote set-url origin git@github.com:your-username/repo-name.git
```

**Fix option 2:** Generate a Personal Access Token and use it as your password. See [§4](#4-personal-access-tokens) for the full walkthrough.

---

### Error: Credentials cached for the wrong account

**Symptom:** Git pushes succeed but the commits appear under the wrong GitHub user, or you keep getting `403 Forbidden` even though you are logged into GitHub in the browser.

**What it means:** Your operating system has cached HTTPS credentials for a different GitHub account.

**Fix — macOS (Keychain):**
```bash
git credential-osxkeychain erase
host=github.com
protocol=https
# Press Enter, then Ctrl+D
```

**Fix — Windows (Git Credential Manager):**

Open **Control Panel → Credential Manager → Windows Credentials**, find the `git:https://github.com` entry, and click **Remove**. The next push will prompt for your credentials and cache them correctly.

Alternatively, from Git Bash or PowerShell:
```bash
git credential reject
host=github.com
protocol=https
# Press Enter, then Ctrl+D
```

**Fix — Linux:**
```bash
# If using the cache helper, clear it by setting a zero timeout
git config --global credential.helper 'cache --timeout=1'
# Then immediately run any git command to trigger expiry
```
Or delete the stored credentials file directly:
```bash
# If using the store helper
rm ~/.git-credentials
```

**Fix — all platforms:** Switch to SSH, which does not use the credential cache at all.

---

## 4. Personal Access Tokens

### What is a Personal Access Token?

A **Personal Access Token (PAT)** is a password substitute for GitHub's HTTPS interface and API. When GitHub removed plain-password authentication in 2021, PATs became the only way to authenticate over HTTPS.

Think of a PAT like a hotel key card: it gives access to specific areas (scopes), can be set to expire, and can be revoked instantly without changing your main account password.

### When you need one

| Situation | Need a PAT? |
|---|---|
| Pushing/pulling over SSH | No — your SSH key handles it |
| Pushing/pulling over HTTPS | Yes |
| Using the GitHub API | Yes |
| Authenticating GitHub CLI (`gh`) | Not directly — `gh auth login` handles it |
| CI/CD pipelines and automation | Yes — use a fine-grained token with minimal scope |

---

### Two types of token

| | Classic Token | Fine-Grained Token |
|---|---|---|
| **Scope** | Broad (all repos or all of a type) | Per-repository, per-permission |
| **Expiry** | Optional | Required (max 1 year) |
| **Best for** | Quick personal use | Automation, CI/CD, sharing with scripts |
| **Created at** | GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic) | Same page → Fine-grained tokens |

For everyday HTTPS use, a classic token with `repo` scope is simplest. For anything automated or shared, use a fine-grained token.

---

### Creating a Personal Access Token

1. Go to **GitHub → Settings** (click your profile picture, bottom of the menu).
2. Scroll to **Developer settings** (bottom of the left sidebar).
3. Click **Personal access tokens → Tokens (classic)**.
4. Click **Generate new token (classic)**.
5. Give it a descriptive name (e.g. `laptop-https-2024`).
6. Set an expiration — 90 days is a sensible default.
7. Under **Scopes**, tick **repo** (gives full access to your repositories).
8. Click **Generate token**.
9. **Copy the token immediately** — GitHub will never show it again.

> **Treat your PAT like a password.** Do not paste it into a chat message, commit it to a file, or share it. If you accidentally expose one, go to GitHub and delete it immediately, then generate a new one.

---

### Using a PAT for HTTPS authentication

When Git prompts for credentials over HTTPS:

```
Username: your-github-username
Password: <paste your PAT here — NOT your GitHub account password>
```

---

### Storing your PAT so Git does not ask every time

**macOS — store in Keychain:**
```bash
git config --global credential.helper osxkeychain
```
The first time you push, enter your username and PAT. macOS stores it in the Keychain and Git will not ask again.

**Windows — Git Credential Manager (installed automatically with Git for Windows):**
```bash
git config --global credential.helper manager
```
A browser window or dialog will appear the first time you push. Sign in to GitHub and the token is stored securely in the Windows Credential Manager.

**Ubuntu / Linux — cache in memory for a session:**
```bash
# Cache for 1 hour (3600 seconds)
git config --global credential.helper 'cache --timeout=3600'
```

**Ubuntu / Linux — store permanently using libsecret (GNOME Keyring):**
```bash
sudo apt install libsecret-1-0 libsecret-1-dev
sudo make -C /usr/share/doc/git/contrib/credential/libsecret
git config --global credential.helper /usr/share/doc/git/contrib/credential/libsecret/git-credential-libsecret
```

**All platforms — store in a plain file (least secure):**
```bash
git config --global credential.helper store
```
This writes your token in plain text to `~/.git-credentials`. Only use this on a personal machine with full-disk encryption.

---

### Error: PAT rejected / `Authentication failed`

**Common causes:**

| Cause | Fix |
|---|---|
| Token has expired | Generate a new token and update your stored credentials |
| Token lacks the required scope | Edit the token on GitHub and add the `repo` scope |
| You pasted the token with extra whitespace | Re-copy carefully; some terminals add a newline |
| Old password cached in Keychain | Clear the cached credential and re-enter (see below) |

**Clearing a cached PAT — macOS:**
```bash
git credential-osxkeychain erase
host=github.com
protocol=https
# Press Enter, then Ctrl+D
```

**Clearing a cached PAT — Windows:**

Open **Control Panel → Credential Manager → Windows Credentials**, find `git:https://github.com`, and click **Remove**. The next push will prompt for your new token.

**Clearing a cached PAT — Linux:**
```bash
# cache helper: just wait for the timeout, or set it to 1 second
git config --global credential.helper 'cache --timeout=1'
# store helper: remove the file
rm ~/.git-credentials
```

The next push will prompt for credentials and cache the new token.

---

### Revoking a compromised token

If you accidentally commit a PAT or share it somewhere public:

1. Go to **GitHub → Settings → Developer settings → Personal access tokens**.
2. Find the token and click **Delete**.
3. Generate a new token.
4. Update your local credential cache.

GitHub also automatically scans public repositories for leaked tokens and will revoke them on your behalf — but do not rely on this. Revoke it yourself immediately.

---

## 5. Cloning Errors

### Error: `Repository not found`

**Full message:**
```
ERROR: Repository not found.
fatal: Could not read from remote repository.
```

**What it means:** One of three things: the URL is wrong, the repository is private and your SSH key does not have access, or the repository has been deleted or renamed.

**Fix checklist:**

1. Double-check the URL — go to the repo on GitHub and click the green **Code** button to copy the exact SSH URL.
2. Confirm your SSH key is working: `ssh -T git@github.com`
3. Confirm you have been added as a collaborator (for private repos).
4. Check whether the repo was renamed — GitHub will redirect the browser but Git will not follow redirects.

---

### Error: Cloning into a non-empty directory

**Full message:**
```
fatal: destination path 'repo-name' already exists and is not an empty directory.
```

**What it means:** A folder with that name already exists where you are trying to clone.

**Fix options:**

```bash
# Option 1: Clone into a differently-named folder
git clone git@github.com:user/repo.git my-new-folder-name

# Option 2: Delete the existing folder first (only if you are sure it has nothing you need)
rm -rf repo-name
git clone git@github.com:user/repo.git

# Option 3: If the folder already contains the repo, just pull instead
cd repo-name
git pull
```

---

### Error: `RPC failed` / connection timeout during clone

**Full message:**
```
error: RPC failed; curl 18 transfer closed with outstanding read data remaining
fatal: the remote end hung up unexpectedly
```

**What it means:** The connection dropped mid-transfer, usually because the repository is large or your network is unstable.

**Fix:**

```bash
# Increase the Git HTTP buffer size
git config --global http.postBuffer 524288000

# Try cloning with a shallow clone first (downloads only the latest snapshot)
git clone --depth 1 git@github.com:user/repo.git

# Then fetch the full history afterwards if needed
cd repo
git fetch --unshallow
```

---

## 6. Push & Pull Errors

### Error: `rejected — non-fast-forward`

**Full message:**
```
! [rejected]        main -> main (non-fast-forward)
error: failed to push some refs to 'git@github.com:user/repo.git'
hint: Updates were rejected because the tip of your current branch is behind its remote counterpart.
```

**What it means:** Someone else pushed new commits to the remote branch while you were working. Your local branch is now behind and Git refuses to overwrite their work.

**Analogy:** Imagine two people editing the same Google Doc offline. When you reconnect, you cannot just overwrite the other person's edits — you need to merge them first.

**Fix:**

```bash
# Pull the remote changes first, then push
git pull --rebase origin main
# Resolve any conflicts if prompted, then:
git push origin main
```

> **Never** use `git push --force` on a shared branch like `main`. It overwrites other people's commits and is very difficult to recover from. Force push is only safe on your own personal feature branches that no one else is using.

---

### Error: `Updates were rejected because the remote contains work you do not have locally`

This is the same as the non-fast-forward error above. Follow the same fix.

---

### Error: `Your branch is ahead of 'origin/main' by N commits`

**What it means:** You have local commits that have not been pushed yet. This is not an error — it is just Git telling you to push.

**Fix:**

```bash
git push origin main
```

---

### Error: `Your branch is behind 'origin/main' by N commits`

**What it means:** The remote has new commits that you have not pulled yet.

**Fix:**

```bash
git pull origin main
```

---

### Error: `fatal: 'origin' does not appear to be a git repository`

**Full message:**
```
fatal: 'origin' does not appear to be a git repository
fatal: Could not read from remote repository.
```

**What it means:** Your local repo has no remote called `origin` configured. This happens when you initialize a repo locally with `git init` but forget to link it to GitHub.

**Fix:**

```bash
# Check what remotes exist (may be empty)
git remote -v

# Add the remote
git remote add origin git@github.com:your-username/your-repo.git

# Verify
git remote -v

# Push for the first time
git push -u origin main
```

---

### Error: Merge conflict during pull

**Symptom:**
```
CONFLICT (content): Merge conflict in filename.py
Automatic merge failed; fix conflicts and then commit the result.
```

**What it means:** The same lines in the same file were changed both locally and on the remote. Git cannot decide which version to keep and needs you to decide.

**Fix:**

1. Open the conflicted file. Git marks the conflict like this:

```
<<<<<<< HEAD
your local version of the line
=======
the remote version of the line
>>>>>>> origin/main
```

2. Edit the file to keep what you want (delete the marker lines too).
3. Stage and commit:

```bash
git add filename.py
git commit -m "Resolve merge conflict in filename.py"
git push
```

If you want to abandon the merge entirely:

```bash
git merge --abort
```

---

## 7. Pull Request Problems

### PR shows unexpected commits / too many files changed

**What it means:** Your feature branch accidentally includes commits from `main` or another branch, or was branched off from the wrong base.

**Fix — check where your branch diverged:**

```bash
git log --oneline main..your-branch
```

This shows only the commits on your branch that are not on `main`. If you see commits you did not make, your branch has diverged from the wrong point.

**Fix — rebase your branch onto main:**

```bash
git fetch origin
git rebase origin/main
git push --force-with-lease origin your-branch
```

> `--force-with-lease` is a safer alternative to `--force` on your own feature branches — it will refuse to push if someone else has pushed to the branch since you last fetched.

---

### PR merge button is greyed out / merge blocked

**Common causes and fixes:**

| Cause | How to check | Fix |
|---|---|---|
| Required reviews not met | PR page shows "X review required" | Ask a teammate to review and approve |
| CI checks failing | Red ✗ next to commits on the PR page | Fix the failing tests or lint errors |
| Branch protection rule | Repo Settings → Branches | Follow the required process (e.g. get a review) |
| Merge conflicts | GitHub shows "This branch has conflicts" | See merge conflict fix above |

---

### PR shows changes you did not intend to include

**What it means:** You committed something accidentally (e.g. a large data file, a `.env` file, or a generated output).

**Fix — if not yet merged:**

```bash
# Remove the file from the last commit (but keep it on disk)
git rm --cached path/to/file
git commit --amend --no-edit
git push --force-with-lease origin your-branch
```

**Prevention:** Add the file pattern to `.gitignore` before committing:

```
# In .gitignore
*.csv
.env
__pycache__/
.ipynb_checkpoints/
```

---

### Cannot merge PR — "base branch was modified"

**What it means:** The target branch (`main`) has moved forward since you opened the PR. The PR needs to be updated.

**Fix:**

```bash
git fetch origin
git rebase origin/main
git push --force-with-lease origin your-branch
```

GitHub will automatically update the PR once you push.

---

## 8. Permissions & Access Errors

### Error: `403 Forbidden` or `not authorized`

**Full message:**
```
remote: Permission to user/repo.git denied to your-username.
fatal: unable to access 'https://github.com/...': The requested URL returned error: 403
```

**What it means:** Your account does not have write access to this repository.

**Fix checklist:**

1. Confirm you are trying to push to the correct repo (not someone else's original when you intended to push to your fork).
2. Ask the repo owner to add you as a collaborator: **Repo → Settings → Collaborators → Add people**.
3. If this is an organization repo, ask an admin to grant your team the correct role (Read / Write / Admin).
4. If you are using HTTPS instead of SSH, your credentials may be cached for a different account — switch to SSH.

---

### Error: `403` when pushing to an organization repo

**What it means:** Organization repos often have stricter access controls. You may have Read access but not Write.

**Fix:** An organization admin must go to **Org Settings → Member privileges** or grant your team Write access to the specific repository.

---

### Error: Cannot push directly to `main` (branch protection)

**Full message:**
```
remote: error: GH006: Protected branch update failed for refs/heads/main.
```

**What it means:** The repository has branch protection rules that prevent direct pushes to `main`. This is intentional — it means all changes must go through a Pull Request.

**Fix:** This is not an error to "fix" — it is a deliberate policy. Create a feature branch, push that, and open a PR:

```bash
git switch -c my-feature-branch
# ... make your changes ...
git push origin my-feature-branch
# Then open a PR on GitHub
```

---

### Error: Cannot delete a remote branch

**Full message:**
```
error: unable to delete 'branch-name': remote ref does not exist
```

Or the delete is blocked:

```
remote: error: GH006: Protected branch update failed for refs/heads/main.
```

**Fix:**

```bash
# If the branch simply does not exist on the remote anymore (already deleted)
git fetch --prune

# If it is a protected branch, an admin must disable branch protection first,
# or delete it via the GitHub interface (Settings → Branches → delete).
```

---

## 9. Quick Diagnostic Checklist

When something goes wrong, run through these checks in order:

```bash
# 1. What state is my local repo in?
git status

# 2. Is my SSH key working?
ssh -T git@github.com

# 3. What remote am I pointed at?
git remote -v

# 4. Is my branch up to date with the remote?
git fetch origin
git log --oneline origin/main..HEAD   # commits I have that remote doesn't
git log --oneline HEAD..origin/main   # commits remote has that I don't

# 5. Am I on the right branch?
git branch
```

| Symptom | Most likely cause | Section to read |
|---|---|---|
| `Permission denied (publickey)` | SSH key not set up or not loaded | [§2](#2-ssh-key-problems) |
| Asked for username and password on every push | Using HTTPS instead of SSH | [§3](#3-https-vs-ssh-cloning) |
| `Support for password authentication was removed` | GitHub no longer accepts account passwords over HTTPS | [§3](#3-https-vs-ssh-cloning) |
| `Authentication failed` with HTTPS | Expired or wrong-scope PAT | [§4](#4-personal-access-tokens) |
| `Repository not found` | Wrong URL or no access to private repo | [§5](#5-cloning-errors) |
| `rejected — non-fast-forward` | Remote has new commits you haven't pulled | [§6](#6-push--pull-errors) |
| `CONFLICT` during pull or merge | Same lines edited in both versions | [§6](#6-push--pull-errors) |
| PR merge button greyed out | Missing review, failing CI, or conflicts | [§7](#7-pull-request-problems) |
| `403 Forbidden` | No write access to the repository | [§8](#8-permissions--access-errors) |
| Protected branch push rejected | Branch protection rules in place | [§8](#8-permissions--access-errors) |

---

## 10. Best Practices Summary & Further Reading

### Best practices cross-reference

| Practice | Why it matters | Where covered |
|---|---|---|
| Use SSH, not HTTPS | No credential prompts; no token management for day-to-day work | [§3](#3-https-vs-ssh-cloning) |
| Use `ed25519` keys, not `rsa` | More secure and shorter than RSA; GitHub recommends it | [§2](#2-ssh-key-problems) |
| Never `git push --force` on shared branches | Overwrites teammates' commits with no recovery path | [§6](#6-push--pull-errors) |
| Use `--force-with-lease` instead of `--force` on feature branches | Refuses to push if someone else has pushed since your last fetch | [§7](#7-pull-request-problems) |
| Set an expiry on every PAT | Limits the damage window if a token is leaked | [§4](#4-personal-access-tokens) |
| Use fine-grained tokens for automation | Restricts token to only the repos and permissions it needs | [§4](#4-personal-access-tokens) |
| Revoke a compromised token immediately | GitHub auto-scans public repos but do not wait for it | [§4](#4-personal-access-tokens) |
| Add secrets to `.gitignore` before first commit | A secret committed to git history is exposed even if later deleted | [§7](#7-pull-request-problems) |
| Pull before you push (`git pull --rebase`) | Keeps history linear and avoids unnecessary merge commits | [§6](#6-push--pull-errors) |

---

### Official documentation

The sources below are authoritative and kept up to date by GitHub and the Git project.

**Git**
- [Git Reference Manual](https://git-scm.com/docs) — complete documentation for every Git command
- [Pro Git Book](https://git-scm.com/book/en/v2) — free, comprehensive book; Chapters 2–3 cover the daily workflow, Chapter 6 covers GitHub
- [Git Cheat Sheet (GitHub)](https://training.github.com/downloads/github-git-cheat-sheet/) — printable one-pager of the most common commands

**SSH**
- [GitHub Docs: Connecting to GitHub with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh) — official step-by-step for key generation, adding to the agent, and registering with GitHub
- [GitHub Docs: Testing your SSH connection](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/testing-your-ssh-connection)
- [GitHub Docs: Using SSH over the HTTPS port](https://docs.github.com/en/authentication/troubleshooting-ssh/using-ssh-over-the-https-port) — the fix for port 22 being blocked

**HTTPS & Personal Access Tokens**
- [GitHub Docs: Creating a personal access token (classic)](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic)
- [GitHub Docs: Fine-grained personal access tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token)
- [GitHub Docs: Caching your credentials](https://docs.github.com/en/get-started/getting-started-with-git/caching-your-github-credentials-in-git)
- [GitHub Docs: Token scopes](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/scopes-for-oauth-apps)

**Pull Requests & Collaboration**
- [GitHub Docs: About pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)
- [GitHub Docs: About protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
- [GitHub Docs: Resolving merge conflicts](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/resolving-a-merge-conflict-using-the-command-line)

**Troubleshooting**
- [GitHub Docs: Troubleshooting SSH](https://docs.github.com/en/authentication/troubleshooting-ssh) — covers every SSH error GitHub knows about
- [GitHub Docs: Troubleshooting cloning errors](https://docs.github.com/en/repositories/creating-and-managing-repositories/troubleshooting-cloning-errors)
- [Oh Shit, Git!](https://ohshitgit.com) — plain-English fixes for common Git mistakes; useful quick reference

---

*Part of the neuefische Data Science Bootcamp — Command Line & Version Control Module*
