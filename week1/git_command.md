# Git Cheat Sheet (Organized)

## 1. Check Current Branches
Show all local branches:

```bash
git branch
```

---

# 2. Switching Branches

### Switch to an existing branch

```bash
git checkout <branch-name>
```

or using the newer command:

```bash
git switch <branch-name>
```

---

### Create and switch to a new branch

Using `checkout`:

```bash
git checkout -b <branch-name>
```

Using `switch`:

```bash
git switch -c <branch-name>
```

---

# 3. Stage Changes

### Add a specific file

```bash
git add <file-name>
```

### Add all changes

```bash
git add .
```

---

# 4. Commit Changes

Save staged changes with a message:

```bash
git commit -m "Your commit message"
```

---

# 5. Push Changes

### Push to a specific branch

```bash
git push -u origin <branch-name>
```

### If the branch does not exist remotely yet

```bash
git push --set-upstream origin <branch-name>
```

---

# 6. Remote Repository Management

## Check the current remote URL

```bash
git remote -v
```

---

## Change or set the remote repository URL

```bash
git remote set-url origin git@github.com:ZeroNeroIV/Code-Search-Engine.git
```

Verify the change:

```bash
git remote -v
```

---

# 7. Remove a File from Git Tracking

Stop tracking a file while keeping it locally:

```bash
git rm --cached <file-name>
```

Useful when:
- You accidentally uploaded a file
- You later add it to `.gitignore`

---

# 8. Git Configuration

## Set your email globally

```bash
git config --global user.email "your-email@example.com"
```

---

# 9. Undo the Most Recent Commit

## Keep file changes locally

```bash
git reset --soft HEAD~1
```

---

## Remove the commit and file changes completely

```bash
git reset --hard HEAD~1
```

⚠️ Warning: `--hard` permanently deletes uncommitted changes.

---

# 10. View an Older Commit

Show detailed information about a commit:

```bash
git show <commit-id>
```

---

# 11. Go Back to a Previous Commit

## Case 1 — Temporarily View an Old Commit (Safe)

```bash
git checkout <commit-hash>
```

- Does not change history
- Enters detached HEAD mode
- Useful for inspecting old code

---

## Case 2 — Revert Changes While Keeping History (Recommended)

Creates a new commit that undoes changes:

```bash
git revert <commit-hash>
```

Recommended if the commit was already pushed to GitHub.

---

## Case 3 — Hard Reset to an Older Commit

Completely removes newer commits and changes:

```bash
git reset --hard HEAD~<number-of-commits>
```

Example:

```bash
git reset --hard HEAD~2
```

⚠️ Warning:
- This rewrites history
- Can permanently delete commits
- Be careful if working with others


## See hash for revisions (Pointers or objects)

```
git rev-parse HEAD 
```

## Content for any objects 
```
git cat-file <hash> 
```

* `t`: type for the object. 
* `p`: print the content for the object.

## Recover yourself (log)
```
git reflog show
```
* Will show you all the logs and moves (hashes) you did in your repo (specifically what happened with HEAD), and you can use to recover from any mistake you did. 

> You can use `git switch -C <branch>` to any branch including main points to HEAD, useful when deleting content and recovering.

### What if you delete a file
* `rm <file>.txt`
```
git restore .
```
* Restore from a specific commit.
```
git restore --source <hash_file> <file_name_deleted> 
```


## Merging approaches

### Merge 
```
git switch main
git fetch origin 
git merge origin/main
```
* When the branches have diverged, Git creates a new merge commit that has both branches as parents.
```
Before: 

A---B---C main
     \
      D---E feature

After: 
A---B---C--------M main
     \          /
      D---E---- feature
```

### Rebase 
```
git switch feature_branch
git rebase <target branch(top of it)>
git rebase main

```
* Advanced merge, instead of making a new commit, you take a whole commits of a branch (feature branch), and add them in-front of the main one, so the history of it will be linear. 
* This is very useful if u want to make several features for a main branch without the main history being affected.

```
Before: 

A---B---C main
     \
      D---E feature
After: 
A---B---C---D'---E' feature
```
### Squash 
`git rebase -i`

* An interactive window will appears after.

| Command  | Meaning                             |
| -------- | ----------------------------------- |
| `pick`   | keep commit normally                |
| `squash` | combine with previous commit        |
| `fixup`  | combine and discard message         |
| `reword` | keep commit but edit message        |
| `drop`   | remove commit                       |
| `edit`   | stop during rebase to modify commit |
* squash combines multiple commits into a single commit.
* It’s commonly used to clean up a feature branch before merging it into main.

==> After editing another window will appear for commit message.
```
Suppose your feature branch has many small commits:
A---B---C main
         \
          D---E---F feature
Where:

D = initial feature
E = fix typo
F = debugging changes

You may not want all these commits in the main history.

git checkout main
git merge --squash feature
git commit

Result: 
A---B---C---S main
```


| Operation | What happens                             |
| --------- | ---------------------------------------- |
| Merge     | Combines histories with a merge commit   |
| Rebase    | Replays commits on top of another branch |
| Squash    | Combines multiple commits into one       |



### Squash merging 

* Squash and merge into main directly.
```
git merge --squash feature
git commit
```

Result: 
```
A---B---C---S
```

It’s basically:

“Take all changes from this branch and make them one commit.”
