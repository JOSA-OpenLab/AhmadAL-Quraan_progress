

**Syft + Grype** → _"What's inside my build, and is any of it known-vulnerable?"_

- Operates on your **built artifact** (a Docker image, a source tree).
- Syft inventories every package/library and its version → SBOM.
- Grype cross-checks that SBOM against a CVE database → tells you which specific dependencies have known vulnerabilities.
- Answers: **"Am I shipping known-bad code, even unintentionally?"**

**Scorecard** → _"Is my repo's process trustworthy?"_

- Operates on your **GitHub repo's practices**, not your build artifact.
- Checks things like: are Actions pinned by SHA, is branch protection on, are releases signed, are tokens over-permissioned, does CI run untrusted code unsafely.
- Doesn't look inside your dependencies at all — it looks at _how you build and ship_.
- Answers: **"Even if my code is clean today, is my process resistant to tampering, compromise, or sloppy practices?"**

**Cosign/Sigstore** → _"Is this specific artifact authentic?"_

- Operates on a **published release file**.
- Doesn't check for vulnerabilities or practices — just proves "this exact file came from this exact pipeline, unmodified."
- Answers: **"Can a downloader trust this file wasn't tampered with after I built it?"**




### How they fit together (the mental model)

Recall the three attack-surface layers from your slides:

| Layer                   | Attack type                  | Defense                                 |
| ----------------------- | ---------------------------- | --------------------------------------- |
| Your dependencies       | malicious/vulnerable package | **Syft + Grype** (find known-bad deps)  |
| Your CI process         | compromised build pipeline   | **Scorecard** (audits pipeline hygiene) |
| Your published artifact | tampered after build         | **Cosign/Sigstore** (proves integrity)  |

They're not chained (Syft's output doesn't feed Scorecard, Scorecard doesn't feed cosign) — they're **three independent checks covering three different attack surfaces**, all under the umbrella of "supply chain security." A project can pass one and fail the others — e.g., your Inception image had zero CVEs in its own code but pulls in a vulnerable `perl` via `mariadb-server` (Grype's job to catch), while separately your workflow tokens were over-scoped (Scorecard's job to catch), while separately your release artifact needed a cryptographic proof of authenticity (cosign's job to provide).

Together they cover: _what's in it, how it was built, and is this copy the real one._