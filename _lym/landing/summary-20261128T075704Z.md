# webster reset already pushes its records synchronously; no change

Issue #293 reports that `webster reset` only commits its records and leaves the push to the background pusher,
so a failed push of the reset's records commit writes no record and does not hold the run.
The code on the base no longer behaves that way, and this change makes no edit.

## What the base already does

- `internal/webstercli/reset.go` pushes the pair's records itself once the reset's records commit is made,
  through `pushPairRecords`, which waits for a push of the pair already running.
- A push that fails, an owed one on an unreachable origin included, is reported in the verb's `push_error` field
  and leaves a failed-push record naming the run, which holds the run at its transition.
- A push that finds the run taken over by another machine fails the verb.
- `TestResetCmd_RecordsTheRunTransitionAtTheNewTip` in `internal/webstercli` pins this:
  its `UnreachableRecordsOriginLeavesAnOwedRecord` row checks both `push_error` and the owed record.
- `pattern/PATTERN-pair-remote.md` lists the reset's records push among the pushes that stay synchronous,
  and `pattern/PATTERN-run-fsm.md` names its owed record as the one that still sets `PushOwed`.

## Design decisions

- **No bug is left to fix.** The issue describes the code partway through the work that landed as `3fb0787e3`;
  that commit's final form pushes the reset's records synchronously and records a failure.
  A test that fails before a fix cannot be written, so none is added.
- **The plan-review coverage check the issue suggests is left to its own change,**
  which covers checking plan and review claims against the code.

## Merge danger

**Door:** two-way
**Blast radius:** none
