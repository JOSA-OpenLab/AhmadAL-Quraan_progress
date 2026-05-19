# Git Internals Explained Like You're 5

Imagine you have a magic toy box.  
Every time you draw something new, the toy box remembers it forever.  
Git works almost like that magic toy box.

---

## 1. Git Saves Snapshots

Git does not save only changes.  
It takes **pictures (snapshots)** of your project again and again.

Like taking photos of your LEGO castle every time you build something new.

---

## 2. Git Uses Secret Names

Every file gets a special secret name called a **HASH**.

If the file changes even a little bit, the secret name changes too.

---

# There are 4 main objects this magic box uses :

## 3. Blobs = File Contents

Git puts your file contents into little boxes called **BLOBS**.

A blob only cares about what is inside the file, not the filename.

---

## 4. Trees = Folders

A **TREE** tells Git how folders are organized.

It says things like:

- "This file goes here"
- "This folder goes there"

---

## 5. Commits = Save Points

A **COMMIT** is like pressing **SAVE** in a video game.

It remembers the whole project at that moment.

---

## 6. Branches = Different Paths

Branches are like drawing different endings to the same story.

You can experiment without breaking your main work.

---

## 7. Git is Very Smart

If two files are exactly the same, Git stores them only once.

This saves space and makes Git fast.

---

## 8. Why People Love Git

People love Git because it:

- Remembers everything
- Helps teams work together
- Lets programmers travel back in time when they make mistakes

---

# Final Idea

Git is basically a giant memory machine for code.

It remembers every version of your project using special content names.
