# Fixer-report — code review round 1

## Changes

- **F1** fixed with my own change: the proposed change was to leave the name empty or resolve it lazily.
  `decisionDeps.programName` is now a `func() (string, error)` built in `preRun`, so `preRun` no longer aborts on a `ReedGeometry` or `FormatProgram` failure.
  `noticeFits` skips the program-name length check when the name does not resolve, and the post as the program fails on the same geometry and is reported per strand.
  I did not refuse in `noticeFits` when the name is unavailable: the pre-record check runs before the live strands are known, so refusing there would abort an add that notifies nobody.
  Commit `b44df920`; `decision_test.go` fake updated.
- **F2** disputed and reverted: see Disputed.
  Commits `b797614f` (fix) and `4d56042d` (revert).
- **F3** fixed as proposed: `tools/sandbox/SANDBOX-CREEL-SUITE.md:115` quotes the sender as `from "<shortname>:EXTERNAL":`.
  Commit `130e2494`.
- **F4** fixed as proposed: a new `cli_test.go` subtest runs `hook-wait` against a rig with no daemon and asserts exit 0 with empty stdout and stderr.
  It has its own rig, because a second `newRig` in one test would replace the shared client seam.
  Its first form hung under the gate: the client waits for a daemon start on the fake client clock, so the subtest now runs `AutoAdvance` as the existing no-daemon subtest does.
  Commits `18a1537e` and the gate-repair commit after `4d56042d`.
- **F5** fixed as proposed: `TestOpenLaunch` asserts neither the launch nor the resume line holds `--append-system-prompt-file`.
  Commit `f45a8e6a`.

After the repair, tier1 of `./internal/creelcli` is green too.

`lym gate test` is green for `./internal/creelcli`, `./internal/creelengine`, `./internal/loomcli` and `./internal/shuttleengine/claudeengine` after the final change.

## Disputed

- **F2**, ground `every fix worse`.
  The premise holds: a strand with neither mark is polled while mail is unread.
  The proposed change, returning `false, false`, spins the worker: `run()` calls `retire()` on a not-pending probe, `retire()` returns false while `store.hasPending` holds (`internal/creelengine/deliver.go:440`, unread mail counts), and the loop continues to `probe` again with no wait, where `(false, true)` reaches the capped backoff.
  My variant, `false, len(snap.injects) > 0`, spins the same way for unread mail with no inject.
  Parking the worker until a wake would stop the poll, but a strand relaunched with a mark raises no wake, so its mail would then wait undelivered; polling is how the worker notices.
  The existing cost is bounded at the backoff ceiling, as the review itself says.

## Deferred

Nothing deferred.

## Self-check

- fix-consistency, BLOCKING: F2 spins the worker loop for a strand with unread mail and no mark. Fixed with the revert above; it is the evidence for the F2 dispute.
- fix-consistency, LOW: no doc or test records the F2 behaviour. Moot after the revert, since nothing changed.
- fix-proposal, LOW: F3 may have left another unquoted site stale. Its own report says the grep was not run; fix-affected-set reran it and found only dated research records, so the finding is not confirmed and nothing changed.
- fix-code-correctness: no findings. Its F2 trace assumed `retire()` removes the worker, which `hasPending` contradicts; I took the consistency lens's reading, which I checked against `deliver.go`.
- fix-affected-set: no findings. It could not say whether a test pins the F2 return; moot after the revert.

Every fork returned a report.
