```yaml
lens: fix-claims
outcome: reported
```

Task: check every claim the round-1 fix added or rewrote against the code at HEAD.

**Findings**

1. **MEDIUM**: the line added to D2 ("A flag without a file arises only from a rename: an older binary's …, or a crash between a rename's save and its move (D4)") and the matching crash sentence added to D4's rename bullet.
   - **Summary:** "only" overstates the bound. A move that fails without a crash also leaves the new slug flagged with no file, and so does a hand deletion of the file.
   - **Evidence:** `internal/boardengine/board.go:151-159` saves `board.json` first and then runs `afterSave`. An `afterSave` error is only returned (`return nil, err`); the saved rename is not undone. So a failed move, such as an IO error, leaves the renamed entry flagged without its file, and the rename verb reports an error after its slug change has already been saved. That also breaks the promise in the `rename.go` header that "a refused rename writes nothing", which the fixed D4 now leans on. The previous support log entry already named hand deletion as a way to reach this state.

2. **LOW**: D8's added sentence "the failed-commit refusal names `lym loom decision store` as the step after `lym fabric commit` lands the record".
   - **Summary:** that refusal text is shared by many paths and runs, and for some of them the named store step is itself refused.
   - **Evidence:** `decisionWayFabricCommit` (`internal/loomcli/decision.go:102`) is used by every failed-commit refusal: the non-amendment one at :784 and the amendment paths at :566, :570, :583, :589 and :593. Those cover runs inside Discussion-Review and darn runs, where D9 refuses `decision store` ("refused … on a run that has not passed its approval"). The added sentence does not limit the store step to a run past its approval.

**Claims checked and found to hold**
- An older binary's rename carries `discussed` onto the new slug. `Store.Rename` changes the slug in place through `renameSlug` (`rename.go:67`), and `Task.Unknown` keeps keys it does not declare (`task.go:244`).
- A rename refused after the mutation moves nothing. `checkNewSlugs` runs at `board.go:144`, before the save and before `afterSave` at :156.
- The landed-despite-error path exists: `afterFailedCommit` returns `landed`, and the code goes on to the post-commit check (`decision.go:786-791`).
- The arbiter test in D9 is the arbiter-strand check, `deps.arbiterStrand` (`decision.go:647`).
- A merge that removes a slug and upserts it again goes through `NewTask`, because the removed entry is dropped from `projected` before the lookup (`store.go:1008-1058`).

**Tool calls**
- grep in `rename.go` and `task.go` for unknown-key handling: my context did not show how rename treats undeclared keys.
- `rename.go:39-110` plus a grep for `afterSave` callers: my context did not hold the body of `Store.Rename`.
- `decision.go:540-560`, `decision.go:770-800` and `board.go:150-165`: my context did not show what happens to the save when `afterSave` errors, or the full call sites of the failed-commit refusals.
