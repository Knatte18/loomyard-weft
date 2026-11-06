---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/beamcli/dobbydeps.go:263
    summary: The wait reminder Post seam, the one line that reaches the operator, is called by no test; the wiring test only checks it is non-nil.
    fix: In internal/beamcli/dobbydeps_integration_test.go, extend TestHubDobbyDeps_WaitReminders (or add a row beside it) to call primeDeps.WaitReminders.Post and beamDeps.WaitReminders.Post with a line over the hubforge hub, then read the posted line from the mailbox of <shortname>:SLACK with its sender signed DOBBY for the dobby's slug, following the creel fixtures in internal/dobbyengine/landnotice_creel_test.go. Fail the row on a wrong recipient, wrong signing or a swallowed post error.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/waitreminder_test.go:61
    summary: Two behaviours the diff adds have no test row; a run with a batten cancel marker is skipped, and a standing anchor change stops the wait check.
    fix: Add one row to TestWaitReminders for a run with CancelMarker set that expects no post and no recorded wait, and one row (or a loop test beside it) that creates an anchor-move record in the dobby's geometry and expects the wait check to be skipped, as the doc.go bound states.
---

Verdict: the diff makes the change D8 and its acceptance criterion ask for, and no defect blocks it.

## Review of the target against the fasit

### [MEDIUM] The Post seam has no test

**Location:** internal/beamcli/dobbydeps.go:263, internal/beamcli/dobbydeps_integration_test.go:200
**Issue:** `waitReminderDeps` builds the recipient with `agentname.FormatProgram`, the sender with `creelcli.NewScopedProgramClient(worktree, slug, dobbyProgram)` and the post with `client.Post`. The integration test reads the `Rows`, `Interval`, `Away` and `LatestResume` seams but only checks `Post` for non-nil, and the engine tests use a fake `Post`. A wrong program name, the wrong scope slug or a swallowed error would pass every test, and this is the call that reaches the operator. The rubric's test economy rule counts a behaviour added with no coverage as a finding.

### [LOW] Two added behaviours have no test row

**Location:** internal/dobbyengine/waitreminder_test.go:61
**Issue:** `readWaits` skips a run with a cancel marker, and `tick` runs the wait check only while no anchor change stands; the doc.go bound states the second. No row in `TestWaitReminders` or the loop tests fails if either is broken.

## What the review checked and found correct

- **Wait start (focus 1):** `readWait` starts a wait at `now` on the first read at the row with no covering pause, moves it to a later `resumed_at`, and returns zero, which `recordWaits` clears, when the child leaves the row, stops running or is covered by a pause. A resume that predates the first read leaves the start at `now`, which is the later of the two as D8 says.
- **Presence (focus 2):** the start is recorded whatever the presence; `remindOfWaits` posts only when `Away` reads true and clears `lastReminders` otherwise, so a return to `slack` reminds at once of a wait past the interval.
- **Cadence and failures (focus 3):** one line per interval from an in-memory last-reminder time. A refused post logs at Warn and does not advance that time, so the next check retries.
- **Rows and cadence of checks (focus 4):** the hub wiring lists only PR-Review and Sampler-Session, so Beam-Await is excluded. The check runs in `tick` for every dobby, the prime's included, throttled to one a minute by `lastWaitCheck`.
- **State record (focus 5):** `OperatorWait` is an `omitzero` field of `RunState`, written through `UpdateState`, so an old or undecodable record still loads under PATTERN-tolerant-records.
- **Pause lookup and wiring (focus 6):** `LatestResumed` picks the latest `ResumedAt` among covering resumed entries, returns not-found for a slug with none or a pause still open, and ignores entries of other slugs. The presence reader, interval loader and SLACK post are wired to the existing slackengine and creel seams.
- **Config (focus 7):** `remind_after_min` keeps its default of 30 from the template, and `LoadConfig` already refuses a value under 1, so the check posts nothing and logs once on a bad value.
- **Docs and commit messages (focus 9):** `docs/overview.md` and the three `doc.go` files carry the change, the old "orch's reminders" wording is gone from every text that described the key, and both commit messages match the diff. Neither commit carries a `Pattern-Departure:` trailer and none is needed. The directive declares no PATTERN change.
- **Test economy (focus 8):** no test duplicates another. The acceptance criterion's paused-and-resumed case is covered with dobby reading the pause (`APauseTheDobbyReadRestartsTheWait`) and without (`APauseAndResumeNoCheckSawRestartsTheWaitAndItsReminders`).
- **Scope:** every changed file serves D8 and the entry's Takes, Delivers and Exports; the slackengine edits only reword the key's meaning.

## Rejected

- Lens code-tests, NIT: no per-field test that a state record without `operator_wait` loads. `internal/state` and its tests own the tolerance, and a per-field row would duplicate them.
- Lens code-correctness, LOW: `recordWaits` may recreate an empty `RunState` for a slug a concurrent writer pruned. The entry is empty and harmless, and the same pattern exists in `runnotice.go` and `owned.go`.
- Lens code-correctness, LOW: a `waitFailures` episode for a run that stops running is never recovered, so a later failure logs at Debug. The effect is cosmetic and inherited from the existing episode pattern for run notices.
- Lens code-tests, LOW: the nil-seam no-op of `waitReminders` has no row. The branch is a one-line guard of a seam the hub wiring always fills, and a test of it only pins internal misuse.
