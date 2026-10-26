```yaml
lens: code-correctness
outcome: reported
```

**Task:** the code-correctness lens over the diff, covering `beamawait.go`, `beamup.go`, `orchcli/start.go`, `orchengine/owner.go`, `runproc/end.go` and `boardengine/owners.go`.

**Per-function list (case ran, correct or not)**
- `BeamAwait.Call` / `wake`, owner read fails: logs and decides nothing. Correct.
- `wake`, owned elsewhere: removes the request and yields. Correct.
- `wake`, status write fails on entry: returns without reading the request, and the next wake retries. Correct.
- `wake`, request names another run: removes it, sets `acted`. Correct.
- `accept`, status write of Parent-Align fails: keeps the request, returns. Correct.
- `accept`, re-check passes: removes the request, then the dobby-stop marker, then the await marker, and returns Done. Correct and in D9 order.
- `landChecks`, marker write fails: returns a blocker, and `refuse` then finds no marker and does not restart the dobby. Correct, since the dobby was never stopped.
- `landChecks`, `StopDobby` fails after the marker is written: `refuse` restarts it because the marker stands. Correct.
- `landChecks`, `Recheck` returns an error: one blocker, then `refuse`. Correct.
- `landChecks`, `WriteSnapshot` fails: one blocker, then `refuse`, which restarts the dobby. Correct.
- `refuse`, status write fails: keeps the request and marker, restarts nothing, posts nothing. Correct per D9 (see F1).
- `refuse`, marker unreadable: treated as stopped, so the dobby is restarted. Correct.
- `refuse`, restart fails: leaves the marker, removes the request, and the notice names `lyx beam start`. Correct per D9.
- `refuse`, restart succeeds: removes the marker, removes the request, posts the blockers. Correct.
- `enter`, interrupted landing: lifts the hold first, removes the stop marker, posts the notice. Correct.
- `enter`, no await marker (first entry): writes the marker and skips the dobby-down check. Correct.
- `enter`, marker stands and not interrupted: calls `noticeDownDobby`, which posts only when no request stands and the dobby is not live. Correct per the Added-after-Discussion entry (see F2).
- `interruptedLanding`, read errors: read as not interrupted and logged. Correct.
- `pause`, `Watch` event: waits out the rest of the one-second floor. Correct.
- `BeamUp.Call`, a failing step returns a halting Stuck naming `lyx loom resume`. Correct.
- `BeamUp.Call`, removes the land request and await marker on every run, and cancellation is checked first. Correct.
- `Watcher.ownerTick`, nil owner: no-op. Correct.
- `Watcher.ownerTick`, read error: logs once per episode, reads as not elsewhere. Correct.
- `Watcher.ownerTick`, failed stop: error and retry next tick. Correct.
- `Watcher.ownerTick`, owned elsewhere: stops the strand, records the exit, done. Correct.
- `runproc.EndRole` / `end`, predicate selects only the role and spares every other record and what is below it. Correct.
- `Owners.Effective` / `ownersOf`, owner read only for beam projects, so a stray task owner is never read. Correct.
- `checkOwnerPayload`, payload equal to the effective owner is dropped, any other value refused with the `adopt` way forward. Correct.
- `orchcli.beamUp`, order is prepare (reconcile, claim, pauses), then the refusals, then dobby, window and orch. Correct per D1 and D3.
- `launchOrch`, resume failure: falls back to a fresh launch from the note or stencil and names the abandoned session. Correct.
- `launchOrch`, `RenderReloadPrompt` error: aborts instead of falling back. Questionable, not raised.
- `ensureDobbies`, `EnsureLive` returns no entries and a nil error: returns a zero `StartEntry`. Raised as F3.

**Findings**

- **F1, LOW, design, `internal/pawlengine/beamawait.go:268-272`:** On a failed re-check whose lifting status write also fails, the hold stays up, the dobby stays stopped and nothing is posted. The wake returns `acted` false, so the wait doubles up to a minute (`beamawait.go:93`). The retry, which re-runs `accept` and the re-check, can therefore be a minute away. D9 says a status left across the hold stands "at most until that wake", and an operator sees nothing meanwhile. Evidence: `refuse` returns `awaitWake{}` with no `acted`.
- **F2, LOW, design, `internal/pawlengine/beamawait.go:198-200` and `:204-218`:** The dobby-down notice fires on every driver re-entry that finds the marker standing, no request standing and the dobby not live. Batten re-spawns the driver at each probe once the previous one is dead, so a beam left with its dobby down is re-announced on every re-spawn, not once. The Added-after-Discussion entry says "posts one notice". Nothing in the code keeps a sent-notice episode.
- **F3, LOW, design, `internal/orchcli/start.go:290-300` and `:455-469`:** `ensureDobbies` returns a zero `StartEntry` with a nil error when `EnsureLive` reports no entries. `startDobbyStep` then sees an empty outcome and adds no warning, so a dobby that was never attempted passes silently. I did not read `EnsureLive` to confirm that no entries can come back for a named worktree. Fix: treat an empty report as a failed start and warn naming `lyx beam start`.

No finding on the other branches listed above.

**Tool calls (3)**
1. Read `internal/pawlengine/beamup.go`: the manifest lacked Beam-Up's steps and its marker removal.
2. `git diff` of `runproc/end.go`, `orchengine/owner.go` and `orchengine/watcher.go`: the manifest lacked the `EndRole` and owner-watch bodies.
3. `sed -n 55,140p internal/boardengine/owners.go`: the persisted diff output was truncated before `Effective`.
