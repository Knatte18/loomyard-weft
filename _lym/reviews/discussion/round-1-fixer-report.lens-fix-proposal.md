```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: for each fix the fixer applied as proposed, I built a case where the proposal is wrong.

**F6 — BLOCKING.** Changed text: D6, "Only a tag with no index row whose history holds Finalize gets a line derived…", and the criterion "a cancelled run's tag is listed as not landed".
- A Finalize entry in the history does not mean the run landed. Stuck outcomes are appended to the history too: `internal/shedengine/reroute.go:49` and `:70` append `HistoryEntry{Producer: def.Name, Outcome: Stuck, …}`, and `run.go:359` appends every outcome.
- `readLoomRecords` checks only `entry.Producer == loomshed.NameFinalize` (`internal/reportengine/runs.go:280-282`).
- Concrete case: Finalize goes Stuck before the merge (spent fixer, failing gate or protected push), and the operator then cancels the run. Its tag has Finalize in its history and no index row, so D6 gives it a derived landing line. The run never landed but reads as landed, which is the premise F6 raised.
- The rule needs a Finalize entry whose outcome is Done.

**F7 — MEDIUM.** Changed text: D6 Bound rule 4, "writes into a new sibling directory and renames it to `<dir>`… so a failure leaves `<dir>` absent", and the criterion "an extract that fails part-way leaves `<dir>` absent".
- The sibling directory is outside `<dir>`, so rule 4 breaks the bound one sentence above it: "writes only under `<dir>`". A process killed mid-extract leaves that sibling behind, and nothing names it or cleans it up.
- When `<dir>` already existed and was empty (which is allowed), a failure leaves `<dir>` present, not absent. Rule 4 and the new criterion are both false in that case.

**F4 — MEDIUM.** Changed text: D1, "a tracked `.gitignore` that ignores the `.tmp-*` names `fsx.AtomicWriteBytes` gives its temp files".
- The rule pins one writer's temp naming. `internal/landingshed`'s own atomic writer uses `os.CreateTemp(dir, "."+kind+"-*.tmp")` (`internal/landingshed/mergerecord.go:92`).
- D4's copies are written from `internal/landingshed`. A copy made through `writeBytesAtomic` leaves `.<kind>-*.tmp` files that the ignore rule misses, and the status commit stages them.
- The rule needs either to name every writer of the kept directory or to require that they all go through `internal/state` and `fsx`.

**F5 — MEDIUM.** Changed text: D3, "A locked write that finds no live record but a tracked copy… first seeds the live record".
- Only writes seed. Readers call `ReadParentAsks`, which reads the live record alone (`parentreview.go:197`), at `internal/landingshed/prreviewask.go:109`, `internal/loomcli/parentreview.go:226` and `:336`, `internal/driftengine/judgeparent.go:84` and `:131`, `internal/shedadapters/parentseat.go:116` and `bouncer.go:1161`.
- After a takeover, every reader sees no asks until some write happens, while the next writer sees the seeded history.
- Concrete case: a pending or open ask from before the move is invisible to the answer path's read on the new machine. The seed has to happen where the record is read, not only inside a locked write.

**F1 — MEDIUM.** Changed text: D5, "A resumed Finalize that finds the landing record reads the commit back from it and never re-reads the parent's head", with only the archive path allowed to replace the record.
- A failed parent push goes Stuck with the way forward "push it by hand" or "reconciles a diverged remote by hand" (`internal/landingshed/finalize.go:811-812`, `:843-848`).
- When the operator reconciles by rebasing or resetting the parent, the landing commit changes. The resumed Finalize then reads the stale record, and the index row names a commit that is not on the parent.
- D5 names no way to refresh the record in that case.

**F10 — LOW.** Changed text: D2, "a file whose hash a copy already carries is not copied again".
- When a later PR-Review round sends back a byte-identical findings file, after Code-Review failed to fix it, that second send-back is not copied. The tracked record then loses the fact that the same findings came back twice.
- Also, "the hash the verb computes" is computed only after the send-back early return (`internal/loomcli/prfindings.go:218-222`), so the send-back path computes no hash today.

**F15 — LOW.** Changed text: the PATTERN change that moves "archive tags are never deleted" into `PATTERN-pair-remote`.
- That entry's trigger is "Pushing run records or tearing down a pair". The future prune that D7's rationale guards against would most likely be a cleanup verb, which neither pushes records nor tears down a pair, so the rule's trigger would not fire for it.

Outside my lens: the fixer's own D4 addition, "A report identical to the latest copy of the same row and row entry is not copied again", collapses two failing turns that wrote identical reports into one copy. That conflicts with D4's criterion that two turns leave two tracked reports.

Tool calls:
- Grepped `internal/state` and `landingshed`'s `writeBytesAtomic` for temp naming; my context had only `fsx.AtomicWriteBytes`.
- Grepped `ReadParentAsks` callers and the pending-post states; my context had only the record's write path.
- Grepped `runRecipes` and how history entries are appended; my context did not show whether a Finalize entry implies Done.
- Grepped the `HistoryEntry` outcomes in `shedengine`; I needed proof that Stuck Finalize entries exist.
