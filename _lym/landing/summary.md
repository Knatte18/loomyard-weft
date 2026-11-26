# webster: let Master's friction note and gate fixes clear the audit

Webster's Master writes the friction note its prompt names, and the run-exit audit flagged that write as `parent-write`.
The run blocked with every batch done.
`lym webster accept-audit` then refused, both because of a verify fixer's commit above the last batch head
and because the note lies on the records side, outside the task worktree.
The only way forward was `lym webster reset --to start`, which discards every batch.

## What changes

- The parent audit lets Master write the friction note its prompt names.
  Any other note path is still a `parent-write` finding.
- `accept-audit` can now check a suspect path on the records side.
  The path clears when it is unchanged since Master spawned, absence included.
  A changed one refuses and names the records-side commit to restore it to.
- `accept-audit` accepts the commits of a verify-gate fixer the gate passed at, as it accepts a clean parent merge.
- A restarted run's gate takes over the earlier run's pre-fix head while that head still bounds the fixes.
- The Master stencil and the orch role's limits table name the friction note beside the two contract files.
  The verify-fix stencil says the listed commits may be hand commits.

## Design decisions

- **The note path is saved, not recomputed.**
  `State.MasterNotePath` holds the path the prompt named at spawn, matched in its told and link-resolved spellings.
- **Records-side evidence is a baseline taken before the spawn.**
  `State.MasterRecordsBase` is the records side's HEAD from before Master could write.
  Without it, and for webster's run, reports and plan directories, a path still clears only through the reset route.
- **Fix commits are accepted as a recorded range.**
  A passing gate with HEAD above its pre-fix head records `State.FixedBase` and `State.FixedHead`.
  `accept-audit` accepts that range only when nothing but clean parent merges lies between the last batch head and `FixedBase`.
  A commit above `FixedHead` other than a clean parent merge still refuses, and a reset clears the range.
- **An earlier pre-fix head is kept conditionally.**
  It must be on HEAD's history, with no batch landed above it and every commit above it passing the gate's commit check.
- A `state.json` written before this change has none of the new fields, so it gets none of the new tolerances.

## To check by hand

- The commits `accept-audit` tolerates may include unvetted ones.
  The `FixedBase`..`FixedHead` range (`recordGatePass` and `earlierPreFixHead` in `verifygate.go`,
  `reconcileAcceptHead` in `auditledger.go`) covers whatever passed the gate's commit check, hand commits included.
  That check admits any non-merge commit.
- Saving and reading the Master note path may be under-tested.
  Check that the tests cover the save in `runlevel.go` and the reads by both the record-batch and run-exit audits.
- The Master stencil may still limit Master to its two contract files somewhere (`webster-template-master.md`).
- A restarted gate may lose the pre-fix head (`earlierPreFixHead` and `newVerifyGate` in `verifygate.go`).
  Check the cases where it is dropped, and that `FixedBase` then does not hide or strand earlier fix commits.

## Merge danger

**Door:** two-way
**Blast radius:** webster
