# Webster and plan fix honour parent decisions; full halt reasons

A parent's recorded decision now reaches the places that halted a run without it, and halt reasons no longer lose their findings.
Three halts forced a parent to hand-edit the plan or the report to get past a call it had already made.

## What changes

**Create check reads the decision record.**
A decision that drops a Create target now carries it: `lym loom decision add --drops-create <target>` (repeatable) writes a `Drops Create:` line into the entry.
`internal/planglyph` skips the done-check for each dropped target and reports it as a warning naming the card, the target and the decision, instead of blocking.
A dropped target that a card declares through a `plan:` handle loses that declaration too, so binding owns no handle for it.
The flag applies to one entry, and a call that records several entries, or a target that is empty or holds a backtick or line break, is refused.
`record-batch` and `recover-batch` give the same way forward through `createTargetWayForward`:
amend the target, have the parent record a dropping decision, or have `recover-batch` build the target as named.
The drop is offered only for an unresolved Create target, never for a Rename pair that did not happen or a signature mismatch.

**Drift halt names a full report.**
`internal/websterengine/verifygate.go` no longer drops findings silently at 400 bytes.
A cut halt reason names the verify-gate report that holds every finding, and the drift entry's way forward adds the alternative to fixing it:
the parent accepts intended drift with `lym drift amend <slug> --entries <file>`, then runs `lym loom resume`.

**Plan amendment passes the round check.**
In `internal/burlerengine/roundcheck.go`, a finding located in a plan card file that an `--amends-plan` decision names is exempt from the changed-file rule.
Webster's halt reason is written to a halt-reason file in the conflict directory on every Stuck and removed on Done.
A plan review the run returns to reads it, so a new BLOCKING `origin: webster` finding passes the changed-file rule while a halt reason is set.
The Burler review stencils and `shedadapters` carry the reason into the review prompt.

**Later decisions supersede earlier ones.**
The batch fork's prompt and the Master template number every parent decision in record order (`{{.parent_decisions}}`).
They state that a later decision, whether a notice or a recorded entry, overrides an earlier one it differs from, even where the earlier one said to revert or drop work.
This is the part of #178 that cost a halt on a run in an earlier task.

Out of scope: the automatic rebaseline after a fix (#170, second point).

Closes #168
Closes #170
Closes #171
Closes #172
Closes #178

## Design decisions

- A drop is an explicit per-entry flag, not inferred from decision prose, so the done-check never guesses.
- `--drops-create` is recorded on a decision entry, so a parent that folds the record and re-adds its calls passes the flag again;
  the parent directive stencil says so.
- The halt-reason exemption is bounded to `origin: webster` BLOCKING findings while a halt reason exists, so it cannot excuse unrelated out-of-scope findings.
- Done-checks skip dropped targets, but drift detection and the scope guard still read the cards as given.

## To check by hand

- `lym drift amend` was not run end to end against a real drift halt; the way-forward text names it as written in `verifyGateStuckFailure`.
- A decision-record read failure in the Create check is an error with a way forward; no fixture exercises an unreadable record file on a live run.

## Merge danger

**Door:** one-way
**Blast radius:** websterengine
