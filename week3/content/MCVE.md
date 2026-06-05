
# MCVE (Minimal, Complete, Verifiable Example)

## What is an MCVE?

MCVE stands for **Minimal, Complete, and Verifiable Example**. It is a small, self-contained piece of code that demonstrates a problem clearly and allows others to reproduce it easily.

Providing an MCVE is considered a best practice when contributing to open-source projects, reporting bugs, or asking for help on developer forums.

---

## Why Use an MCVE?

An MCVE helps maintainers and contributors:

* Understand the issue quickly.
* Reproduce the problem without extra setup.
* Focus on the actual bug instead of unrelated code.
* Provide faster and more accurate assistance.

---

## MCVE Principles

### 1. Minimal

Include only the code necessary to reproduce the problem.

**Bad:**

* Thousands of lines of code.
* Multiple unrelated classes.
* Unnecessary dependencies.

**Good:**

* A few lines of code that trigger the bug.

### 2. Complete

Provide everything needed to run the example.

**Include:**

* Required imports.
* Necessary classes and methods.
* Relevant configuration information.
* Environment details when applicable.

### 3. Verifiable

Anyone should be able to run the example and observe the same issue.

Provide:

* Steps to reproduce.
* Expected behavior.
* Actual behavior.

---

## Example: Bug Report Without an MCVE

### Report

> The application crashes when I try to save a file. Please fix it.

### Problems

* No code provided.
* No reproduction steps.
* No information about the environment.
* Maintainers cannot verify the issue.

---

## Example: Bug Report With an MCVE

### Problem

Saving a file with an empty filename causes the application to crash.

### Minimal Code

```java
public class Main {
    public static void main(String[] args) {
        FileManager fm = new FileManager();
        fm.save(""); // Crash happens here
    }
}
```

### Steps to Reproduce

1. Create a `FileManager` object.
2. Call `save("")`.
3. Run the program.

### Expected Behavior

```text
Error: filename cannot be empty
```

### Actual Behavior

```text
Exception in thread "main" java.lang.NullPointerException
```

### Environment

* Java 21
* Debian 13

---

## Why This Example Is an MCVE

| Criterion  | Explanation                                                        |
| ---------- | ------------------------------------------------------------------ |
| Minimal    | Contains only the code required to reproduce the bug.              |
| Complete   | Includes all information needed to understand and run the example. |
| Verifiable | Running the code reproduces the issue consistently.                |

---

## Tips for Contributors

Before submitting a bug report or asking for help:

* Remove unrelated code.
* Use meaningful variable names.
* Include exact error messages.
* Provide reproduction steps.
* Mention your operating system and software versions.
* Verify that the issue can be reproduced from the example alone.

