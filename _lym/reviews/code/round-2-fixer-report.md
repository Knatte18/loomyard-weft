## Changes

- F1 (fixed as proposed), commit `96c00439`: in `TestRunNotice_OncePerEpisode`, the child's status time is rewritten (after a clock advance) just before the restart, so the restarted dobby sends no second notice only when it reads the episode's `Since` back. In `TestRunNotice_APIErrorRevive`, a `restartAfter` field and the row `ARestartBetweenThemKeepsTheCount` restart the dobby after the second revived episode and expect one notice on the third.
- F2 (fixed as proposed), commits `b2673de3` and `d438edb9`: the int, int64 and float64 switch of `runShedDuration` moved into `durationFromCount`, which `runShedDuration` calls. `TestDurationFromCount` in `internal/beamcli/dobby_test.go` has rows for a positive int, int64 and float64, zero, a negative count, a zero float, an absent value and a non-number. The file header names the new concern.

- Gate finding (fixed): the comment above the restart in `TestRunNotice_OncePerEpisode` was wrapped mid-sentence. It is now two sentences, one per line, in a follow-up commit `dobbyengine: fix F1 — break the restart comment at sentence boundaries`.

`lym gate test ./internal/beamcli ./internal/dobbyengine` passed before the header and comment edits, which change comments only.

## Self-check

- fix-consistency, LOW: `dobby_test.go` file header did not name the fallback test. Fixed as proposed, commit `d438edb9`.
- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-proposal: no findings.

## Disputed

None.

## Deferred

None.
