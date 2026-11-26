```yaml
lens: fix-way-forward
outcome: reported
```

Task: check through the fix-way-forward lens that every refusal, error, next step or test the fix introduced can be carried out from the state it names and makes progress.

**Findings**

1. **MEDIUM** — D3, the new seed sentence: "The same seed writes each tracked report copy back to this anchor's live report path … and rewrites every seeded ask's `Report` field to that live path".
   - **Summary:** The seed has no step for a done ask whose tracked report copy is missing. Readers then still fail on the moved record, and nothing tells the operator how to recover.
   - **Evidence:** D3 writes the record copy and the report copies as separate atomic writes in one locked update. It names no order between them, and its Bound covers only "the last write under the lock". If the process is killed after the record copy and before a report copy, the tracked record holds a done ask with no report copy. The seed writes nothing back for that ask but still rewrites its `Report` to the live path. `ReadPRReviewStanding` (`internal/landingshed/prreviewask.go:118-121`), `SettleParentAsk` (`internal/burlerengine/parentreview.go:314-316`) and the parent seat's gate (`internal/burlerengine/parentseat.go:83`) then fail on the missing file. This is the F16 failure again, now with no way forward.
   - **Possible fix:** write each report copy before the record copy. Alternatively, have the seed treat a done ask that lacks a report copy as ask-again, or refuse it and name the missing path.

All other ways forward the fix introduced can be carried out and make progress:
- **The D6 leftover-sibling refusal.** It names the path. After the operator removes the sibling, the re-run passes the create-if-absent step, and this holds for a symlink sibling too.
- **The D6 failed rename after removing an empty `<dir>`.** It leaves `<dir>` absent, and D6 accepts a new directory, so a re-run succeeds.
- **A kill between that removal and the rename.** It leaves the sibling behind and `<dir>` absent. The re-run is refused on the sibling, the operator removes it, and the next re-run succeeds.
- **The D5 history search.** "Already up to date" means the task's tip is an ancestor of the parent's head, and the archive runs only after `OriginParentHoldsWork`. So on both paths some first-parent commit passes `Fabric.HoldsWork` and the search always finds one.
- **The new D3, D5 and D6 acceptance criteria.** Each can be reached from the state it names.

Tool calls: none; my context already held everything this lens needed.
