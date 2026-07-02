# Git & GitHub — Complete Reference (DevOps Edition)

A single revision file covering every command and concept from my Git/GitHub study notes. Built for quick lookup, not first-time learning.

---

## Table of Contents

1. [Git Basics](#1-git-basics)
2. [Git Workflow (3-Tree Model)](#2-git-workflow-3-tree-model)
3. [Commits](#3-commits)
4. [Snapshot vs Delta](#4-snapshot-vs-delta)
5. [Git Log](#5-git-log)
6. [Git Diff](#6-git-diff)
7. [Untracking Files](#7-untracking-files)
8. [Branching](#8-branching)
9. [Remote-Tracking Branches](#9-remote-tracking-branches)
10. [Merging](#10-merging)
11. [Cherry Pick](#11-cherry-pick)
12. [Reset](#12-reset)
13. [Revert](#13-revert)
14. [Reset vs Revert](#14-reset-vs-revert)
15. [Stash](#15-stash)
16. [.gitignore](#16-gitignore)
17. [Remote Repository](#17-remote-repository)
18. [Clone](#18-clone)
19. [Push](#19-push)
20. [GitHub Repo Setup (Quickstart)](#20-github-repo-setup-quickstart)
21. [Fetch](#21-fetch)
22. [Pull](#22-pull)
23. [Fetch vs Pull](#23-fetch-vs-pull)
24. [Fetch --all](#24-fetch---all)
25. [Prune](#25-prune)
26. [Delete Branches](#26-delete-branches)
27. [Fork](#27-fork)
28. [Fork vs Clone](#28-fork-vs-clone)
29. [Rebase](#29-rebase)
30. [Daily Industry Workflow](#30-daily-industry-workflow)
31. [Golden Interview Answers](#31-golden-interview-answers)
32. [Final Mental Model](#32-final-mental-model)

---

## 1. Git Basics

**Initialize a repository**
```bash
git init
git init -b main       # initialize and name the default branch "main"
```
Creates a hidden `.git` folder — this is the actual database that stores every commit, branch pointer, and config. Deleting `.git` deletes all Git history (the files themselves stay).

**Check files**
```bash
ls -l     # detailed view
ls -a     # show hidden files, including .git
```

**Status**
```bash
git status
git status -s     # short form
```
Every file is in one of these states: **Untracked → Modified → Staged → Committed**

Short status codes:
| Code | Meaning |
|---|---|
| `??` | Untracked |
| `M`  | Modified |
| `A`  | Added (staged) |

---

## 2. Git Workflow (3-Tree Model)

```
Working Directory  →  git add  →  Staging Area  →  git commit  →  Local Repository  →  git push  →  Remote Repository
```

**Stage files**
```bash
git add file.txt
git add .          # stage everything
```

---

## 3. Commits

```bash
git commit -m "message"
```

A commit is a snapshot object containing:
- **Author** (who)
- **Timestamp** (when)
- **Commit message**
- **Snapshot** of the project at that point
- **Parent commit** pointer (link to previous commit)
- **SHA-1 hash** (a hash *function* output — it's an identifier/fingerprint, not encryption; it can't be reversed to get the content back, but the same content always produces the same hash)

**Empty commit** (no file changes)
```bash
git commit --allow-empty -m "trigger pipeline"
```
Common uses:
- Manually trigger a CI/CD pipeline (Jenkins, GitHub Actions) with no real code change
- Test that a pipeline/webhook fires correctly

**Signing commits — `-s` vs `-S`**
| Flag | Meaning |
|---|---|
| `-s` | Adds a `Signed-off-by` line (DCO — a statement of authorship, plain text) |
| `-S` | Cryptographically signs the commit with GPG/SSH so GitHub shows "Verified" |

---

## 4. Snapshot vs Delta

**Snapshot** — Git stores the full state of every tracked file at each commit (not a diff chain like some older VCS tools).
```
Commit 1: f1 = "hello"
Commit 2: f1 = "hello world"
```
Git internally stores each version of `f1` as a complete blob, and reuses unchanged blobs across commits for efficiency.

**Delta (conceptual)** — the *difference* between two snapshots, which is what tools like `git diff` compute and display for humans:
```diff
- hello
+ hello world
```

---

## 5. Git Log

```bash
git log
git log --oneline               # compact, one line per commit
git log -n 2                    # last 2 commits
git log -p                      # show the actual diff per commit
git log --since="1 week ago"
git log --until="2025-01-01"
git log --author="name"
git log --grep="fix"            # search commit messages
git log --graph --oneline --all # visual branch/merge graph, all branches
```

---

## 6. Git Diff

| Command | Compares |
|---|---|
| `git diff` | Working Directory vs Staging Area (unstaged changes) |
| `git diff --staged` | Staging Area vs Last Commit (what will be committed) |
| `git diff commit1 commit2` | Two commits |
| `git diff main feature` | Two branches |

---

## 7. Untracking Files

**Stop tracking, keep the file on disk**
```bash
git rm --cached file.txt
git rm --cached .env       # very common — stop tracking a secrets file
```
Note: this alone doesn't stop future changes from showing up unless the file is also added to `.gitignore`.

---

## 8. Branching

```bash
git branch feature1          # create
git checkout feature1        # switch (older syntax)
git switch feature1          # switch (modern syntax)
git checkout -b feature1     # create + switch in one step
git switch -c feature1       # same, modern syntax
git branch                   # list local branches
```

**Concept**
```
main
C1 ---- C2 ---- C3
                  ↑
                HEAD
```
A **branch** is just a movable pointer to a commit. **HEAD** is a pointer to whichever branch (or commit) you currently have checked out.

---

## 9. Remote-Tracking Branches

This is the piece that ties branching, fetch, and pull together — worth treating as its own concept.

**Local branch vs remote-tracking branch**
```bash
git branch      # local branches only        → main, feature1
git branch -r   # remote-tracking branches    → origin/main, origin/feature1
git branch -a   # both, in one list
```

`origin/main` is **not** your working branch — it's a read-only bookmark that records *where the remote's `main` was, as of your last fetch*. It only moves when you run `git fetch` or `git pull`; it never moves just because someone pushed to GitHub.

```
git log   →  shows both your local branch position AND origin/main's last-known position,
             which is how you can tell if you're ahead, behind, or diverged.
```

If you check `git log` right after cloning and again a week later without fetching, `origin/main` in your log will still show the old position — it's frozen until you fetch.

---

## 10. Merging

```bash
git checkout main
git merge feature1
```

**Fast-forward merge** — happens when `main` hasn't moved since the branch was created. Git just slides the `main` pointer forward. No merge commit is created.
```
Before:
main:     C1 --- C2
feature:  C1 --- C2 --- F1

After:
main:     C1 --- C2 --- F1
```

**Three-way merge** — happens when both branches have new commits. Git creates a new **merge commit** with two parents.
```
       F1
      /
C1 -- C2
      \
       M1
```

---

## 11. Cherry Pick

Apply one specific commit from another branch onto your current branch, without merging the whole branch.
```bash
git cherry-pick commitID
```
**Use case:** Feature 1 is finished and ready to ship. Feature 2 is on the same branch but still incomplete. Cherry-pick just Feature 1's commit onto `main` to deploy it, without dragging in Feature 2's unfinished work.

---

## 12. Reset

| Type | Command | HEAD moves? | Staging area | Working directory |
|---|---|---|---|---|
| **Soft** | `git reset --soft commitID` | Yes | Changes stay staged | Unchanged |
| **Mixed** (default) | `git reset --mixed commitID` / `git reset commitID` | Yes | Changes become unstaged | Unchanged |
| **Hard** | `git reset --hard commitID` | Yes | Cleared | ⚠️ Changes deleted — dangerous, not recoverable via normal means |

---

## 13. Revert

```bash
git revert commitID
```
Creates a **new commit** that undoes the changes of a previous commit. History is preserved — safe to use on shared/public branches.

---

## 14. Reset vs Revert

| | Reset | Revert |
|---|---|---|
| History | Rewritten | Preserved |
| Safe on shared branches? | No | Yes |
| Result | HEAD moves backward | New commit added on top |

```
Reset:   C1 --- C2 --- C3
                 ↑
                HEAD

Revert:  C1 --- C2 --- C3 --- C4 (revert of C3)
```

---

## 15. Stash

Temporarily shelve uncommitted changes without making a commit — useful when you need to switch branches for something urgent but aren't ready to commit.

**The real-world trigger:** if you've modified a tracked file and try to `git checkout`/`git switch` to another branch, Git will **block the switch** if that change would conflict with the target branch. You have exactly two ways out — commit the change, or stash it.

```bash
git stash                          # save
git stash push -m "login feature"  # save with a label
git stash -u                       # also stash untracked (new) files
git stash list                     # see all stashed sets
git stash apply                    # re-apply, but keep it in the stash list
git stash pop                      # re-apply AND remove from the stash list
git stash drop stash@{0}           # delete one specific stash
git stash clear                    # delete all stashes
```

Typical flow:
```bash
# working on branch A, uncommitted changes, urgent bug on branch B
git stash push -m "wip: login form"
git switch B
# ...fix the bug, commit, push...
git switch A
git stash list
git stash pop        # your changes are back
```

---

## 16. .gitignore

```gitignore
node_modules/
.env
dist/
*.log
```
Commonly ignored: dependency folders (`node_modules`), secrets (`.env`), build output (`dist/`, `build/`), log files.

⚠️ **Important:** `.gitignore` only prevents *new* untracked files from being picked up. It does **nothing** for files Git is already tracking.
```bash
git rm --cached .env     # fix: untrack it first, then .gitignore keeps it out going forward
```

---

## 17. Remote Repository

```bash
git remote add origin URL
git remote -v              # view configured remotes (fetch + push URLs)
```

---

## 18. Clone

```bash
git clone URL
```
Downloads the full repository (all history, all branches) and automatically configures `origin` as the remote.

---

## 19. Push

```bash
git push -u origin main    # first push — sets up upstream tracking
git push                   # every push after that, no flags needed
```

---

## 20. GitHub Repo Setup (Quickstart)

GitHub shows two different snippets when you create a new empty repo — pick based on whether you already have local history.

**A. Brand-new project (no `.git` folder yet)**
```bash
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/jishad10/kubestarter90.git
git push -u origin main
```

| Command | Purpose |
|---|---|
| `git init` | Initializes local repo |
| `git add README.md` | Stages the README |
| `git commit -m "first commit"` | First commit |
| `git branch -M main` | Force-renames current branch to `main` |
| `git remote add origin <url>` | Links local repo to GitHub |
| `git push -u origin main` | Pushes + sets upstream tracking |

**B. Existing local repo, new empty GitHub repo**
```bash
git remote add origin https://github.com/jishad10/kubestarter90.git
git branch -M main
git push -u origin main
```
No `init`/`add`/`commit` — history already exists. Just wire up the remote and push.

| | A: Create New | B: Push Existing |
|---|---|---|
| Local repo exists? | No | Yes, already has commits |
| Steps | init → add → commit → branch → remote → push | remote → branch → push |
| When | Starting from zero | Already coded locally, GitHub repo just created |

**`-M` vs `-m` on `git branch`**
- `-M` → **force** rename (overwrites `main` if it already exists)
- `-m` → rename only if there's no naming conflict

(This exists because old Git defaulted to `master` as the initial branch name — `-M main` standardizes it before the first push.)

---

## 21. Fetch

```bash
git fetch                    # updates remote-tracking branches for all remotes
git fetch origin main        # safer/explicit — only fetches main from origin
```

**What actually happens:** `fetch` downloads new commits from the remote and updates the **remote-tracking branch** (e.g. `origin/main`). It does **not** touch your local `main` branch and does **not** merge anything — your working directory is untouched. This is what makes fetch "safe": you can inspect what changed before deciding to merge.

**Conflict-avoidance habit:**
```bash
git fetch origin main
git log main..origin/main -p     # or: git diff main origin/main
# review what's new, THEN decide:
git merge origin/main            # (equivalent to what `git pull` does)
```

---

## 22. Pull

```bash
git pull
```
Equivalent to:
```bash
git fetch
git merge
```
Pull is fetch + an automatic merge — convenient, but skips the "review before merging" safety step.

---

## 23. Fetch vs Pull

| | Fetch | Pull |
|---|---|---|
| Downloads new commits | Yes | Yes |
| Updates remote-tracking branch (`origin/main`) | Yes | Yes |
| Merges into your local branch | **No** | **Yes** |
| Touches working directory | No | Yes |

---

## 24. Fetch --all

```bash
git fetch --all
```
Fetches from **every configured remote** (useful when a repo has more than one — e.g. `origin` and `upstream` on a fork, or mirrors across GitHub/GitLab/Bitbucket).

---

## 25. Prune

```bash
git fetch --prune
```
If a branch was deleted on GitHub (or via `git push origin --delete`), your local remote-tracking branches don't know that automatically. `--prune` removes local references to remote branches that no longer exist on the remote — keeps `git branch -r` clean and accurate.

---

## 26. Delete Branches

```bash
git branch -d feature1                    # local delete (safe — refuses if unmerged)
git branch -D feature1                    # local delete (force)
git push origin --delete feature1         # remote delete
```

---

## 27. Fork

A **fork** is a full copy of someone else's repository under your own GitHub account.
```
Original Repo → Fork (GitHub) → Clone (Local Machine)
```

---

## 28. Fork vs Clone

| | Fork | Clone |
|---|---|---|
| Direction | GitHub → GitHub | GitHub → Local machine |
| Creates | A new repo under your account | A local working copy |
| Typical use | Contributing to someone else's project | Working on a repo you already have access to |

---

## 29. Rebase

An alternative to merge — replays your commits on top of another branch instead of creating a merge commit. Goal: **linear, cleaner history**.

```bash
git checkout feature
git rebase main
```
```
Before:
main:     C1 --- C2 --- C3 --- C4
                        \
feature:                 F1 --- F2

After:
          C1 --- C2 --- C3 --- C4 --- F1' --- F2'
```
Note the `'` — rebase doesn't move the original commits, it creates **new commits** with the same changes but new parents/hashes.

**Interactive rebase** — edit, squash, or reorder recent commits
```bash
git rebase -i HEAD~3
```
Used for: squashing commits into one, renaming commit messages, general history cleanup before a PR.

```bash
git rebase --continue     # after resolving a conflict mid-rebase
git rebase --abort        # bail out, return to pre-rebase state
```

**Golden rule:** rebase freely on your own **feature branches**. Never rebase `main`, `develop`, or any branch other people are also pulling from — since it rewrites history, it breaks everyone else's copy.

---

## 30. Daily Industry Workflow

```bash
git checkout main
git pull origin main

git checkout feature-login
# ...work...
git add .
git commit -m "feature completed"

git fetch origin
git rebase origin/main     # keep feature branch up to date, linear history

git push
```

---

## 31. Golden Interview Answers

| Question | One-line answer |
|---|---|
| Fetch vs Pull | Fetch = download only. Pull = fetch + merge. |
| Merge vs Rebase | Merge preserves history. Rebase rewrites it for a cleaner, linear graph. |
| Reset vs Revert | Reset rewrites history (moves HEAD back). Revert adds a new commit that undoes changes. |
| Fork vs Clone | Fork = copy on GitHub. Clone = copy on your local machine. |
| Stash vs Commit | Stash = temporary, off to the side. Commit = permanent, part of history. |
| Create new repo vs push existing | New = `init`+`add`+`commit`+`remote`+`push` (no local history yet). Existing = `remote`+`push` (history already exists). |
| Why `git push -u origin main`? | The `-u` sets up upstream tracking, so future `git push`/`git pull` need no arguments. |
| What is a remote-tracking branch? | A local bookmark (e.g. `origin/main`) recording where the remote branch was at your last fetch — it only moves on `fetch`/`pull`, never automatically. |
| Why does fetch feel "safer" than pull? | Fetch only updates the remote-tracking branch; nothing touches your working directory until you explicitly merge. |

---

## 32. Final Mental Model

```
Working Directory
      │  git add
      ▼
Staging Area
      │  git commit
      ▼
Local Repository
      │  git push               ▲  git fetch (updates origin/main only)
      ▼                         │  git pull  (fetch + merge into local branch)
Remote Repository (GitHub) ─────┘
```
