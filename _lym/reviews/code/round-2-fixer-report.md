# Round 2 fixer-report

## Changes

- F1 (MEDIUM), fixed as proposed, commit `506acd0f`:
  - `HoldsReexec` in `internal/hibernateengine/carrier.go` now returns `c.running && c.phase != PhaseWaiting`, so the hold lasts until the goroutine ends after the lift and the notice.
  - The `HoldsReexec` doc comment and the carrier paragraph of `internal/hibernateengine/doc.go` say the hold lasts until the sequence has lifted its entry and sent its notice.
  - The last assertion of the first `TestCarrier` subtest in `carrier_test.go` is now a poll with a ten second deadline for `HoldsReexec` to turn false, which removes the race with `waitTerminal`.
- Self-check follow-ups, one commit after `506acd0f`:
  - New `TestCarrier` subtest `HoldsReexec keeps holding after the request turns terminal until the sequence has sent its notice`, with an `onNotify` hook on `fakeWorld` that blocks the notice, so a hold that drops at the terminal write fails it.
  - `internal/beamcli/doc.go` names that the hold lasts until the notice is sent.

## Self-check

- fix-code-correctness, MEDIUM: the poll alone does not fail with the defect in place. Fixed with the blocked-notify subtest above.
- fix-code-correctness, LOW: `beamcli/doc.go:147` does not mention the hold outlasting the terminal write. Fixed, merged with the fix-consistency finding.
- fix-consistency, LOW: same finding as above. Fixed.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
