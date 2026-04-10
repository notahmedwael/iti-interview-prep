# 🌿 Git — The Complete Field Guide

> **Part 9 of Ahmed's shadow interview prep.**
> Not just commands — real scenarios that make you sweat, with ASCII visualizations showing exactly what's happening under the hood. Read this like a story.

---

## 📑 Table of Contents

1.  [How Git Actually Works (The Mental Model)](#1-how-git-actually-works-the-mental-model)
2.  [Setup & Configuration](#2-setup--configuration)
3.  [The Three Trees & Staging Area](#3-the-three-trees--staging-area)
4.  [Branching — The Core of Git](#4-branching--the-core-of-git)
5.  [Merge vs Rebase — The Most Important Decision](#5-merge-vs-rebase--the-most-important-decision)
6.  [Remote Workflows](#6-remote-workflows)
7.  [Undoing Things — The Panic Section](#7-undoing-things--the-panic-section)
8.  [Stash — The Escape Hatch](#8-stash--the-escape-hatch)
9.  [Cherry-Pick — Surgical Commit Transplants](#9-cherry-pick--surgical-commit-transplants)
10. [Tags & Releases](#10-tags--releases)
11. [Advanced Commands That Will Save You](#11-advanced-commands-that-will-save-you)
12. [Git Bisect — Finding the Bug in History](#12-git-bisect--finding-the-bug-in-history)
13. [Submodules](#13-submodules)
14. [Hooks — Automate Git Events](#14-hooks--automate-git-events)
15. [Real Scenarios — "What Do I Do?"](#15-real-scenarios--what-do-i-do)
16. [Interview Questions & Answers](#16-interview-questions--answers)

---

## 1. How Git Actually Works (The Mental Model)

Before any command makes sense, you need to understand what Git actually stores.

### Git Stores Snapshots, Not Diffs

```
Most version control systems store diffs (what changed):
  Version 1: "Hello World"
  Version 2: + "How are you?"   ← stores the change

Git stores SNAPSHOTS (the full state at each point):
  Commit A: {file1: "Hello World", file2: "..."}
  Commit B: {file1: "Hello World\nHow are you?", file2: "..."}
  ← stores the whole tree, but reuses unchanged files via content hashing
```

### The Object Database

```
Git is fundamentally a content-addressable key-value store.
Everything is stored as an "object" identified by its SHA-1 hash.

Object types:
  blob    → file content
  tree    → directory (list of blobs + trees with names)
  commit  → snapshot (tree + parent + author + message)
  tag     → annotated tag object

Every commit points to:
  - A tree (the full project snapshot)
  - Parent commit(s) (previous state)
  - Author, timestamp, message

This means:
  ✅ Commits are immutable — the hash is based on content
  ✅ History is a linked list (or DAG for merges)
  ✅ Changing anything creates a new hash (new commit)
```

### The Commit DAG (Directed Acyclic Graph)

```
What your commit history actually looks like:

A ← B ← C ← D ← E  (main)
              ↑
              └── F ← G  (feature/login)

Each ← means "this commit's parent is..."
Every commit knows its parent(s).
Branches are just named pointers to commits.
HEAD is the pointer to your current location.
```

### Visualizing HEAD

```
HEAD can point to:
  1. A branch (normal state — "attached HEAD")
  2. A commit directly ("detached HEAD" — be careful!)

Attached HEAD:
  HEAD → main → [commit E]

Detached HEAD (happened after git checkout <commit-hash>):
  HEAD → [commit C]  (branch pointer still at E)
  ⚠️ Any commits you make here will be "floating" — no branch points to them
  ⚠️ Git garbage collection can delete them!
```

---

## 2. Setup & Configuration

```bash
# ── Identity (required before first commit) ─────────────────────────────────
git config --global user.name  "Ahmed Ali"
git config --global user.email "ahmed@kickoff.dev"

# ── Editor ──────────────────────────────────────────────────────────────────
git config --global core.editor "code --wait"   # VS Code
git config --global core.editor "vim"
git config --global core.editor "nano"

# ── Default branch name ─────────────────────────────────────────────────────
git config --global init.defaultBranch main

# ── Line endings (critical for cross-platform teams) ───────────────────────
# On Mac/Linux:
git config --global core.autocrlf input   # convert CRLF→LF on commit

# On Windows:
git config --global core.autocrlf true    # convert LF→CRLF on checkout, CRLF→LF on commit

# ── Useful aliases ──────────────────────────────────────────────────────────
git config --global alias.st    status
git config --global alias.co    checkout
git config --global alias.br    branch
git config --global alias.lg    "log --oneline --graph --decorate --all"
git config --global alias.last  "log -1 HEAD"
git config --global alias.unstage "reset HEAD --"
git config --global alias.wip   "!git add -A && git commit -m 'WIP'"

# ── View config ─────────────────────────────────────────────────────────────
git config --list
git config --list --show-origin   # where each setting comes from

# Config levels (later overrides earlier):
#   --system  → /etc/gitconfig         (all users on machine)
#   --global  → ~/.gitconfig           (your user)
#   --local   → .git/config            (this repo only) ← default

# ── Initialize / Clone ──────────────────────────────────────────────────────
git init                          # start a new repo in current directory
git init my-project               # create a new directory and init inside it
git clone https://github.com/user/repo.git
git clone https://github.com/user/repo.git my-folder  # clone into specific folder
git clone --depth 1 https://github.com/user/repo.git  # shallow clone (latest only)
git clone --branch develop https://...    # clone and checkout specific branch
```

---

## 3. The Three Trees & Staging Area

This is the most misunderstood concept in Git. Understanding it makes every command click.

### The Three Areas

```
┌─────────────────┐    git add     ┌──────────────────┐    git commit   ┌───────────────────┐
│  Working Tree   │ ─────────────► │  Staging Area    │ ──────────────► │  Repository       │
│  (your files)   │                │  (Index)         │                 │  (.git/objects)   │
│                 │ ◄───────────── │                  │ ◄────────────── │                   │
│  What you edit  │  git restore   │  What will go    │  git reset      │  Committed history│
│                 │  (discard)     │  into next commit│  (unstage)      │                   │
└─────────────────┘                └──────────────────┘                 └───────────────────┘

git status shows you the difference between these three areas.
```

### Status, Add & Commit

```bash
# ── See the state of all three areas ────────────────────────────────────────
git status
git status -s            # short format:  M = modified, A = added, ? = untracked
git status --short --branch  # with branch info

# ── Staging (adding to index) ────────────────────────────────────────────────
git add file.txt             # stage one file
git add src/                 # stage everything in src/ directory
git add .                    # stage everything in current directory (+ subdirs)
git add -A                   # stage everything: new + modified + deleted
git add -p                   # INTERACTIVE: stage specific HUNKS within files
# git add -p is gold — choose exactly which changes within a file to stage

# ── Committing ───────────────────────────────────────────────────────────────
git commit -m "feat: add booking validation"
git commit                   # opens editor for longer message
git commit -am "fix: typo"   # add ALL tracked modified files AND commit (skip staging)
                             # ⚠️ -a does NOT add untracked new files
git commit --amend           # modify the LAST commit (message or content)
git commit --amend --no-edit # amend last commit keeping the same message

# ── Viewing diff ─────────────────────────────────────────────────────────────
git diff                     # working tree vs staging area (what's NOT staged yet)
git diff --staged            # staging area vs last commit (what IS staged)
git diff HEAD                # working tree vs last commit (all changes)
git diff main..feature/login # compare two branches
git diff abc123 def456       # compare two commits

# ── Log ──────────────────────────────────────────────────────────────────────
git log
git log --oneline            # compact: one commit per line
git log --oneline --graph    # with branch visualization
git log --oneline --graph --decorate --all   # full picture including all branches
git log -n 5                 # last 5 commits
git log --author="Ahmed"     # filter by author
git log --since="2024-01-01" # filter by date
git log --grep="booking"     # filter by commit message
git log -p                   # show patch (diff) for each commit
git log -- path/to/file.ts   # history of a specific file
git log main..feature        # commits in feature that aren't in main

# Beautiful log alias:
git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --all
```

### .gitignore

```bash
# .gitignore patterns
node_modules/      # ignore the node_modules directory
*.log              # ignore all .log files
.env               # ignore .env file
.env.*             # ignore .env.local, .env.production, etc.
!.env.example      # except .env.example (! = negate)
dist/              # ignore built output
*.tmp              # ignore temp files
.DS_Store          # ignore macOS metadata
**/.cache/         # ignore .cache/ anywhere in the tree

# Check why a file IS or ISN'T ignored
git check-ignore -v src/generated.ts

# If you already committed a file that should be ignored:
git rm --cached .env          # remove from tracking (keep local file)
git rm --cached -r node_modules/   # remove directory from tracking

# Global gitignore (for editor files, OS files — not project-specific)
git config --global core.excludesfile ~/.gitignore_global
# ~/.gitignore_global:
# .DS_Store
# .idea/
# .vscode/
# *.swp
```

---

## 4. Branching — The Core of Git

### Branch Commands

```bash
# ── Create & switch ──────────────────────────────────────────────────────────
git branch                        # list local branches (* = current)
git branch -a                     # list local + remote branches
git branch feature/booking        # create branch (stays where you are)
git checkout feature/booking      # switch to branch
git checkout -b feature/booking   # create AND switch (classic)
git switch feature/booking        # modern: switch (Git 2.23+)
git switch -c feature/booking     # modern: create AND switch

# Create branch from specific point:
git checkout -b hotfix/login main     # branch from main
git checkout -b feature/X abc123      # branch from a specific commit

# ── Delete ───────────────────────────────────────────────────────────────────
git branch -d feature/booking     # delete (safe — won't delete unmerged work)
git branch -D feature/booking     # force delete (even if unmerged)
git push origin --delete feature/booking  # delete remote branch

# ── Rename ───────────────────────────────────────────────────────────────────
git branch -m old-name new-name       # rename local branch
git branch -m new-name                # rename CURRENT branch

# ── Track remote branches ────────────────────────────────────────────────────
git branch -u origin/main             # set upstream for current branch
git branch --set-upstream-to=origin/main feature/login

# ── See tracking info ────────────────────────────────────────────────────────
git branch -vv                    # verbose: shows upstream + ahead/behind
```

### Visualizing Branch Operations

```
Start:
  main:  A ─ B ─ C
                 ↑ HEAD

git checkout -b feature/login:
  main:    A ─ B ─ C
  feature:         ↑ (same point, new branch pointer)
  HEAD → feature/login

Make commits on feature:
  main:    A ─ B ─ C
  feature: A ─ B ─ C ─ D ─ E
                             ↑ HEAD

Meanwhile, main got new commits:
  main:    A ─ B ─ C ─ F ─ G
  feature: A ─ B ─ C ─ D ─ E ─ ↑ HEAD
```

---

## 5. Merge vs Rebase — The Most Important Decision

This is the single most debated topic in Git. Get this right and everything else falls into place.

### The Setup — Two Branches Have Diverged

```
Start state:
  main:    A ─ B ─ C ─ F ─ G
                   ↑
                   └── D ─ E  (feature/login)

C is the common ancestor (where the branches split).
F and G are on main — happened while you worked on feature.
D and E are on feature — your new work.

Both approaches must integrate these changes. They just do it differently.
```

---

### MERGE — "These histories really happened this way"

```
git checkout main
git merge feature/login

Result (fast-forward when possible, merge commit when not):

If main hadn't moved since branching (fast-forward):
  Before: main: A─B─C,  feature: A─B─C─D─E
  After:  main: A─B─C─D─E  (no merge commit, pointer just moves forward)
  HEAD → main → E

If main DID move (creates a merge commit M):
  Before: main: A─B─C─F─G,  feature: A─B─C─D─E
  After:
      main:    A ─ B ─ C ─ F ─ G ─ M
                         \         ↗
                          D ─ E ──

  M = merge commit with TWO parents (G and E)
  Both histories are preserved
  git log shows: M, G, E, F, D, C, B, A (interleaved by time)
```

**Merge Characteristics:**
```
✅ Preserves complete historical context — every commit stays exactly where it happened
✅ Non-destructive — doesn't change existing commits
✅ Easier to understand "what actually happened"
✅ Merge commit clearly marks where integration happened
❌ Creates "merge commits" that some find noisy
❌ git log can look tangled with lots of branches
❌ Harder to find a specific feature's changes (interleaved with others)
```

---

### REBASE — "Pretend my work started from the tip of main"

```
git checkout feature/login
git rebase main

What rebase does step by step:
  1. Finds the common ancestor (C)
  2. Saves your commits (D, E) as patches
  3. Resets feature to point to main's tip (G)
  4. Re-applies your patches one by one → D' and E'
  (D' and E' are NEW commits with new hashes — same changes, different history)

Before:
  main:    A ─ B ─ C ─ F ─ G
                   ↑
                   └── D ─ E  (feature/login)

After: git rebase main (from feature/login branch)
  main:    A ─ B ─ C ─ F ─ G
                             └── D' ─ E'  (feature/login)

  D' and E' are COPIES of D and E but their parent is now G, not C.
  The original D and E are orphaned (will be garbage collected).
```

**Rebase Characteristics:**
```
✅ Clean, linear history — git log looks like a straight line
✅ Each feature's commits stay together, easy to review
✅ Easier to use git bisect (linear history)
✅ No merge commits cluttering the log
❌ REWRITES HISTORY — commits get new hashes
❌ NEVER rebase commits that have been pushed to a shared branch (see golden rule)
❌ Can be confusing during conflicts (you resolve conflicts commit by commit)
❌ Hides "what actually happened" — makes history look cleaner than reality
```

---

### The Golden Rule of Rebase

```
╔══════════════════════════════════════════════════════════════════╗
║  NEVER rebase commits that exist on a remote shared branch.     ║
║  Never rebase main, develop, or any branch others work on.      ║
╚══════════════════════════════════════════════════════════════════╝

Why? Because rebase creates NEW commits (new hashes).
If your colleague has the OLD commits and you push the new ones,
their history and your history will diverge. They'll have a nightmare merge.

Safe: rebase your LOCAL feature branch before merging to shared branch
Dangerous: rebase main, rebase a branch others have checked out
```

---

### When to Choose Each

```
USE MERGE WHEN:
  ✅ Integrating a completed feature into main/develop
  ✅ Working on a shared branch with teammates
  ✅ You want to preserve the exact history ("this PR was merged on this date")
  ✅ Complex features where the branch history is meaningful context
  ✅ Hotfixes — clearly shows when the fix was integrated

USE REBASE WHEN:
  ✅ Updating your LOCAL feature branch with latest main before a PR
  ✅ You want a clean, linear history for review
  ✅ Cleaning up messy local commits before pushing (interactive rebase)
  ✅ Personal projects where you're the only contributor

REAL-WORLD TEAM WORKFLOW:
  1. Create feature branch from main
  2. Work on feature, commit locally
  3. git rebase main  ← sync your branch with latest main (rebase = clean)
  4. Push feature branch, open PR
  5. Team reviews, approves
  6. git merge feature/login  ← integrate to main (merge = preserve PR history)
  7. Delete feature branch

OR (squash merge workflow):
  5. Squash all feature commits into ONE commit on main
     git merge --squash feature/login
     git commit -m "feat: add login (#42)"
     → Clean main history, each PR = one commit
```

---

### Merge Commands

```bash
# Standard merge (creates merge commit if needed)
git merge feature/login

# Force a merge commit even for fast-forward (no-fast-forward)
git merge --no-ff feature/login
# This is the "always use merge commits" preference — shows when feature was integrated

# Only merge if fast-forward is possible (fail otherwise)
git merge --ff-only feature/login

# Squash merge — all feature commits become staged changes, you make one commit
git merge --squash feature/login
git commit -m "feat: add login system (#42)"

# Abort a merge that's in conflict
git merge --abort

# Continue a merge after resolving conflicts
git add .
git merge --continue

# Visualize what a merge would do (dry run)
git merge --no-commit --no-ff feature/login
# Then: git merge --abort  if you don't like what you see
```

### Rebase Commands

```bash
# Basic rebase — replay feature commits on top of main
git checkout feature/login
git rebase main

# Abort a rebase in progress
git rebase --abort

# Continue after resolving conflicts at a commit
git add .
git rebase --continue

# Skip a commit that's entirely conflicting and you want to drop it
git rebase --skip

# Interactive rebase — the most powerful command in Git
git rebase -i HEAD~3         # interactive rebase of last 3 commits
git rebase -i main           # interactive rebase of all commits not in main

# In interactive mode, each commit has an action:
# pick   p → keep as-is
# reword r → keep commit but change the message
# edit   e → stop and let me amend this commit
# squash s → combine with previous commit (keep both messages)
# fixup  f → combine with previous commit (discard THIS message)
# drop   d → remove this commit entirely
# exec   x → run a shell command after this commit

# Rebase onto — more surgical
git rebase --onto main feature/base feature/child
# "Take commits on feature/child that aren't on feature/base, and replay them on main"
# Used when: feature/child was based on feature/base, and feature/base was merged
```

---

### Interactive Rebase — Cleaning Up Before PR

```
Scenario: You made a mess of commits locally and want to clean up before pushing.

Your current branch:
  D: "add login form"
  E: "fix typo"
  F: "fix another typo"
  G: "WIP save"
  H: "finally working!!"
  I: "remove debug logs"

These 6 commits should be 2 clean commits. Interactive rebase to the rescue:

git rebase -i HEAD~6

Git opens editor with:
  pick D add login form
  pick E fix typo
  pick F fix another typo
  pick G WIP save
  pick H finally working!!
  pick I remove debug logs

You change it to:
  pick D add login form         ← keep
  fixup E fix typo              ← squash into D, discard E's message
  fixup F fix another typo      ← squash into D, discard F's message
  squash G WIP save             ← squash into D, KEEP G's message (will prompt)
  fixup H finally working!!     ← squash into D
  fixup I remove debug logs     ← squash into D

Save → Git replays and squashes.

Result:
  D': "add login form" (with all the fixes incorporated)
  → One clean commit. Your PR reviewer will thank you.

ALTERNATIVE: squash G as "pick" and combine only E,F,H,I as fixup,
to get two meaningful commits: "add login form" + "add session handling"
```

---

## 6. Remote Workflows

```
Terminology:
  origin  → the default name for the remote you cloned from (can be renamed)
  upstream → often used for the original repo when you've forked
  remote  → any remote repository (GitHub, GitLab, self-hosted)
```

```bash
# ── Remote management ────────────────────────────────────────────────────────
git remote -v                          # list remotes with URLs
git remote add origin https://...     # add a remote named 'origin'
git remote add upstream https://...   # add another remote (e.g., original after fork)
git remote rename origin old-origin   # rename a remote
git remote remove upstream            # remove a remote
git remote set-url origin https://new-url.git  # change the URL of a remote

# ── Fetch — download changes without affecting working tree ─────────────────
git fetch origin                   # fetch all branches from origin
git fetch origin main              # fetch just main
git fetch --all                    # fetch from all remotes
git fetch --prune                  # fetch AND delete local references to deleted remote branches

# ── Pull — fetch + merge (or fetch + rebase) ─────────────────────────────────
git pull                           # fetch + merge (default)
git pull origin main               # explicit: pull main from origin
git pull --rebase                  # fetch + rebase instead of merge
git pull --rebase=interactive      # fetch + interactive rebase (review what you're rebasing)

# Set rebase as default for pulls (recommended for clean history):
git config --global pull.rebase true

# ── Push ─────────────────────────────────────────────────────────────────────
git push                           # push current branch to its upstream
git push origin feature/login      # push specific branch to origin
git push -u origin feature/login   # push AND set upstream tracking
git push --all origin              # push all local branches to origin
git push origin --delete feature/login   # delete remote branch
git push --tags                    # push all tags
git push origin v1.2.0             # push specific tag

# Force push — DANGEROUS on shared branches
git push --force                   # force push (can destroy others' work)
git push --force-with-lease        # SAFER: force push only if no one else pushed
                                   # (fails if remote has commits you don't have locally)
# Always use --force-with-lease instead of --force

# ── Tracking ─────────────────────────────────────────────────────────────────
git branch -vv                     # see all branches and their upstream
git branch --set-upstream-to=origin/main main   # set tracking branch
```

### Fetch vs Pull — Visualized

```
Before fetch:
  Local:   A ─ B ─ C  (main)
  Remote:  A ─ B ─ C ─ D ─ E  (origin/main)

After git fetch origin:
  Local main:          A ─ B ─ C  (unchanged — your branch NOT moved)
  Local origin/main:   A ─ B ─ C ─ D ─ E  (remote tracking updated)

Now you can review D and E before merging:
  git diff main origin/main
  git log origin/main --not main

After git merge origin/main (or git pull which does both):
  Local main: A ─ B ─ C ─ D ─ E ─ M  (or fast-forward to E if no local commits)

git pull = git fetch + git merge  (or + git rebase if configured)
```

---

## 7. Undoing Things — The Panic Section

This section will save your job one day. Know these cold.

### The Undo Decision Tree

```
What did you do?

├── Changed files but didn't stage (just edited)
│   └── git restore file.txt           (discard working tree changes)
│       git restore .                   (discard ALL working tree changes)
│       git checkout -- file.txt        (old syntax, same thing)

├── Staged changes (git add) but didn't commit
│   └── git restore --staged file.txt  (unstage, keep changes in working tree)
│       git reset HEAD file.txt         (old syntax, same thing)

├── Committed (locally, not pushed yet)
│   ├── Want to undo last commit but KEEP changes staged
│   │   └── git reset --soft HEAD~1
│   ├── Want to undo last commit and UNSTAGE changes (keep in working tree)
│   │   └── git reset --mixed HEAD~1   (default reset behavior)
│   └── Want to completely NUKE last commit and all changes
│       └── git reset --hard HEAD~1    ⚠️ DESTRUCTIVE — changes gone forever

├── Committed and PUSHED to shared branch
│   └── NEVER rewrite pushed history! Use git revert instead:
│       git revert HEAD               (create new commit that undoes last commit)
│       git revert abc123             (create new commit that undoes specific commit)
│       git revert abc123..def456     (revert a range of commits)

└── Something else terrible happened
    └── git reflog                    (your lifeline — see EVERYTHING git tracked)
```

### reset: Three Modes Visualized

```
State before reset:
  Repo:         A ─ B ─ C ─ D  (D = HEAD, your commits)
  Staging:      [files staged for next commit]
  Working tree: [files you edited]

git reset --soft HEAD~2  (move HEAD back 2, keep staging + working tree)
  Repo:         A ─ B  (HEAD moved back to B)
  Staging:      [C's changes + D's changes are staged]
  Working tree: [unchanged]
  → Useful: "I want to squash my last 2 commits into one"

git reset --mixed HEAD~2  (default — move HEAD, unstage, keep working tree)
  Repo:         A ─ B
  Staging:      [empty — nothing staged]
  Working tree: [C's changes + D's changes are here, untracked]
  → Useful: "Undo my commits but keep the file changes to redo them"

git reset --hard HEAD~2  ⚠️ DESTRUCTIVE
  Repo:         A ─ B
  Staging:      [empty]
  Working tree: [empty — changes from C and D are GONE]
  → Useful: "These commits were a complete mistake, delete everything"
```

```bash
# ── Restore (Git 2.23+, cleaner than checkout/reset for files) ───────────────
git restore file.txt              # discard working tree changes to file.txt
git restore .                     # discard ALL working tree changes
git restore --staged file.txt     # unstage file (keep working tree changes)
git restore --staged .            # unstage everything
git restore --source=HEAD~3 file.txt  # restore file to its state 3 commits ago

# ── Reset ────────────────────────────────────────────────────────────────────
git reset HEAD~1             # undo last commit, unstage changes (--mixed is default)
git reset --soft HEAD~1      # undo last commit, KEEP staged
git reset --hard HEAD~1      # undo last commit, DESTROY changes ⚠️
git reset --hard origin/main # reset to exactly what's on remote (nuclear option)

# ── Revert — safe undo for shared history ────────────────────────────────────
git revert HEAD              # create new commit that undoes HEAD
git revert abc123            # create new commit that undoes commit abc123
git revert abc123 --no-edit  # auto-accept the commit message
git revert abc123..def456    # revert a range (creates multiple revert commits)
git revert -n abc123         # revert but DON'T auto-commit (let me stage manually)

# ── Clean — remove untracked files ───────────────────────────────────────────
git clean -n                 # DRY RUN: show what would be deleted
git clean -f                 # delete untracked files
git clean -fd                # delete untracked files AND directories
git clean -fdx               # delete untracked + gitignored files (nuclear)
# ALWAYS run git clean -n first to preview!

# ── Reflog — your safety net ─────────────────────────────────────────────────
git reflog                   # see ALL recent HEAD movements
git reflog show feature/login # reflog for a specific branch
# Every commit, checkout, merge, rebase is recorded here
# Data is kept for 90 days by default
# Even "deleted" commits can be recovered via reflog!
```

### The Revert vs Reset Visualization

```
Revert — safe for shared history:
  Before: A ─ B ─ C ─ D  (C was bad, D depended on C)
  After git revert C:
          A ─ B ─ C ─ D ─ C'
  C' = "Revert C" — undoes C's changes, doesn't erase C from history
  ✅ Safe: everyone who has C can pull C' without conflict

Reset — rewrites history (only for local/private commits):
  Before: A ─ B ─ C ─ D  (C was bad)
  After git reset --hard B:
          A ─ B  (C and D are gone)
  ❌ Dangerous if pushed: others have C and D, you've removed them
```

---

## 8. Stash — The Escape Hatch

Stash temporarily shelves changes so you can switch context without committing.

```bash
# ── Save to stash ────────────────────────────────────────────────────────────
git stash                         # stash tracked modified files
git stash -u                      # stash + untracked files too
git stash -a                      # stash everything (+ gitignored files)
git stash push -m "WIP: booking form validation"  # stash with a message
git stash push -- src/booking.ts  # stash only specific file(s)
git stash push -p                 # interactive: choose which hunks to stash

# ── List stashes ─────────────────────────────────────────────────────────────
git stash list
# stash@{0}: On feature/login: WIP: booking form validation
# stash@{1}: On main: hotfix experiment
# stash@{2}: WIP on feature/login: abc123 add form

# ── Apply stash ──────────────────────────────────────────────────────────────
git stash pop                     # apply most recent stash AND remove it from stash list
git stash apply                   # apply most recent stash but KEEP it in list
git stash pop stash@{2}           # apply a specific stash
git stash apply stash@{1}         # apply specific stash and keep it

# ── Inspect stash ─────────────────────────────────────────────────────────────
git stash show                    # what files changed in most recent stash
git stash show -p                 # full diff of most recent stash
git stash show -p stash@{1}       # full diff of specific stash

# ── Create branch from stash ─────────────────────────────────────────────────
git stash branch feature/rescue stash@{0}
# Creates a new branch, checks it out, applies the stash, drops it
# Useful when the stash conflicts with current branch

# ── Delete stash ─────────────────────────────────────────────────────────────
git stash drop                    # remove most recent stash
git stash drop stash@{2}          # remove specific stash
git stash clear                   # remove ALL stashes ⚠️
```

### Stash Scenario

```
You're halfway through refactoring the booking form.
Your boss messages: "URGENT — payment is broken in production!"

git stash push -m "WIP: booking form refactor"
git checkout main
git pull
git checkout -b hotfix/payment-broken
# ... fix the payment bug ...
git commit -m "fix: payment gateway timeout handling"
git push origin hotfix/payment-broken
# (PR merged by colleague)
git checkout feature/booking-form
git stash pop
# Back to where you were. 30 second context switch.
```

---

## 9. Cherry-Pick — Surgical Commit Transplants

Cherry-pick takes a specific commit from one branch and applies it to another.

```bash
git cherry-pick abc123                 # apply commit abc123 to current branch
git cherry-pick abc123 def456          # apply two specific commits
git cherry-pick abc123..def456         # apply a RANGE (exclusive: not abc123)
git cherry-pick abc123^..def456        # apply a range (inclusive: includes abc123)
git cherry-pick -n abc123              # apply but DON'T auto-commit (stage only)
git cherry-pick --no-commit abc123     # same as above
git cherry-pick -x abc123             # add "(cherry picked from commit abc123)" to message
git cherry-pick --abort               # abort in-progress cherry-pick
git cherry-pick --continue            # continue after resolving conflicts
```

### Cherry-Pick Scenario

```
The team has two active branches:
  main:    A ─ B ─ C ─ F ─ G
  develop: A ─ B ─ C ─ D ─ E ─ H

Commit E on develop = "fix: prevent double booking" — a critical bug fix.
This fix needs to be in main NOW (without merging all of develop).

git checkout main
git cherry-pick E

Result:
  main:    A ─ B ─ C ─ F ─ G ─ E'
  develop: A ─ B ─ C ─ D ─ E ─ H

E' = E's changes applied on top of G (same changes, new hash, new parent)
The fix is now in both branches.

⚠️ Cherry-pick is a copy, not a move.
   E still exists on develop AND E' now exists on main.
   If you later merge develop into main, git will see E and E' and might create conflicts.
   Use sparingly — prefer proper branching and merging.
```

---

## 10. Tags & Releases

```bash
# ── Lightweight tags (just a pointer to a commit) ────────────────────────────
git tag v1.0.0               # tag current commit
git tag v1.0.0 abc123        # tag a specific commit

# ── Annotated tags (full object — recommended for releases) ──────────────────
git tag -a v1.0.0 -m "Release version 1.0.0"
git tag -a v1.0.0 abc123 -m "Release version 1.0.0"

# ── View tags ────────────────────────────────────────────────────────────────
git tag                      # list all tags
git tag -l "v1.*"            # list tags matching pattern
git show v1.0.0              # show tag details + the commit it points to

# ── Push tags ────────────────────────────────────────────────────────────────
git push origin v1.0.0       # push a specific tag
git push origin --tags       # push ALL tags
git push --follow-tags       # push commits AND reachable annotated tags

# ── Delete tags ──────────────────────────────────────────────────────────────
git tag -d v1.0.0-beta       # delete local tag
git push origin --delete v1.0.0-beta  # delete remote tag

# ── Checkout a tag ───────────────────────────────────────────────────────────
git checkout v1.0.0          # ⚠️ detached HEAD — you're on a tag, not a branch
git checkout -b hotfix/v1.0.0 v1.0.0  # create branch from tag (safe)
```

### Semantic Versioning with Tags

```
MAJOR.MINOR.PATCH  →  v2.3.1

v1.0.0  → initial release
v1.0.1  → patch: bug fix, backwards compatible
v1.1.0  → minor: new feature, backwards compatible
v2.0.0  → major: breaking changes

Pre-release:
  v2.0.0-alpha.1  → early alpha
  v2.0.0-beta.2   → beta testing
  v2.0.0-rc.1     → release candidate
```

---

## 11. Advanced Commands That Will Save You

```bash
# ── git blame — who changed what line and when ──────────────────────────────
git blame file.ts                        # show author + commit for each line
git blame -L 10,25 file.ts              # only lines 10-25
git blame -w file.ts                    # ignore whitespace changes
git blame --ignore-rev abc123 file.ts   # ignore a specific commit (e.g., mass formatting)

# ── git log — power filtering ───────────────────────────────────────────────
git log --all --full-history -- "**/booking.service.ts"  # deleted file history
git log -S "calculatePrice"             # commits that ADDED or REMOVED this string
git log -G "calculatePrice.*hours"      # commits where this regex changed
git log --merges                        # only merge commits
git log --no-merges                     # exclude merge commits
git log --first-parent main             # only direct commits on main (skip merged branches)

# ── git show ────────────────────────────────────────────────────────────────
git show HEAD                           # last commit's changes
git show abc123                         # specific commit's changes
git show v1.0.0:src/booking.ts          # file content at a specific tag/commit
git show HEAD~3:package.json            # package.json from 3 commits ago

# ── git diff with power ─────────────────────────────────────────────────────
git diff --stat main feature            # summary: how many lines changed per file
git diff --name-only main feature       # just the filenames that changed
git diff --name-status main feature     # filenames + status (M=modified, A=added)

# ── git ls-files ─────────────────────────────────────────────────────────────
git ls-files                            # list all tracked files
git ls-files --others --exclude-standard # list untracked files
git ls-files --deleted                  # list deleted (but not staged) files

# ── git shortlog ─────────────────────────────────────────────────────────────
git shortlog -sn                        # contributor stats (number of commits)
git shortlog -sn --since="1 month"      # last month's activity

# ── git archive ─────────────────────────────────────────────────────────────
git archive --format=zip HEAD -o snapshot.zip          # zip the current state
git archive --format=tar.gz v1.0.0 -o release.tar.gz  # zip a specific tag

# ── git worktree — work on multiple branches simultaneously ─────────────────
# Run two branches side by side in different folders (no stash needed!)
git worktree add ../hotfix-branch main      # new folder for main
git worktree add ../feature-review feature/login  # new folder for feature
git worktree list
git worktree remove ../hotfix-branch        # remove when done

# ── git grep — search across tracked files ──────────────────────────────────
git grep "calculatePrice"               # search all tracked files
git grep -n "TODO"                      # with line numbers
git grep -l "calculatePrice"            # just filenames
git grep "calculatePrice" v1.0.0        # search in a specific version

# ── git rev-parse ────────────────────────────────────────────────────────────
git rev-parse HEAD                      # get full SHA of HEAD
git rev-parse --short HEAD              # short SHA
git rev-parse HEAD~3                    # SHA of 3 commits ago
git rev-parse --abbrev-ref HEAD         # current branch name (useful in scripts)

# ── git count-objects ────────────────────────────────────────────────────────
git count-objects -vH                   # repo size info
git gc                                  # garbage collect (clean up old objects)
git gc --aggressive                     # more thorough gc
```

---

## 12. Git Bisect — Finding the Bug in History

Bisect uses binary search to find the commit that introduced a bug. O(log n) — for 1000 commits, you only check ~10.

```bash
# ── The workflow ─────────────────────────────────────────────────────────────
git bisect start               # begin bisect session
git bisect bad                 # current commit IS buggy
git bisect good v1.0.0         # this tag/commit was GOOD (bug didn't exist here)

# Git now checks out the middle commit.
# Test if the bug exists here, then mark it:
git bisect bad                 # this commit also has the bug
# OR
git bisect good                # this commit is fine

# Git keeps halving until it finds the first bad commit.
# When done:
git bisect reset               # return to original HEAD

# ── Automated bisect ─────────────────────────────────────────────────────────
# Write a test script that exits 0 for good, non-zero for bad:
cat > test.sh << 'EOF'
#!/bin/bash
npm test -- --grep "booking validation"  # returns 0 = pass, 1 = fail
EOF
chmod +x test.sh

git bisect start
git bisect bad HEAD
git bisect good v1.0.0
git bisect run ./test.sh       # Git runs script at each midpoint AUTOMATICALLY
# Git finds the bad commit without any manual intervention!
git bisect reset
```

### Bisect Visualization

```
History: A ─ B ─ C ─ D ─ E ─ F ─ G ─ H ─ I ─ J  (J is current, buggy)
         ✅                                      ❌

git bisect start
git bisect bad     (J is bad)
git bisect good A

Step 1: Git checks out E (middle)
  Test E → Good ✅
  git bisect good

Step 2: Git checks out H (middle of E─J)
  Test H → Bad ❌
  git bisect bad

Step 3: Git checks out G (middle of E─H)
  Test G → Good ✅
  git bisect good

Step 4: Git checks out H again? No — now it knows:
  G is good, H is bad → H is the first bad commit!

"abc123 is the first bad commit"
git bisect reset  → back to J
git show abc123   → see what H changed
```

---

## 13. Submodules

Submodules let you include another Git repository as a subdirectory.

```bash
# ── Add a submodule ──────────────────────────────────────────────────────────
git submodule add https://github.com/team/shared-components src/shared
git submodule add https://github.com/team/shared-components  # added to repo root

# ── Clone a repo WITH its submodules ────────────────────────────────────────
git clone --recurse-submodules https://github.com/kickoff/app.git
# Or if you already cloned without it:
git submodule update --init --recursive

# ── Update submodules ────────────────────────────────────────────────────────
git submodule update --remote        # update all submodules to their remote latest
git submodule update --remote src/shared  # update specific submodule

# ── Common operations ─────────────────────────────────────────────────────────
git submodule status                 # see current commit of each submodule
git submodule foreach git pull       # pull in every submodule
git submodule foreach git status     # run any command in every submodule

# ── Remove a submodule ───────────────────────────────────────────────────────
git submodule deinit src/shared      # unregister submodule
git rm src/shared                    # remove from working tree and index
rm -rf .git/modules/src/shared       # remove cached module data
git commit -m "chore: remove shared submodule"
```

---

## 14. Hooks — Automate Git Events

Hooks are scripts that run automatically at specific Git events.

```bash
# Hooks live in: .git/hooks/
# To enable, remove .sample extension and make executable: chmod +x

# ── Most useful hooks ────────────────────────────────────────────────────────

# pre-commit: runs before git commit (run tests, lint, format)
# .git/hooks/pre-commit
#!/bin/bash
npx lint-staged           # runs linting on staged files only
exit $?                   # 0 = pass, non-zero = abort commit

# commit-msg: validate commit message format
# .git/hooks/commit-msg
#!/bin/bash
COMMIT_MSG=$(cat "$1")
PATTERN="^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .{1,72}$"
if ! echo "$COMMIT_MSG" | grep -qP "$PATTERN"; then
    echo "❌ Commit message does not follow conventional commits format"
    echo "   Expected: feat(scope): description"
    exit 1
fi

# pre-push: run tests before pushing
# .git/hooks/pre-push
#!/bin/bash
npm test
if [ $? -ne 0 ]; then
    echo "❌ Tests failed — push aborted"
    exit 1
fi
echo "✅ Tests passed — pushing"

# ── Share hooks with the team (Husky — Node.js projects) ─────────────────────
npm install --save-dev husky
npx husky init
# Creates .husky/ directory (committed to repo — shared with team!)
# .husky/pre-commit:
npx lint-staged
# .husky/commit-msg:
npx commitlint --edit $1
```

---

## 15. Real Scenarios — "What Do I Do?"

### 😰 Scenario 1: "I committed to main by accident"

```
You meant to create a new branch, but you committed directly to main.
The commit is not pushed yet.

Before:
  main: A ─ B ─ C ─ D  (D = your accidental commit, not pushed)

Step 1: Create the branch you meant to use
  git branch feature/login
  (this creates feature/login pointing to D, which has your changes)

Step 2: Undo the commit on main
  git reset --hard HEAD~1
  (moves main back to C, D is now only on feature/login)

After:
  main:          A ─ B ─ C   ← clean
  feature/login: A ─ B ─ C ─ D  ← your work is safe here

Step 3: Switch to your new branch
  git checkout feature/login
  → Your work is there, exactly where it should be.
```

### 😱 Scenario 2: "I pushed bad code to main"

```
You pushed a broken commit to main. Others might have pulled it.
You CANNOT use reset (would rewrite pushed history).

The ONLY safe fix: git revert

git revert HEAD              # creates a new commit that undoes the last commit
git push origin main

Before:      A ─ B ─ C  (C = the bad commit, pushed)
After:       A ─ B ─ C ─ C'  (C' = "Revert C", pushed)

✅ Everyone who has C can pull C' without any issues.
The fix is visible in history — no mysteries.

If the bad commit was N commits ago:
  git revert abc123           # revert that specific commit
  # Resolve any conflicts (the code since abc123 might depend on it)
  git push origin main
```

### 🤦 Scenario 3: "I need to undo a merge that was already pushed"

```
A bad feature was merged into main via a merge commit.
main: A ─ B ─ C ─ M  (M = merge commit, bad feature is in M)

Revert the merge commit:
  git revert -m 1 M_hash

  -m 1 = "mainline parent 1" — tells git which side of the merge to revert TO
  (parent 1 = the main branch before the merge, parent 2 = the feature branch)

This creates:
  main: A ─ B ─ C ─ M ─ M'
  M' undoes everything that M introduced.

⚠️ IMPORTANT: If you later want to re-merge the feature (after fixing it),
   you must ALSO revert M' first, or the feature's commits will be invisible to git
   (git sees them as already merged via M, even though M was reverted).
   git revert M'_hash   # undo the revert of the merge
   git merge feature/fixed  # now merge the fixed feature
```

### 😤 Scenario 4: "My feature branch is way behind main and has horrible conflicts"

```
Your branch diverged from main 3 weeks ago.
main has 50 new commits. Your rebase is a nightmare of conflicts.

Situation:
  main:    A ─ B ─ ... ─ (50 commits) ─ Z
  feature: A ─ B ─ C ─ D ─ E  (your 3 commits from 3 weeks ago)

Option 1: Rebase (clean but painful for many conflicts)
  git checkout feature/my-branch
  git rebase main
  # Fix conflicts at each commit D, E...
  # 50 commits worth of changes to integrate one by one

Option 2: Create a new branch from main and cherry-pick
  git checkout main
  git checkout -b feature/my-branch-v2
  git cherry-pick C D E   # cherry-pick just your 3 commits
  # Fix conflicts ONCE (in one cherry-pick if needed)
  # Much less painful for a small number of feature commits

Option 3: Merge (preserves history, merge conflicts all at once)
  git checkout feature/my-branch
  git merge main
  # Fix all conflicts ONCE (one merge conflict resolution)
  # Creates a merge commit but your branch is now up to date
  # Best when: many conflicts, prefer to resolve once and move on
```

### 😩 Scenario 5: "I accidentally deleted a branch"

```
git branch -D feature/payment   # whoops — force deleted without merging

But the commits still exist in git's object store for 90 days!
Git reflog is your hero:

git reflog
# HEAD@{0}: checkout: moving from feature/payment to main
# HEAD@{1}: commit: feat: add Fawry integration
# HEAD@{2}: commit: feat: add payment gateway abstraction
# ...

Find the SHA of the last commit on the deleted branch:
git checkout -b feature/payment HEAD@{1}
# Or: git branch feature/payment abc123  (the SHA from reflog)

Branch restored! All commits are back.
```

### 🤯 Scenario 6: "I did git reset --hard and lost hours of work"

```
git reset --hard HEAD~5   # you meant HEAD~1... but typed 5
5 commits of work gone from your branch!

But reflog saves you:
git reflog

# 0: HEAD@{0}: reset: moving to HEAD~5
# 1: HEAD@{1}: commit: fix payment edge case
# 2: HEAD@{2}: commit: add Stripe webhook handler
# ...the commits that disappeared are listed here

git reset --hard HEAD@{1}  # go back to where you were before the reset!

Or cherry-pick specific commits back:
git cherry-pick HEAD@{2} HEAD@{3} HEAD@{4} HEAD@{5}

⏰ This ONLY works if you do it within 90 days (default reflog expiry).
```

### 😰 Scenario 7: "I made commits with the wrong email address"

```
You committed 10 commits with work@company.com but should have used ahmed@kickoff.dev.
These commits are NOT pushed yet.

Fix the last 10 commits:
git rebase -i HEAD~10

In the editor, change all "pick" to "exec git commit --amend --reset-author --no-edit":
  exec git commit --amend --author="Ahmed Ali <ahmed@kickoff.dev>" --no-edit
  pick abc123 fix payment
  exec git commit --amend --author="Ahmed Ali <ahmed@kickoff.dev>" --no-edit
  ...

OR — quicker for all commits from a point:
git rebase --exec 'git commit --amend --reset-author --no-edit' HEAD~10

After: verify with git log --format="%an <%ae>" -10
```

### 😵 Scenario 8: "A colleague rebased a shared branch and now my git pull fails"

```
Your colleague ran git rebase main on a shared branch and force-pushed.
Now your local copy and the remote have diverged.

git status shows:
  "Your branch and 'origin/feature/payment' have diverged,
  and have 5 and 8 different commits each."

WRONG approach: git pull  (creates a messy merge of the rebased + old commits)

RIGHT approach:
  git fetch origin
  git reset --hard origin/feature/payment

  # This throws away your LOCAL commits and matches the remote exactly.
  # ⚠️ Only safe if you haven't added NEW commits on top of the old ones.

If you DID add new commits on top:
  git fetch origin
  git rebase origin/feature/payment
  # Replays YOUR new commits on top of the new remote history

LESSON: Never force-push to shared branches without team coordination.
Always use: git push --force-with-lease (safer — checks for others' commits first)
```

### 😬 Scenario 9: "I committed a secret/password and pushed it"

```
Oh no — you committed .env with real API keys and pushed to GitHub.

IMMEDIATE STEPS:
1. Revoke the exposed credential IMMEDIATELY (rotate the API key, change password)
   This is step 1 — do it before anything else. GitHub scanners and bots see pushes
   within seconds.

2. Remove from git history:
   # Modern approach (git 2.24+):
   git filter-repo --path .env --invert-paths
   # This rewrites ALL history — every commit hash changes

   # Alternative — BFG Repo Cleaner (faster for large repos):
   # Download bfg.jar from https://rtyley.github.io/bfg-repo-cleaner/
   java -jar bfg.jar --delete-files .env
   git reflog expire --expire=now --all
   git gc --prune=now --aggressive

3. Force-push ALL branches:
   git push --force --all
   git push --force --tags

4. Everyone on the team must re-clone:
   The old commits existed on their machines — they need fresh clones.

5. Add to .gitignore (should have been there from the start):
   echo ".env" >> .gitignore
   git add .gitignore
   git commit -m "chore: add .env to gitignore"

⚠️ If the repo is public, assume the secret is compromised regardless.
   GitHub's secret scanning may have already detected it.
   Rotating the credential is the only safe action.
```

### 🔥 Scenario 10: "Git rebase has conflicts and it's a mess"

```
You're rebasing feature/payment onto main.
main has 20 new commits. Your branch has 5 commits.
Conflict after conflict after conflict.

Understanding what's happening:
  Git replays your commits ONE BY ONE on top of main.
  If commit D conflicts with main's changes, you fix it.
  Then commit E might ALSO conflict (because D's changes shifted).

Strategy A: Power through it
  git rebase main
  # Conflict in commit D:
  # Edit the files to resolve
  git add .
  git rebase --continue
  # Conflict in commit E:
  # Edit the files to resolve
  git add .
  git rebase --continue
  # (repeat until done)

Strategy B: Abort and merge instead
  git rebase --abort    # back to original state, nothing broken
  git merge main        # merge all conflicts at once
  # Resolve all conflicts in one shot
  git add .
  git merge --continue
  # Single merge commit — but you're done

Strategy C: Interactive rebase to squash first
  git rebase -i HEAD~5   # squash your 5 commits into 1
  # Then:
  git rebase main        # only ONE commit to replay, ONE conflict to resolve
  # Cleanest result if you don't need granular feature history

Strategy D: Rerere (Reuse Recorded Resolution)
  git config rerere.enabled true    # enable this globally
  # Git remembers how you resolved conflicts.
  # If the same conflict comes up again (during rebase, merge, etc.)
  # it automatically applies your previous resolution.
```

---

## 16. Interview Questions & Answers

**Q: What is the difference between git merge and git rebase?**

Both integrate changes from one branch into another, but they do it differently and leave different histories. Merge creates a new "merge commit" with two parents — it preserves the exact historical context of when and how branches diverged and were integrated. Rebase replays your commits on top of the target branch as if you had started your branch from that point — it creates new commit hashes and produces a linear history. The golden rule: never rebase commits that have been pushed to a shared branch, because it rewrites history and causes problems for anyone who has the original commits. Use rebase to clean up local feature branches before PR review; use merge to integrate features into shared branches.

---

**Q: What does git reset --soft, --mixed, and --hard do?**

All three move the HEAD (and current branch pointer) to a different commit. The difference is what happens to the files. `--soft` moves the HEAD but leaves all changes staged in the index — you can immediately re-commit them differently. `--mixed` (the default) moves the HEAD and unstages the changes — they're still in your working tree but you need to re-add them. `--hard` moves the HEAD AND discards all changes in both the staging area and working tree — the files go back to the state of the target commit and any uncommitted changes are permanently lost.

---

**Q: What is the difference between git fetch and git pull?**

`git fetch` downloads commits, files, and references from the remote repository and updates your remote-tracking branches (like `origin/main`), but it does NOT change your local working branch or working tree. You can inspect the fetched changes before integrating. `git pull` is essentially `git fetch` followed immediately by `git merge` (or `git rebase` if configured). Fetch is safer when you want to review what changed before integrating. Many developers prefer `git fetch` + `git merge`/`git rebase` manually for the extra control.

---

**Q: When would you use git revert instead of git reset?**

Use `git revert` when the commit you want to undo has already been pushed to a shared remote branch. Revert creates a NEW commit that introduces the inverse of the unwanted commit — it doesn't erase history. This is safe because other developers who have pulled the bad commit can simply pull the revert commit. Use `git reset` only for commits that exist solely on your local machine and haven't been pushed anywhere — it rewrites history, which causes conflicts for anyone who has the original commits.

---

**Q: What is a detached HEAD state?**

Normally, HEAD points to a branch name (e.g., HEAD → main). In detached HEAD state, HEAD points directly to a commit hash instead of a branch. This happens when you `git checkout` a specific commit hash, a tag, or a remote-tracking branch. In this state you can make commits, but they won't be attached to any branch. When you switch away, those commits become "floating" — no branch points to them, and git's garbage collector will eventually delete them. To save work done in detached HEAD, create a branch before leaving: `git checkout -b rescue-branch`.

---

**Q: What does git cherry-pick do and when would you use it?**

Cherry-pick takes a specific commit from anywhere in the history and applies its changes as a new commit on top of your current branch. The original commit remains where it is; you get a copy with a new hash. Use cases: a critical bug fix that was committed to a development branch and urgently needs to go into the production branch without bringing all of development along; backporting a feature to an older version; saving a useful commit from a branch you're about to delete. Avoid overusing it — if you cherry-pick the same commit into multiple branches, you'll have duplicate commits that can cause confusing conflicts when the branches eventually merge.

---

**Q: What is git bisect and when is it useful?**

`git bisect` uses binary search to find the specific commit that introduced a bug. You mark a "bad" commit (current broken state) and a "good" commit (known working state), and git checks out the midpoint. You test and mark it good or bad, and git keeps halving the remaining range. For 1,000 commits, you find the culprit in about 10 steps. It's most powerful when combined with `git bisect run` and a test script — git can fully automate the binary search without manual intervention. It's invaluable when you know something was working in v1.0 but is broken now, and the git log has hundreds of commits between them.

---

**Q: What's the difference between a fast-forward merge and a merge commit?**

A fast-forward merge happens when the branch being merged into hasn't had any new commits since the divergence — the branch pointer can simply move forward to the tip of the feature branch without creating a new commit. The history stays linear. A merge commit (also called a non-fast-forward merge or 3-way merge) creates a new commit with two parents when both branches have new commits since they diverged. Teams often use `git merge --no-ff` to always create a merge commit even when fast-forward is possible — this preserves a record of when a feature branch was integrated.

---

**Q: What is git reflog and why is it useful?**

The reflog (reference log) is git's "undo history for git operations themselves." It records every time HEAD or a branch tip moved — including commits, checkouts, resets, rebases, and merges. Unlike regular git log which only shows committed history, reflog shows everything git tracked about your local movements, even if you deleted branches or reset commits. It's your safety net when you think you've lost work: commits that "disappeared" from branches still exist in git's object store for 90 days and can be recovered via reflog by finding their hash and creating a branch pointing to them.

---

## 📖 Resources

- [Pro Git Book (free)](https://git-scm.com/book/en/v2) ← the definitive reference
- [Oh Shit, Git! (practical rescue)](https://ohshitgit.com/) ← relatable, practical
- [Learn Git Branching (visual, interactive)](https://learngitbranching.js.org/) ← best visual tool
- [Atlassian Git Tutorials](https://www.atlassian.com/git/tutorials) ← clear, comprehensive
- [Git Flight Rules](https://github.com/k88hudson/git-flight-rules) ← "I did X, what now?"
- [Conventional Commits spec](https://www.conventionalcommits.org/) ← commit message standard
- [git-filter-repo](https://github.com/newren/git-filter-repo) ← modern history rewriting

---

*Git is not about memorizing commands — it's about understanding the mental model.
Once you know that branches are just pointers, HEAD is just a pointer, and commits are immutable,
every command makes sense. Go ship great code, Ahmed. 🌿*

> **Legend:**
> ✅ = safe / recommended | ❌ = dangerous / avoid | ⚠️ = caution needed
> A ─ B ─ C = commit history (A is oldest, C is newest)
> HEAD → branch → commit = the pointer chain
