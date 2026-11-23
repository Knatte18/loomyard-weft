# Deliver the PR-Review re-ask and record a refused post as undelivered

A PR-Review session that re-asked its parent orch with `lym loom parent-review ask` never reached the parent.
The verb signed the post as the loom program, and creel lets only an orch strand's pane claim a program name.
Creel refused the post, the ask was recorded `no-parent`, and the agent read that as a parent that was down and ended its turn.

Closes #200

## What changes

- `parent-review ask` signs the post as the session's own strand, through the new `orchcli.PostAsCaller`.
  A claim creel refuses as coming from outside that strand's pane is posted once more as the program.
  The PR-Review row's own ask passes no claim and still signs as the loom program.
- A post that creel refuses or fails to deliver to a parent that resolved ends the ask in a new state, `undelivered`, with the error as its reason.
  `no-parent` now means only that no parent orch resolved.
  The PR-Review ask, the Drift judge ask and the parent seats of the burler reviews all record this.
- `parent-review await` answers an undelivered ask with its reason, as it does for `not-sent`.
- `Drift-Verify` halts on an undelivered judge ask as it does on `timeout`, `not-sent` and `no-parent`.
  A PR-Review ask that ended undelivered reads as ask-again.
- The PR-Review landing block tells the agent that `undelivered` is a delivery failure, not a down parent.
  It sends the parent the reason and ask number, ends its turn, and asks again on the next mail.
  The orch role contract notes that a `[pr-review request]` can come from the run's PR-Review session.

## Design decisions

- The claim is the strand of the session that runs the verb, passed through the `askPR` seam, so the asker stays one type for the row and the session.
- `PostAsCaller` shares `postSigned` with the existing notice path, so the strand-first, program-fallback rule lives in one place.
- `undelivered` is a new `ParentAskState` rather than a new reason on `no-parent`:
  an agent branches on the state, and the reason is free text.

## To check

- A place that lists or switches over the parent-ask end states may still omit `undelivered` (merge-danger-omits-undelivered-ask-state).
  Search for the other end states, `not-sent` and `no-parent`, and check that each such place handles `undelivered`.
- The session's strand claim in `parent-review ask` is pinned by a tier-0 test of the verb and a tier-1 test over a real hub with an in-process creel daemon.
  Neither runs a real creel pane, so the claim creel accepts from a live PR-Review pane is unproven outside the fake delivery (parent-review-ask-strand-claim-untested).
  Run `lym loom parent-review ask` from a live PR-Review session once and confirm the parent orch receives it.

## Merge danger

**Door:** two-way
**Blast radius:** loom
