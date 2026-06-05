# Git Bisect - Complete Guide with Examples

## What is Git Bisect?

Git bisect is a binary-search tool built into Git that helps you find the exact commit that introduced a bug.

Instead of manually checking every commit, Git repeatedly chooses the middle commit between a known good commit and a known bad commit, reducing the number of tests dramatically.

---

## Why Use Git Bisect?

If you have 100 commits between a working version and a broken version, checking commits one-by-one could require 100 tests.

Git bisect uses binary search and can often find the offending commit in about:

```text
log2(100) ≈ 7 tests
```

---

## Basic Workflow

1. Start bisect:

```bash
git bisect start
```

2. Mark current commit as bad:

```bash
git bisect bad
```

3. Mark a known working commit:

```bash
git bisect good <commit-hash>
```

4. Test commits selected by Git.

5. Mark each tested commit as good or bad:

```bash
git bisect good
```

or

```bash
git bisect bad
```

6. Git identifies the first bad commit.

7. End the session:

```bash
git bisect reset
```

---

## Example Repository History

Imagine the following commit history:

```text
fed4567  Latest commit (BAD)
89ab123  More changes (BAD)
cd1d4c4  Added file2 and file3 (BUG INTRODUCED)
377fd8e  Added file5 (GOOD)
a3e9a53  Initial project (GOOD)
```

The bug first appears in commit:

```text
cd1d4c4
```

---

## Example Session

Start bisecting:

```bash
git bisect start
git bisect bad
git bisect good a3e9a53
```

Git checks out:

```text
377fd8e
```

You test the code and determine it works:

```bash
git bisect good
```

Git responds:

```text
cd1d4c4f3a83f40fb20da6ad13000ea6e539c3fe is the first bad commit
```

---

## Expected Git Log

```bash
git log --oneline
```

Output:

```text
fed4567 Latest changes
89ab123 More changes
cd1d4c4 Totall edit, fixing and making it beautiful
377fd8e file5
a3e9a53 Initial commit
```

---

## Understanding the Result

When you marked `377fd8e` as good, Git knew:

- `a3e9a53` is good
- `377fd8e` is good
- A later commit is bad

Therefore, the next commit after `377fd8e`:

```text
cd1d4c4
```

must be the first commit that introduced the bug.

Visualization:

```text
a3e9a53  GOOD
   |
377fd8e  GOOD
   |
cd1d4c4  BAD  <-- First bad commit
   |
89ab123  BAD
   |
fed4567  BAD
```

---

## Useful Commands

Show bisect decisions:

```bash
git bisect log
```

Visualize commits:

```bash
git bisect visualize
```

Return to original branch:

```bash
git bisect reset
```

Inspect a commit:

```bash
git show <commit-hash>
```

Compare two commits:

```bash
git diff <commitA> <commitB>
```

---

## Automating Tests

Git can run tests automatically during bisect.

Example:

```bash
git bisect start
git bisect bad
git bisect good a3e9a53
git bisect run ./test_script.sh
```

Your script should return:

```text
0         => Good commit
Non-zero  => Bad commit
```

Git will automatically continue testing until it finds the first bad commit.

---

## Example Test Script

### Bash

```bash
#!/bin/bash

npm test

if [ $? -eq 0 ]; then
    exit 0
else
    exit 1
fi
```

Run:

```bash
git bisect run ./test.sh
```

---

## Common Mistakes

### 1. Marking the Wrong Commit as Good

If a commit actually contains the bug but is marked as good, the bisect result will be incorrect.

### 2. Testing with Uncommitted Changes

Always ensure your working tree is clean:

```bash
git status
```

### 3. Forgetting to Reset

After bisecting:

```bash
git bisect reset
```

Otherwise you'll remain in a detached HEAD state.

### 4. Assuming the Latest Bad Commit Is the First Bad Commit

The latest bad commit only tells Git where the bug exists now.

Bisect helps locate where it first appeared.

---

## Complexity Analysis

### Linear Search

Checking every commit manually:

```text
100 commits -> up to 100 tests
1000 commits -> up to 1000 tests
```

### Git Bisect (Binary Search)

```text
100 commits  -> ~7 tests
1000 commits -> ~10 tests
10000 commits -> ~14 tests
```

Formula:

```text
Tests ≈ log2(number_of_commits)
```

---

## Summary

Git bisect uses binary search to efficiently locate the exact commit that introduced a bug.

In the example:

```text
GOOD:
a3e9a53
377fd8e

BAD:
cd1d4c4
89ab123
fed4567
```

Git correctly identified:

```text
cd1d4c4f3a83f40fb20da6ad13000ea6e539c3fe
```

as the first bad commit because it was the first commit after the last confirmed good commit (`377fd8e`).

---

## Final Cleanup

After finishing:

```bash
git bisect reset
```

Git returns you to the branch and commit where you started the bisect session.
