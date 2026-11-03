---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/hibernateengine/carrier.go:85
    summary: HoldsReexec stops holding the moment the sequence writes its terminal phase, while the sequence still lifts the entry and queues the wake notice, so a re-exec there loses both for good
    fix: In internal/hibernateengine/carrier.go change HoldsReexec to return c.running && c.phase != PhaseWaiting, dropping the Terminal test, so the hold lasts until the goroutine ends and running turns false after the lift and the notice. Update the HoldsReexec doc comment, and the Bound paragraph of internal/hibernateengine/doc.go, to say the hold lasts until the sequence has sent its notice. In internal/hibernateengine/carrier_test.go, replace the last assertion of the first TestCarrier subtest, HoldsReexec() = true once the request ended, with a poll that waits up to a deadline for HoldsReexec to turn false. waitTerminal returns as soon as the file is terminal, which is before the goroutine ends.
---

### [MEDIUM] HoldsReexec lets a re-exec cut the sequence's lift and notice after the terminal write

**Location:** internal/hibernateengine/carrier.go:85 (with `finish`, `failRequest` and `sendNotice` in internal/hibernateengine/watch.go)

**Issue:** `HoldsReexec` returns `c.running && c.phase != PhaseWaiting && !(Request{Phase: c.phase}).Terminal()`.
`finish` calls `deps.tellPhase` once it has written a terminal phase, so `c.phase` turns done, failed or cancelled at that point.
The goroutine keeps running after that write, and `c.running` stays true until `carry` returns.
On the success path `carry` runs `finish(PhaseDone)` and then `sendNotice`, which retries with doubling waits for up to ten minutes (`noticeRetryWindow`).
On every failure path `failRequest` runs `finish(PhaseFailed)` and only then `deps.Lift` and `sendNotice`.
While the carrier sits in that stretch the dobby's binary check is not held.
A re-exec there ends the process with the notice unsent.
On the failure path it can also end it with the owned pause entry unlifted.
The request is terminal, so `takeUp` leaves it alone, and nothing sends the notice or lifts the entry again.
The orch then never hears that the machine woke or that the request failed, and the hub can stay paused with no message saying so.
Only `lym hibernate cancel` on a terminal request lifts the entry, and the operator has no signal to run it.
The `Bound` paragraph in doc.go names only the instant the sequence moves past waiting, and does not mention this gap.
The decision record asks that the request survive the move into dobby, and the hold exists to protect the wake, which this gap leaves open.

The test hides the gap and is racy.
The last check of the first `TestCarrier` subtest asserts `HoldsReexec() == false` straight after `waitTerminal`.
`waitTerminal` returns once the file reads terminal.
`finish` writes the file first and calls `tellPhase` after `updateRequest` returns, so the check can run in the window between the two and see `true`.
A test that passes only most of the time breaks the testing rules.

## Rejected

- Focus 1, one source of truth for the ended phases: `hibernateEndedPhases` repeats the three terminal values because the package cannot import the engine.
  `TestHibernateEndedPhases_AreTheEnginesTerminalPhases` compares it with `hibernateengine.TerminalPhases()` in both directions, so any difference fails the test.
  `TestHibernateDaemonLive` covers no file, a live carrier, a dead carrier with the wrong start time, a zero pid, and done, failed and cancelled with a live carrier.
  The live row also pins the `request.json` name, the `daemon_pid` and `daemon_start` tags and the `HubScratchDir` path to the engine's own request type.
  Not a defect.
- Focus 2, hibernate directory on a hub that never hibernated: `NewDobbyCarrier` creates the directory with `os.MkdirAll` inside `hubDobbyDeps`, before the `Watch` closure runs `openDobbyWatch`.
  `TestHubDobbyDeps_PrimeTidyShuttle` stats the directory before any tick, and `dobbyWatchDirs` lists it for the prime only.
  Not a defect.
- Focus 3, the reconcile wording and the wake after a cancel: `carry` adds ", its wake reconcile may not have run, which `lym cleanup --apply` runs" to the reason of a request found in `resuming`.
  `failRequest` puts that reason into the notice, and the third `TestTakeUp` case asserts both phrases.
  `Carrier.Tick` ends with `wakeWatch`, which rewrites the request file after the goroutine clears `running`, so the file watch runs the next tick and a request submitted after a cancel is taken up without waiting for the loop timer.
  The second `TestCarrier` subtest covers it.
  Not a defect.
- Focus 4, cleanupcli comments: the test comment at `cleanup_integration_test.go:329` and the comments in `doc.go` and `reconcile.go` now name the hibernate request's wake.
  A search of the hibernate, cleanup and beam packages finds no remaining reference to the removed daemon spawn, the `watch` verb or the daemon log.
  Not a defect.
- `PATTERN-named-constants` on `hibernateRequestFileName` in `anchordeps.go`: the constant and its import-cycle comment are not in the diff, and the new test now pins the literal.
- Scope against the decision record: the prime's dobby alone carries the request, the request is kept on disk and retaken on start, the confirmation wait and its fork rule are unchanged, and `ResumeOnWake` is kept, so the diff matches the amended entry.
