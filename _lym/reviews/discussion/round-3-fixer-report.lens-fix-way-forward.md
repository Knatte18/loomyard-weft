```yaml
lens: fix-way-forward
outcome: reported
```

fix-way-forward lens: I checked the ways forward that the round-3 fix to `_lym/discussion/decision-record.md` introduced or still depends on.

The fix added no refusal, step, row or test. It removed D5's motif-run clear, its Bound clause, the tier 1 criterion and the branch Fact. It added three sentences: a finished motif run becomes a Sketch, only raddle takes a Sketch, and batten gains no clear.

**Ways forward the changed text now relies on:**
- **The batten refusal's kept `lym batten cleanup --apply` clause (D5, first bullet).** It holds without the clear. A tapestry start over a done motif run is not cleared, because `startClearsMotifRun` admits only raddle. Cleanup removes the done record, then the upsert to tapestry runs. That text predates this fix and is unchanged.
- **The draft task a refused Sketch leaves over its done motif run (`sketchFinish`).** The new sentence "jacquard builds ... other discussed tasks" covers this task when it is set to jacquard. It holds its stored record, so the jacquard refusal passes it, and the start meets batten's already-completed refusal. That refusal names `lym batten cleanup --apply` from the prime and deleting the leftover task branch (`internal/battencli/arm.go:947`). So it can be carried out and leads to a fresh start.
- **The F17 loop (cancel refused at Worktree-Teardown, `internal/battencli/cancel.go:345`).** The record no longer names that sequence, so no way forward in it leads into the loop.

no findings

Tool calls:
1. I grepped `internal/battencli` for the done-run start refusal text, to check what a start over an uncleared done run names. My context did not show that refusal.
2. I grepped `internal/battencli` for refusals naming `cleanup --apply`, to confirm that the already-completed refusal names a way forward that can be carried out. My context held only `startClearsMotifRun` and `childReachedParent`.
