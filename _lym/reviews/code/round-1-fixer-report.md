---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "Named the outward push and widened the blast radius, but wrote it as the one word `runs` rather than `webster` plus the run lifecycle, since the change-description spec asks for one word. Kept the door two-way, because the records commit already starts a detached push of the same pair (internal/fabricengine/commit.go:171)."
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
---

## Changes

- F1, `_lym/landing/summary.md`, not committed:
  `## Merge danger` names the synchronous push on the sync-failure path and the failed-push record that holds the run.
  The blast radius is `runs`.
  The proposed `webster` plus run lifecycle was not written, for the one-word rule above.
  The Change, Design decisions and Tests sections follow the F2 and F3 changes.
- F2 and F3, commit `81117f1a6`:
  - `internal/webstercli/reset.go`: a handed-over run on the sync-failure path is named first in the error,
    which keeps the sync error and drops the sync's way forward.
    `push_error` stays.
  - `internal/webstercli/reset_test.go`: `TestResetCmd_SyncFailureAndTheRecordsPush` covers a sync failing before the commit lands
    (a fabric open that fails) and a hand-over after it lands.
    The hand-over is a stubbed `pushRecords` returning `fabricengine.ErrHandedOver`, with no staged takeover on origin.
    The rows are a test of their own, because the transition test's assertions need a landed commit.
- Self-check fixes, in two further commits:
  the hand-over refusal carries its own way forward,
  and the comments in `reset_test.go` and `beginbatch.go` match the new refusal.
  The Merge danger prose in the summary was corrected.

Verification: `lym gate test --tier 1 ./internal/webstercli -- -run TestResetCmd -v` passed after the last change, with nothing skipped.

## Self-check

- fix-code-correctness, MEDIUM, Merge danger claims the detached push already sent the commit: fixed.
- fix-consistency, MEDIUM, the same claim: fixed, merged with the above.
- fix-consistency, LOW, `fabricSyncWayForward` comment says every sync refusal carries it: fixed.
- fix-consistency, LOW, `reset_test.go` header says each refusal checks that nothing moved: fixed.
- fix-proposal, LOW, the hand-over refusal has no way forward: fixed differently.
  The sync's clause was not restored; the refusal got a clause of its own.
  The success path's hand-over refusal is unchanged.
- fix-proposal, outside its lens, the same detached-push claim: fixed, merged with the first.
- fix-affected-set, MEDIUM, the same claim, and "the records side" understating a pair push: fixed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
