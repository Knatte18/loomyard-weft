---
round: 2
focus:
  - "In internal/fabricengine/refscanner_test.go, the git config flag rows: settle whether each row reaches the `-`-prefixed word check in `readOnlyGitSegment` rather than being rejected earlier by the two-or-three-word count bound, so that removing the flag check would turn a row red."
  - "In internal/websterengine/recordbatch.go, `pathlessViolationClasses` against the class constants in audit.go: settle whether every class the audit can emit as pathless is skipped by the three evidence readers, and whether a test fails if the list drifts from the constants."
---

The second entry follows the fix for the first; a fresh reviewer should judge both from the committed diff.
