---
dispositions:
  - id: F1
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
---

F2 is `decision`-class and held by the parent; this report leaves it out.

## Changes

- F1, F5, F7 (one commit, they share `crashresume_test.go`): `Runner.start` keeps the fresh spec, warning and copied flag beside the resumed ones.
  When a resumed attempt fails and its strand is removed, it sets both runs aside and switches back to the fresh values with `resuming` false.
  The retry then starts a fresh session, and the success path sends the carried-instructions notice.
  Added `TestStart_FailedResumeRetriesAsAFreshSession`, a carried-instructions row in `TestStart_ResumesAnInterruptedRunsSession`, and an unwritable-record run in the retire test with its joined-error and count assertions.
- F3: added `TestAttach_RecordsASupersededRunningCandidateAsInterrupted` in `attach_test.go`.
- F6: `TestStop_RecordsTheStopBeforeRemovingTheStrand` asserts `Interrupted` is false.
- F4: rewrote the clause in `contracts/orchrole/orch-role.md`, and, from the self-check, the twin comment in `internal/loomcli/rownotice.go`.
- F8: rewrote the header of `internal/loomcli/gotoresume_test.go` to say what the test pins.
- F9: widened the blast radius in `_lym/landing/summary.md` and listed the new tests in its `## Tests` section.
  The `_lym` file is edited only; no git command touched it.

`lym gate test ./internal/shuttleengine ./internal/loomcli` passed (tier0: 3398 passed, 0 failed) before the commits.
The later `rownotice.go` edit is a comment change and was not re-gated.

## Gate repair

The comment gate refused the doc comments of `TestAttach_RecordsASupersededRunningCandidateAsInterrupted` and `TestStart_FailedResumeRetriesAsAFreshSession`:
one wrapped mid-sentence and three lines reached 120 characters.
I rewrote both comments as one sentence per line, each under 120 characters, and checked every comment line this round added.
No test code changed.

## Self-check

- fix-code-correctness: returned no findings.
- fix-proposal: returned no findings.
- fix-consistency, MEDIUM: `internal/loomcli/rownotice.go:49` still said a resumed halt spawns a fresh agent. Fixed as proposed.
- fix-consistency, LOW: the summary's `## Tests` list missed the new tests. Fixed as proposed.
- fix-affected-set, LOW: duplicate of the `rownotice.go:49` finding. Merged and fixed with it.
- fix-affected-set, LOW: `docs/overview.md:1510` repeats the clause, but it states only when the row-reached mail is sent, which still holds. Read and left unchanged; no stale claim.

## Disputed

None.

## Deferred

None. F2 is the parent's.

## Rulings

None reached me this round.
