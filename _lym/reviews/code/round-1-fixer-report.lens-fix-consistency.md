**Findings**

1. **LOW** — `_lym/landing/summary.md`, Behaviour notes, the bullet "The land check's own halt on a failed parent-holds read is gone: that read now happens once…".
   - The F2 fix made `Finalize.landUnderLock` read `parentHoldsWork` twice, so "now happens once" is false.
   - The "Helpers merged" bullet reflects the double read; this twin was left as it was.

2. **LOW** — `_lym/landing/summary.md`, "Helpers merged", the bullet on Finalize and PR-Review sharing "the read of the description, regression check and credit block".
   - F3 split `landingSummary`: the parse sits at each caller with its own wrap, and only `Deps.completeSummary` is shared.

Nothing else stale: the `Finalize.readAttempts` doc, the `readRowAttempts` doc, the `deps.go` optional-seam comment and the `doc.go` sentence still match the code.

Tool calls: none.
