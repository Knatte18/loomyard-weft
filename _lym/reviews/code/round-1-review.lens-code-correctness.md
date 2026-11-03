```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff. The list is below.

**Per-function list (case run → verdict)**

- `StoreDecisionRecord`, held entry, same bytes and flagged → `noWrite`, correct. Absent slug → refused with way forward, correct. Done entry → `ErrRecordEntryDone`, correct. Held entry with new bytes → file written, flag set, `scopeChanged` ignores `Discussed`, no revised stamp, correct. A later check refusing the write leaves the new file with the old flag: documented, correct.
- `DecisionRecord`: absent entry, unflagged entry and missing file all answer not found, correct. Other read errors are returned, correct.
- `followDecisionRecords`, rename with old file present → moved, correct. Old file absent → file at the new name removed (this also happens for an unflagged entry; harmless), correct. Other rename error → reported after the save. Remove, prune, merge dropping a slug → file removed. A merge that upserts the slug again keeps its slug in `after`, so the file stays, correct. A failed removal is only logged, as designed.
- `boardCriticalSection`: `recordErr` is returned after render and sync, so the write is saved and the error follows. Correct, but see F1.
- `checkDesignPrefix`: "" , "d", "dec", "decision-" and "decision-x" are refused. "design-" passes. Correct.
- `checkDiscussedPayload`: key absent → pass. A matching bool is dropped. `true` against a stored false, and a non-bool, are refused. A new slug has stored false. Correct for upsert, batch and merge.
- `MergeTasks` `keepDiscussed`: the stored value is read before the removal, and re-applied to the rebuilt entry. Correct.
- `boardRecordSlug`, `Claims`, `CommitRuns`: "decision-a.md" gives "a"; "decision-.md", nested paths and "design-a.md" are not records. Correct.
- `Apply` record loop: the record is applied after the entry changes. A write whose entry origin lacks is lost; a removal (`After == nil`) skips that check and goes through the design-file rule. Correct.
- `PastDiscussionApproval`: an empty row, a row inside the segment and an unknown row are not past. A row after the segment is past, and retired rows are mapped. A recipe with no approval row gives `hasRow` false. Correct.
- `approveDiscussion`: a read error fails the settle with a way forward. A done entry warns and passes. An absent entry or board-open failure fails the settle. The slug comes from `ResolveRunID`, the same call `SetRunStatus` uses (`wiring.go:222`). Correct.
- `copyRecordToBoard`: not past → "". HEAD without a record, or a store error → text ending in the `decisionWayStore` way forward. Done → text with no pointer at the store verb. Correct.
- `decisionStoreVerb`: an unnamed shell skips admission. The arbiter passes. The parent passes; any other named session is refused, and an unresolvable parent is refused with a way forward. No approval row → refused. Not past → refused. No committed record → refused. Done → refused. Correct.
- `decisionVerb` `wayCommit`: computed from the status read before the commit, and appended only when past. Correct.
- `bouncerEntry` `approve_seam: discussion`: guarded by `requireSeam`. `approveAndCommit` calls Approve, then Commit, on all four callers: `approveWithoutReview`, `resettle`, `settle`, `acceptRound`. Correct.
- `lym board get --record`: refuses with `--body`, an absent entry, an undiscussed entry and a missing file. Correct.

**Findings**

- **F1 — LOW, design.** `internal/boardengine/rename.go:31-33`: `Board.Rename` returns `RenameResult{}` whenever `boardCriticalSection` returns an error. After a record-step failure the entry is already saved under the new slug, but the caller gets an empty result and loses the saved `Task` and `Mentions`. The decision requires only that the error is reported. Fix: when the error is a post-save record error, return the result with the error. That needs a distinct error type; otherwise accept the loss and say so in the doc. The only `.Rename(` callers my grep found outside tests are `os.Rename` calls, so the boardcli verb is the one reader.
- **F2 — LOW, design.** `internal/boardengine/decision.go:202`: the orphan removal at the new slug runs for any rename whose old file is absent, including an unflagged entry. The parent's added decision says a flagged entry. The behavior is a harmless superset, since a file at a fresh slug's name is always an orphan. Fix: gate the removal on the renamed entry's `Discussed` flag, or document it as intended. I did not check whether a test pins it.

Nothing else wrong in the paths I ran.

**Tool calls**
1. `grep` of tapestry-recipe.yaml for the DiscussionWrite row: to confirm its `on_done` is a Bouncer row (it is, Discussion-Bouncer).
2. `grep` for `.Rename(` callers: to see who reads the rename result (no non-test caller of `Board.Rename` found beyond the boardcli verb).
3. `grep` of `board.go` for `RemoveTask`, `MergeTasks` and `Prune`: to confirm they go through `boardCriticalSection`, so the record removal runs (it does).
