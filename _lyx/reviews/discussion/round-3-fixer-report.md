# Fixer report — discussion round 3

## Changes

- **F1** (fixed as proposed): D4 now gives one ordered account.
  The darn writer leaves the own-entry step out from the start, because the run lock refuses the edit anyway, and asks its parent to record the drop.
  Until the record carries the `Added after Discussion` entry, it names the omission as an unrecorded parent correction, which the darn-review rubric files as a MEDIUM `decision` finding deferred to the parent (checked against `contracts/stencils/loom/loom-rubric-darn-review.md:51-54`).
  Once the entry exists, the omission is no deviation.
  It stops and reports only with no parent to ask.
  The closing sentence on own-entry steps only is kept.
  D5's darn-review sentence now matches, and D4's rationale adds that the darn route leaves the step out at once where the Plan route waits, because a darn run is one autonomous session that cannot wait on a reply.
- **F2** (fixed as proposed): the Goal now says the entry's `run_settings` are outside the own-entry edit, and that scope means the other fields the run lock compares, given as examples (checked against `scopeChanged` in `internal/boardengine/runlock.go`).
  The acceptance criterion that opens "Every new rule is scoped to the body or scope" now lists a `run_settings` write as allowed.
  The stencils keep the phrase "body or scope"; the record orders no field list into them.
- **F3** (fixed as proposed): D1's sentence ordering the `CLAUDE.md` rule now ends with "because the run's board lock refuses such a change".
  The rationale paragraph is unchanged.

## Disputed

None.

## Deferred

None.
