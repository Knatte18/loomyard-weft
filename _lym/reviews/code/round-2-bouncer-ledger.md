---
round: 2
ledger:
  - key: entry-vs-archive-tag-push
    rounds: [1]
    status: resolved
  - key: lock-not-pinned-per-method
    rounds: [1]
    status: resolved
  - key: hold-through-verdicts-unpinned
    rounds: [1]
    status: resolved
  - key: pushrebasefree-doc-lockfree
    rounds: [1]
    status: resolved
  - key: fetch-head-read-after-release
    rounds: [1]
    status: resolved
  - key: locks-held-through-lsremote-retries
    rounds: [1]
    status: resolved
  - key: fabricengine-fetch-sites-unpinned
    rounds: [1]
    status: resolved
  - key: reconcile-release-paths-unpinned
    rounds: [1, 2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: entry-lacks-test-marker
    rounds: [1]
    status: resolved
  - key: remote-refs-lock-departure-unrecorded
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
---

Round 1's gating design findings (per-method lock pinning, hold through verdicts) and its doc and consistency findings no longer appear in round 2, and the review's focus answers confirm the fixes.
`reconcile-release-paths-unpinned` remains in part: the failed-fetch return still lacks a lock-free assertion (F2).
F1 is new: the repository-wide remote-refs lock departs from the push lock the entry names, and only the parent can record or refuse it.
