---
round: 1
ledger:
  - key: list-find-text-layout-goldens-remain
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised one finding, F1: the sweep for layout goldens of human-read output missed the byte-exact `list --text` and `find --text` compares in `stepListAndFindText`, and the `sketches:` line pin in `stepListShowsASketch`.
The review's fix text names the exact replacement checks, so the fix is mechanical.
