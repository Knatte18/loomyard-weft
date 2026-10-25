```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1, F2 and F3 (the fix-proposal lens).

**Findings**

1. **LOW** — `moverecord.go`, the `!acquired` branch of `MoveInProgress` with `startingMoveWayForward(slug, record.B)`.
   - **Summary:** the "starting" stop names the stale record's `B` as the re-run target, which can be the wrong anchor.
   - **Evidence:**
     - Take a killed run A→B, then a new run A→C that holds the lock and has not rewritten the record.
     - Pull, push, reconcile and the dobby gate then print "wait for it to end, then run `lyx fabric anchor <slug> --set B`".
     - The holder is heading to C, so that advice would send the weft to B.
     - Only the refusing run's own caller knows its target. `heldMoveRefusal` uses it; `MoveInProgress` has none.
   - **Mitigation:** the wording could drop the re-run command for the starting case.

2. **LOW** — `anchor.go`, the `case !errors.Is(readErr, fs.ErrNotExist)` branch in `readRecordedAnchor`.
   - **Summary:** on Linux, `ENOTDIR` is not `fs.ErrNotExist`, so a records sibling path whose parent is a regular file now errors instead of falling back.
   - **Evidence:** that case was previously treated as absent. It is unlikely in practice, since the sibling path is derived from the hub and not from user input.
   - **Mitigation:** if the case is real, test for `syscall.ENOTDIR` too.

**Checked and found sound**
- **F1:** the one other caller, `moveForkedWeftToAnchor`, is called from `add.go:532`. On any error that caller runs `rollbackAdd`, which removes the whole new pair. A landed commit with an unread sha therefore cannot leave a half-moved weft, and the `committed` flag it ignores does no harm.
- **F2 and F3:** the proposals are otherwise sound. `recordNamesLiveHolder` matches the old `heldMoveRefusal` check, and the no-record, live-record and stale-record paths all give the right wording.

**Further tool calls (2)**
- `grep` of `moveForkedWeftToAnchor` and `moveWeftContent` in `addanchor.go` and the other `fabricengine` non-test files — my context did not show who else calls them.
- `sed -n 520,545p internal/fabricengine/add.go` — my context did not show how the call site handles an error that arrives after the commit.
