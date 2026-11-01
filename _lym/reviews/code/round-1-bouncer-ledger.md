---
round: 1
ledger:
  - key: entry-vs-archive-tag-push
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: lock-not-pinned-per-method
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: hold-through-verdicts-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pushrebasefree-doc-lockfree
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: fetch-head-read-after-release
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: locks-held-through-lsremote-retries
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: fabricengine-fetch-sites-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: reconcile-release-paths-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: entry-lacks-test-marker
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round; no earlier ledger exists.
Two gating design findings, `lock-not-pinned-per-method` (F2) and `hold-through-verdicts-unpinned` (F3), are both missing deterministic tests.
`entry-vs-archive-tag-push` (F1) needs a choice between narrowing the PATTERN entry and routing the tag push through `gitrepo`.
`fetch-head-read-after-release` (F5) is a predating race the diff left open.
The remaining LOW findings are doc, test-coverage and enforcement gaps.
