---
round: 2
ledger:
  - key: fold-tables-missing-t-parallel
    rounds: [1, 2]
    status: open
    class: consistency
    severity: BLOCKING
  - key: hard-wrapped-new-comments
    rounds: [1, 2]
    status: open
    class: consistency
    severity: BLOCKING
  - key: stale-test-name-in-batten-bookend-background
    rounds: [1]
    status: resolved
  - key: helpers-left-without-caller
    rounds: [1]
    status: resolved
---

Round 1 raised four findings; round 2 raised three, all consistency and BLOCKING.
Two keys recur.
Fold tables the prune wrote still lack `t.Parallel` or a comment naming the global state that forbids it: round 2 found one serial `selfreportengine` file with no stated reason and one `loomshed` table missing its top-level `t.Parallel`.
New comments are still broken mid-sentence: round 2 found two `godocreflow` test doc comments.
The stale test name in `pattern/PATTERN-batten-bookend.md` and the helpers left without callers did not reappear, so both are resolved.
