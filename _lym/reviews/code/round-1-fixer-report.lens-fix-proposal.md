```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals of F1–F5, which the fixer fixed as proposed, through the fix-proposal lens.

**F1 — `refuseKeptPause` in `Submit` (`verbs.go`, the `found && cur.Terminal()` branch)**

- LOW: a failed hub read now blocks every later request after an ended request that owns an entry.
  - Evidence: an on-path request that ended done keeps `Owned` recorded after its lift, so `refuseKeptPause` reaches `deps.TenderStatus()` for each such request.
  - The error becomes `hibernate: read the hub to check the previous request's pause: ...`, which names no way forward.
  - The call runs inside `updateRequest`, so the request lock is held across a status reading, which in the engine's other use (`awaitConfirmation`) is a pane-reading call that can fail.
  - The proposal read a seam built for a different purpose and made its failure fatal. A request that ended after a lift could instead pass, or the refusal could name `lym hibernate cancel`, which lifts by identity and needs no status read.
  - No test pins this error.
- Checked and sound: the entry match on scope, caller and `PausedAt` cannot hit another caller's entry. A lifted entry passes, and `TestSubmit` pins both outcomes.

**F3 — `recordOwned(deps, *kept)` on the keep path (`watch.go`)**

- LOW: the scenario the finding names cannot occur, so the fix covers a path only a test can reach.
  - Evidence: `kept` is set inside `Added`, and `Added` returns `recordOwned`'s error. In `fakeWorld.pause` that error fails the pause (`pause: report the added entry`), and `Watch` returns through `fail(...)` at the pause error branch, before the wake.
  - `r.Owned == nil && kept != nil` at the wake therefore happens only when something outside the daemon clears `Owned`.
  - The new row simulates that with an `onSleep` hook. It exercises the branch but does not show a real failure that reaches it.
  - `code-quality` says not to handle a state the code's guarantees rule out.
  - The change is harmless, and the on-wake path already tolerates the same clearing.
- Checked and sound: `recordOwned` writes with no `ours` check, so a request cancelled and replaced by a new request could take a stale `Owned`. Only a live, unkilled daemon can reach that, and `Submit` refuses over a live daemon. No finding.

**F2, F4, F5**

- No findings.
- Each new row fails with the defect in place:
  - The F2 pause-error and hibernate-error rows fail if `fail` followed the switch before the hibernate.
  - The F2 resuming-write row fails if `advance` went back to `fail`.
  - The F4 rows fail on a regression to the kept sentence.
  - The F5 flag-given row fails if the config were read before the flag was checked.

**Tool calls:** none. My context already held every changed file and the review.
