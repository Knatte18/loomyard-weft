---
round: 1
focus:
  - "`hubreconcile.Ensure` and its callers (cards 7, 9, 10, 11): does each start verb call it before any module config load, per Decision call-before-config-load, and does the `Preflight` row call it before `preflight.Check`?"
  - "Stamp and hub lock in `hubreconcile` (cards 2, 6, 7): is the stamp written only after a fully successful walk, is the lock released on every error path, and does the mid-merge skip and restore cover a commit that refuses after the write?"
  - "Pair re-enumeration and `lyx batten run` (cards 5, 7, 10): does the walk skip an incomplete pair, and does batten reconcile its new pair unconditionally after creating it?"
  - "Error surfaces (cards 3, 7, 9, 11): do `WorktreeError` and `LockTimeoutError` name the file and a way forward, and do the new `lyx loom` start-verb refusals have the loom-section rows `PATTERN-refusal-way-forward` requires?"
  - "`behind` drift class (cards 12, 13, 14): is ancestry supplied lazily, does `behind` carry no production-deploy advice, and does a missing or unreadable build commit degrade without misclassifying?"
  - "Edit directive (cards 15 to 21): does every spawned role's opening stencil other than the orch's carry the `edit_directive` marker, does each renderer read it through `editdirective.Directive` without a signature change, and is Master's restated script ban gone?"
  - "Per-card mechanical checks: for every `Rename` and `Delete` group in the cards (notably card 1's move out of `shedverbs`), did the type-specific check run against each of the group's targets?"
  - "Test economy: do the new reconcile, drift and stencil-scan tests duplicate coverage an existing test already gives, and does any added behavior lack a test?"
  - "Docs-in-same-commit: do the new `hubreconcile`, `editdirective` and `buildvcs` packages carry `doc.go`, and do the two new PATTERN entries land with their code?"
---

Seed focus from the plan's card index and shared decisions; no diff has been read yet.
