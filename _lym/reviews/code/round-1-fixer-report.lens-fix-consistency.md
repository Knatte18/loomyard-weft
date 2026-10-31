```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the round-1 fix (carryover.go doc lines, doc.go sentence, new hold_test.go row).

No findings.

Checks:
- **F1 twin docs.** The `Raw` skip comes from `positionedTurnStarts` (`wait.go:972-974`). Both of its callers of concern, `runTurnStarts` and the `operatorInstructions` that reads it, now carry the same sentence. The other caller, `wait.go:946`, is the Wait path and is out of scope. `hold.go:111` uses `runTurnStarts`, whose doc now states the rule.
- **F3 doc.go.** The rewritten sentence matches the `runTurnStarts` contract: past the prompt offset, not the prompt, and not typed by a send. It no longer points at a missing section. The following sentence ("the turn start is the newest one before the turn end…") is unchanged and still agrees with it.
- **F2 row.** It uses the table's existing `deferral`, `sleepJump` and `timeout` fields, as the sibling deferred rows do. Its `sleepJump` and `timeout` match the "always outstanding" row, so it passes the 15m maximum. The `check` assertions match those of the non-deferred operator-turn row.

Calls made: none beyond one `grep` and one `sed -n` on `wait.go`, to confirm the other callers of `positionedTurnStarts` and its `Raw` skip; my context lacked `wait.go` and the call-site list.
