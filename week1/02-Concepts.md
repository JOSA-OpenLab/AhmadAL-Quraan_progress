# Concepts


## 1. What is git 
* A version control.

## 2. The Four Main Areas of Git

### 1) Working Directory
Your actual files on disk.

### 2) Staging Area (Index)
A preparation area where you select changes before committing.

### 3) Local Repository
The database of commits stored on your machine.

### 4) Remote Repository
A shared repository hosted on GitHub, GitLab, etc.

![](../pic/git_workflow.jpg)
---

## 3. Basic Git Flow

### `git add`
Moves changes from the working directory into the staging area.

### `git commit`
Creates a snapshot from the staged changes.

### `git push`
Uploads commits to the remote repository.

### `git fetch`
Downloads remote changes without merging.

### `git pull`
Equivalent to fetch + merge.

### `git checkout`
Switches branches or restores files.

---

## 4. The Real Core of Git

Git is not just a backup tool.

Git is actually a **content-addressed versioned graph database**.

The most important concept:

> Commits form a Directed Acyclic Graph (DAG).

---

## 5. Commit Graph Example

```text
A --- B --- C main
 \
  D --- E feature
```

Each commit points to parent commits.

Branches are simply movable labels pointing to commits.

---

## 6. Understanding HEAD and Branches

HEAD points to the current branch or commit.

Example:

```text
main -> C
HEAD -> main
```

After another commit:

```text
main -> D
HEAD -> main
```

---

## 7. The Staging Area 

The staging area is not just temporary storage.

You can:

- Stage parts of files
- Stage specific hunks
- Create clean commits

Useful command:

```bash
git add -p
```

---

## 8. Intermediate Git Topics

Important topics after the basics:

- Branching and merging
- Fast-forward merges
- Merge conflicts
- Remote tracking branches
- Undoing changes
- Viewing history with graphs

---

## 9. Rebase

```bash
git rebase main
```

Rebase rewrites commit history by moving commits onto another base commit.

This creates a cleaner linear history.

---

## 10. Interactive Rebase

```bash
git rebase -i HEAD~5
```

Allows:

- Squashing commits
- Renaming commits
- Reordering commits
- Splitting commits

---

## 11. Reset Internals

Understanding reset requires understanding:

- HEAD
- Index (staging area)
- Working tree

Commands:

```bash
git reset --soft
git reset --mixed
git reset --hard
```

---

## 12. Git Internals

Git stores four major object types:

- Blob
- Tree
- Commit
- Tag

All objects are identified using SHA hashes.

---

## 13. Advanced Git Commands

### `git reflog`
Recover lost commits.

### `git bisect`
Find which commit introduced a bug.

### `git worktree`
Use multiple working directories.

### `git stash`
Temporarily save unfinished work.

### `git blame`
See who changed each line.

---

## 14. A Very Useful Visualization Command

```bash
git log --graph --decorate --oneline --all
```

This helps visualize the commit graph directly.

