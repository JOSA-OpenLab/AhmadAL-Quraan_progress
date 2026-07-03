# Code Review & the Maintainer Mindset

## Task1 

**Find three open PRs. Leave substantive comments on each, using Conventional Comments labels. Substantive means design or correctness, not formatting.**

- First [PR](https://github.com/celery/celery/pull/10359): I think this PR is was little bit of a rush given.

- Second [PR](https://github.com/httpie/cli/pull/1883):
 The requests library (which httpie uses under the hood) automatically un-squishes gzip/deflate files for you as they download. So by the time httpie counts the bytes it received, it's counting the un-squished, much bigger version, maybe 10,000 bytes The old, buggy code compared:

  That produced the real-world bug people reported: `http --download` claiming a perfectly good gzip download was "incomplete," when it wasn't.

  This PR exists to stop httpie from wrongly comparing compressed-size to decompressed-size and crying wolf about a fine download.
My comment, in that context, was flagging: "your 'skip the check' rule fires for any compression type — but for one specific type (br), the un-squishing might not actually happen if a tool is missing, so in that one case, skipping the check throws away a comparison that would've actually been valid and useful."    
That's it — it's not "this is definitely broken," it's "here's a specific case where I think it might quietly stop catching a real problem, can you verify?"



## Task 2 

**Self-review write-up. Take one of your own past PRs (the one you’re least proud of).
Pretend you’re the reviewer. Write what you would have said. Write about it in your weekly
journal.**


* As my peers did, I decided as well to review the same [PR](https://github.com/wasmerio/Python-Scripts/pull/564#pullrequestreview-4598634725) I did for JOSA. 
* Mistakes I mentioned: 

   1) A real bug I didn't notice was what if we still had website lists from a previous sessions, `self.website` just remove websites from the current session.
   2) `working_hours` logic wasn't give option for users to set for weeks or months, or no specific timezones.
   3) block_websites checks if entry not in content, which guards against duplicate lines within a single file, but it appends without ever sorting or namespacing the block, so the hosts file accumulates whatever ordering happens to occur from prior runs with different site lists. 


## Task 3 

* I think I'm currently (technically) not in a good position to know if a design/code is over-engineered or not, I'm still learning. I am gonna come back to this task later on .

## Task4

* As someone who currently study Golang, I wanted a huge project which made in it for this task and since there are so many projects, I decided to go with one of the main tools used in DevOps for monitoring "Prometheus".

* This task was so long and tough for me, but I learned a lot from it, big shout to ChatGpt and GO community on **reddit**.

### Code Review Norms in a Go Project: Prometheus

[prometheus/prometheus](https://github.com/prometheus/prometheus), the Go-based monitoring system and time-series database, one of the most actively reviewed CNCF projects written in Go, maintained by a small, opinionated group of maintainers.

**Method:** Read the full discussion threads on several merged pull requests (a major architectural rewrite, a routine release PR, and supporting context on the project's review conventions), then extracted patterns in what reviewers flag, what they let slide, and how disagreements got resolved.

---

## Why Prometheus

* I thought of reading Golang codebase itself but Prometheus was chosen instead of golang/go itself for one important reason: **golang/go does not actually run its code review on GitHub.** Contributions arrive as GitHub PRs, but a bot (GerritBot) immediately imports them into Gerrit, closes the GitHub thread, and posts a link to the real review at `go-review.googlesource.com`. All substantive review — comments, iteration, approvals — happens on Gerrit, not GitHub. See:
- [golang/go Issue #18517 – "codereview: accept GitHub PRs"](https://github.com/golang/go/issues/18517) — the original proposal to bridge GitHub PRs into Gerrit.
- [golang/go Discussion #61182 – "should the Go project stop importing GitHub PRs?"](https://github.com/golang/go/discussions/61182) — a 2023 re-litigation of that decision, useful reading on why the Go team still prefers Gerrit's tooling (inline patchsets, mandatory "publish drafts" step, stricter reviewer/owner model) over GitHub's native review UI.
- [Go Wiki: GerritBot](https://go.dev/wiki/GerritBot) — mechanics of the GitHub → Gerrit bridge.
- Example of a real PR being auto-converted: [golang/go#43450](https://github.com/golang/go/pull/43450) and [golang/go#24123](https://github.com/golang/go/pull/24123) — both show the bot's boilerplate message and the PR auto-closing once the Gerrit CL merges.

Since the goal was to read actual *discussion threads* on merged PRs, Prometheus is a better fit: it's Go, it has a genuinely strict review culture, and all review happens natively on GitHub, which makes the norms directly legible.

---

## PR #1: [prometheus/prometheus#3966 — "Optimise PromQL"](https://github.com/prometheus/prometheus/pull/3966)

**What it is:** A large, from-scratch rework of the PromQL query engine by a core maintainer (brian-brazil), aiming for a ~5x speedup on `rate()`-style queries over large series counts, with a claimed >99% reduction in garbage generated. Submitted deliberately mid-progress ("I started this last week, and while I'm not finished I want to share it") rather than waiting for a polished final diff.

**What happened in review:**

- **Architectural pushback.** A second maintainer, juliusv, raised a real concern: collapsing ~50 small, per-step evaluation functions into a handful of dense, do-everything functions made the engine faster but harder to reason about — especially for newcomers trying to touch the codebase later. He described it as a "frog in boiling water" problem: each individual change seems justified, but the aggregate result is an engine nobody dares modify anymore.
- **The author's counter-argument** was that centralizing the logic in fewer places was *more* legible than the status quo, which only felt readable because people were already familiar with it — not because it was objectively simpler.
- **A third maintainer, fabxc, made the actual merge call.** He explicitly said he found these judgment calls hard, acknowledged he hadn't personally put in the work to suggest alternatives, but ultimately decided to merge — reasoning that PromQL's engine was expected to stay structurally stable going forward, so this wasn't setting a precedent for continuous complexity growth. Crucially, he stated the tradeoff **was not being ignored**, just accepted this once.
- **A stream of small, non-blocking nits ran in parallel** with the big architectural debate:
  - An exported type (`evalNodeHelper`) accidentally referenced an unexported type, meaning it couldn't actually be used outside the package — a real bug, fixed inline.
  - A `for` loop that should have been a plain `if`.
  - Whether a particular context-cancellation check was redundant.
  - A request to move comments onto their own line above a struct field (rather than trailing it), specifically so IDEs would surface them as tooltips — a readability/tooling concern, not a correctness one.
  - A house style rule enforced even on a huge, already-complex diff: local variables shouldn't be capitalized.
- **Explicit gatekeeping on sign-off.** The author himself said he wanted at least one more maintainer's OK before merging, despite having commit rights — treating a second LGTM as a real requirement for a change of this size, not a formality.
- **A "drive-by" positive comment** — one participant said they weren't going to review the change in depth but wanted to say it was nice work. Small signal that not every comment on a thread is a blocking review comment; encouragement is separate from approval.

**Link:** https://github.com/prometheus/prometheus/pull/3966

---

## PR #2: [prometheus/prometheus#12114 — "Release 2.43.0-rc.0"](https://github.com/prometheus/prometheus/pull/12114)

**What it is:** A routine release-candidate PR, opened by a maintainer (roidelapluie) as part of the normal release process.

**What happened in review:**

- A second maintainer (kakkoyun) reviewed and left a single **LGTM**, no comments on the diff itself.
- Before merge, the author triggered `/prombench v2.42.0` — an automated benchmarking bot that deploys both the new PR build and the previous release, runs them side by side, and posts links to comparison dashboards (Prometheus meta-monitoring, Grafana, Loki logs) so reviewers can visually check for performance regressions instead of reasoning about them from the diff alone.
- The benchmark run took multiple days; the PR wasn't merged until it completed.
- No architectural or style discussion at all — the "review" here is almost entirely automated regression detection plus a single human sanity check.

**Link:** https://github.com/prometheus/prometheus/pull/12114

**Why this one matters as a contrast case:** it shows that Prometheus's rigor is *proportional to risk*, not a flat bar applied to every PR. A release-prep PR with no logic changes gets a one-line approval; a structural rewrite of the query engine gets a multi-day design debate.

---

## Supporting context on review conventions

- [Go Wiki: CodeReview](https://github.com/golang/go/wiki/CodeReview/2f90205c2a5e179948caf9be2b5662bf0102e40e) — the general Go project conventions around review etiquette (e.g., "DO NOT REVIEW" / "DO NOT SUBMIT" markers, replying only inside the review tool, not by email) — useful as a comparison point even though it's Gerrit-specific rather than GitHub-specific.
- [golang.org/x/review/git-codereview docs](https://pkg.go.dev/golang.org/x/review/git-codereview) — explains mechanics like `-trybot` (kicks off CI) and `-autosubmit` flags, which parallel Prometheus's `/prombench` bot in spirit: automated gates that reduce how much reviewers have to manually verify.

---

## Synthesized review norms

### What gets flagged
- **Architectural/API surface changes** — anything that increases coupling between packages, accidentally exports internal types, or makes core logic (like the query engine) harder to trace later. This is treated as a first-class review concern even when the change also delivers a clear, measurable win.
- **Style consistency**, enforced even inside large, already-complex, already-important PRs (e.g., local variable capitalization). Being a "big" PR doesn't buy exemption from small-scale conventions.
- **Control-flow cleanliness** — redundant loops, unnecessary conditionals — flagged as nits even when functionally harmless.
- **Documentation ergonomics** — comment placement that affects IDE tooltip behavior was treated as worth raising, not dismissed as bikeshedding.
- **Performance regressions**, but checked empirically (via the benchmarking bot) rather than argued about in the abstract.

### What doesn't get flagged
- **Unfinished or rough submissions.** The big PromQL rewrite was explicitly posted mid-progress, and no one objected to reviewing work-in-progress code — the norm is "share early," not "polish before you show anyone."
- **Small, well-understood performance tradeoffs**, once named. A few percent slowdown on already-fast, small-scale queries was accepted without pushback after the author flagged it upfront.
- **Routine, low-risk changes** (dependency bumps, release-candidate cuts) get essentially no line-by-line scrutiny — review effort scales down sharply once risk is low and the safety net (CI, benchmarks) is automated.

### How disagreements got resolved
- **Named tradeoffs, not votes.** The maintainer making the final call (fabxc) didn't average opinions or defer to seniority — he explicitly laid out both sides, said the decision was genuinely hard, and then made a call, while being clear the underlying concern (rising complexity) wasn't being dismissed, just accepted this once.
- **The objecting reviewer's concern remained on record even after being overruled.** juliusv's "frog in boiling water" framing didn't block the merge, but it wasn't waved away either — it reads as a deliberate, documented trade rather than a maintainer simply winning an argument.
- **A second sign-off was treated as a real gate**, not a courtesy, even for a maintainer with commit rights — the author explicitly waited for another maintainer's OK on the large PR.
- **Automation substitutes for debate where possible.** For the release PR, instead of reviewers manually reasoning about performance impact, a bot ran an actual before/after benchmark and posted the dashboard — turning a potential subjective disagreement into an empirical check.

