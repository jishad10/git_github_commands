# Git & GitHub Learning Notes (DevOps / Backend Developer Edition)

# 1. Git Basics

## Initialize Repository
```bash
git init
git init -b main
```

Creates a `.git` directory that stores Git metadata.

---

## Check Files
```bash
ls -l
ls -a
```

- `-l` = detailed view
- `-a` = show hidden files (including `.git`)

---

## Git Status
```bash
git status
git status -s
```

States:
- Untracked
- Modified
- Staged
- Committed

Short status:
```text
?? = Untracked
M  = Modified
A  = Added
```

---

# 2. Git Workflow

```text
Working Directory
       ↓
Staging Area
       ↓
Repository
```

## Stage Files
```bash
git add file.txt
git add .
```

Moves changes to staging area.

---

## Commit Changes
```bash
git commit -m "message"
```

A commit contains:

- Author (Who)
- Timestamp (When)
- Commit Message
- Snapshot
- Parent Commit
- SHA-1 Hash

---

## Empty Commit
```bash
git commit --allow-empty -m "trigger pipeline"
```

Use cases:
- Test CI/CD
- Trigger GitHub Actions
- Trigger Jenkins Pipeline

---

# 3. Snapshot vs Delta

## Snapshot

Git stores the state of the project.

```text
Commit 1
f1 = hello

Commit 2
f1 = hello world
```

Git thinks in snapshots.

---

## Delta (Conceptual)

Difference between versions.

```diff
- hello
+ hello world
```

Useful for understanding changes.

---

# 4. Git Log

```bash
git log
git log --oneline
git log -n 2
git log -p
git log --since="1 week ago"
git log --until="2025-01-01"
git log --author="name"
git log --grep="fix"
```

---

# 5. Git Diff

## Unstaged Changes
```bash
git diff
```

Working Directory vs Staging Area

---

## Staged Changes
```bash
git diff --staged
```

Staging Area vs Last Commit

---

## Compare Commits
```bash
git diff commit1 commit2
```

---

## Compare Branches
```bash
git diff main feature
```

---

# 6. Remove Tracking

## Remove from Git Only
```bash
git rm --cached file.txt
```

Removes file from Git tracking but keeps file on disk.

Common use:
```bash
git rm --cached .env
```

---

# 7. Branching

## Create Branch
```bash
git branch feature1
```

---

## Switch Branch
```bash
git checkout feature1
```

or

```bash
git switch feature1
```

---

## Create + Switch
```bash
git checkout -b feature1
```

---

## View Branches
```bash
git branch
```

---

# Branch Concept

```text
main

C1 ---- C2 ---- C3
                  ↑
                HEAD
```

Branch = Pointer to latest commit.

HEAD = Current branch pointer.

---

# 8. Merging

## Merge Feature Branch

```bash
git checkout main
git merge feature1
```

---

## Fast Forward Merge

```text
main

C1 --- C2

feature

C1 --- C2 --- F1
```

Result:

```text
C1 --- C2 --- F1
```

No merge commit.

---

## Three-Way Merge

```text
       F1
      /
C1 -- C2
      \
       M1
```

Creates merge commit.

---

# 9. Cherry Pick

Apply a specific commit.

```bash
git cherry-pick commitID
```

Use Case:

Feature 1 finished.
Feature 2 incomplete.

Deploy only Feature 1.

---

# 10. Reset

## Soft Reset

```bash
git reset --soft commitID
```

Moves HEAD only.

Changes remain staged.

---

## Mixed Reset (Default)

```bash
git reset --mixed commitID
```

Moves HEAD.

Changes become unstaged.

---

## Hard Reset

```bash
git reset --hard commitID
```

Moves HEAD.

Deletes local changes.

Dangerous.

---

# 11. Revert

```bash
git revert commitID
```

Creates a new commit that undoes changes.

Safe for shared branches.

---

# Reset vs Revert

## Reset

History rewritten.

```text
C1 --- C2 --- C3
       ↑
      HEAD
```

---

## Revert

History preserved.

```text
C1 --- C2 --- C3 --- C4(revert)
```

---

# 12. Stash

Temporarily save uncommitted work.

## Save

```bash
git stash
```

---

## Named Stash

```bash
git stash push -m "login feature"
```

---

## List

```bash
git stash list
```

---

## Apply

```bash
git stash apply
```

---

## Pop

```bash
git stash pop
```

Apply + Remove

---

## Delete One

```bash
git stash drop stash@{0}
```

---

## Delete All

```bash
git stash clear
```

---

## Include Untracked Files

```bash
git stash -u
```

---

# 13. .gitignore

Example:

```gitignore
node_modules/
.env
dist/
*.log
```

Common files to ignore:

- node_modules
- .env
- build artifacts
- logs

Important:

`.gitignore` does NOT affect already tracked files.

Solution:

```bash
git rm --cached .env
```

---

# 14. Remote Repository

## Add Remote

```bash
git remote add origin URL
```

---

## View Remote

```bash
git remote -v
```

---

# 15. Clone

```bash
git clone URL
```

Downloads repository and configures origin.

---

# 16. Push

## First Push

```bash
git push -u origin main
```

Creates upstream tracking.

---

## Future Pushes

```bash
git push
```

---

# 17. Fetch

Downloads updates only.

```bash
git fetch
```

or

```bash
git fetch origin main
```

No merge occurs.

---

# 18. Pull

```bash
git pull
```

Equivalent:

```bash
git fetch
git merge
```

---

# Fetch vs Pull

## Fetch

Download only.

## Pull

Fetch + Merge

---

# 19. Fetch All

```bash
git fetch --all
```

Fetch updates from all remotes.

---

# 20. Prune

```bash
git fetch --prune
```

Removes stale remote-tracking branches.

---

# 21. Delete Branches

## Local

```bash
git branch -d feature1
```

Force:

```bash
git branch -D feature1
```

---

## Remote

```bash
git push origin --delete feature1
```

---

# 22. Fork

Fork = GitHub copy of another repository.

```text
Original Repo
      ↓
    Fork
      ↓
   Clone
```

---

# Fork vs Clone

Fork:

```text
GitHub → GitHub
```

Clone:

```text
GitHub → Local Machine
```

---

# 23. Rebase

Alternative to merge.

Goal:

- Linear history
- Cleaner history
- Fewer merge commits

---

## Rebase Example

```bash
git checkout feature
git rebase main
```

Before:

```text
main

C1 --- C2 --- C3 --- C4
            \
             F1 --- F2
```

After:

```text
C1 --- C2 --- C3 --- C4 --- F1' --- F2'
```

---

## Interactive Rebase

```bash
git rebase -i HEAD~3
```

Useful for:

- Squash commits
- Rename commits
- Cleanup history

---

## Continue

```bash
git rebase --continue
```

---

## Abort

```bash
git rebase --abort
```

---

# Rebase Rule

Safe:

```text
Feature Branches
```

Avoid:

```text
main
develop
shared branches
```

---

# 24. Daily Industry Workflow

```bash
git checkout main
git pull origin main

git checkout feature-login

# work

git add .
git commit -m "feature completed"

git fetch origin
git rebase origin/main

git push
```

---

# Golden Interview Answers

## Fetch vs Pull

Fetch = Download

Pull = Fetch + Merge

---

## Merge vs Rebase

Merge = Preserve History

Rebase = Rewrite History for Cleaner Graph

---

## Reset vs Revert

Reset = Rewrite History

Revert = New Commit That Undoes Changes

---

## Fork vs Clone

Fork = GitHub Copy

Clone = Local Copy

---

## Stash vs Commit

Stash = Temporary

Commit = Permanent History

---

## Why git push -u origin main?

Creates upstream tracking relationship.

---

# Final Rule

Always remember:

Working Directory
→ git add
→ Staging Area
→ git commit
→ Local Repository
→ git push
→ Remote Repository
