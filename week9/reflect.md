## [Task1](https://github.com/AhmadAL-Quraan/Inception/tree/main), [Task2](https://github.com/AhmadAL-Quraan/Inception/tree/main), [Task3](https://github.com/AhmadAL-Quraan/Inception/blob/main/CONTRIBUTING.md), [Task4](https://github.com/AhmadAL-Quraan/Inception/blob/main/GOVERNANCE.md)

Why choosing it MIT License ?
- Lowest friction for reuse, someone else at 42 (or anywhere) can copy a Dockerfile or compose pattern into their own submission without worrying about license compatibility.
- No copyleft obligations. GPL/AGPL would be the wrong tool here, there's no standalone software identity to protect, and copyleft would just be a barrier for people using this as a learning reference.
- Matches norms in the 42 community, nearly every public 42 project repo I've seen (including the one I cited in my own README) uses MIT.
- Requires attribution but nothing else, keeps my name on the work without adding legal overhead disproportionate to what's actually here (mostly config, not novel algorithms).


**Task2: Achieve 100% GitHub Community Standards on a personal repo**

Completed the full GitHub Community Standards checklist for the `Inception` repo (42 project, Docker Compose stack for NGINX, WordPress/PHP-FPM, and MariaDB). Added every file GitHub's checklist requires that wasn't already present:

- `LICENSE` MIT (see prior journal entry for justification)
- `CODE_OF_CONDUCT.md` Contributor Covenant v2.1
- `CONTRIBUTING.md` contribution workflow, build/test steps, code style notes
- `SECURITY.md` vulnerability reporting process
- `.github/ISSUE_TEMPLATE/bug_report.md` and `feature_request.md`
- `.github/PULL_REQUEST_TEMPLATE.md`

README and repo description were already in place. Pushed all files to `main`, then verified via **Insights → Community Standards** that the checklist shows 100% (all green). Screenshot captured for the journal.


![[community standard.png]]





## [Task5](https://github.com/gruns/icecream/pull/249)


**The gap I found:**
The README's `argToStringFunction` section shows a nice example of registering a custom formatter for `numpy.ndarray`, but there's no equivalent example for the very common case of **pandas DataFrames**, which come up constantly in the same debugging context `ic()` targets (data science / numerical code). It's a natural doc gap: the library already supports this via `singledispatch`, it's just not documented for the second-most-common data type after numpy arrays.