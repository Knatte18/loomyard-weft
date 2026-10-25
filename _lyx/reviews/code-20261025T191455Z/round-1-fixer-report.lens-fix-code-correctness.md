```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F1, F2 and F3 code changes. I made no further tool calls.

No findings.

- **F1 (`moveWeftContent`, `moveWeft`):**
  - A failed `weftHeadSHA` read after the commit now returns `(true, err)`, so `moveWeft` skips `rollback`.
  - Every pre-commit failure returns `false`.
  - The only other caller, `addanchor.go`, discards the bool, so its behaviour is unchanged.
  - The new integration row would take the old `sha == ""` branch with the defect in place. That means `rollback`, `landed` false and no record, so the row fails without the fix.
  - The row is not parallel, and its seam is restored by `t.Cleanup`.
- **F2 (`MoveInProgress`, `heldMoveRefusal`):**
  - A held lock with a record whose pid and start time are not live now words the stop as "starting".
  - A live record still names the pid, so the live row passes.
  - A dead pid 4242 with an empty start time cannot match a live process, so the stale row is deterministic.
  - The `LockHeld` row and the `addparent` row both asserted the old wrong "in progress"/4242 text, and both were corrected.
  - `heldMoveRefusal` and `MoveInProgress` now share `recordNamesLiveHolder` and `startingMoveWayForward`.
  - Reviewed without edge-case testing: the `--set` target differs between the two. `MoveInProgress` uses `record.B` and `heldMoveRefusal` uses the requested target, as before.
- **F3 (`readRecordedAnchor`):**
  - An own marker that exists but cannot be read returns an error naming the file.
  - An absent marker, or an empty or whitespace-only one, still falls through to the board marker.
  - The `data, readErr =` reassignment compiles.
  - The new directory-marker test would get `("backend", true, nil)` from the old code, so it fails without the fix.

Tool calls: none.
