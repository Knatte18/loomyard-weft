# Round 3 fixer report — discussion

All five findings are fixed in `_lym/discussion/decision-record.md`.
`_lym/discussion/support-log.md` is unchanged.

## Changes

- **F1** (fixed as proposed): D6's slug paragraph takes `<N>` from three sources: the board, the run records, and the task branches fabric named for a `<prefix>-t<N>` slug, local or on origin.
  The paragraph cites `doneSlugRefusal` for a torn-down pair keeping its branch after `lym batten cleanup --apply`.
  The guarantee now reads over the three sources and also covers the create row's leftover-branch refusal.
  The D6 acceptance criterion adds the case where cleanup removed the landed first fix's run directory.
  The auto-mode assumption names the task branches.
- **F2** (fixed differently in part): D6 has a new bullet for the subject's own fix task while it stands open, with one more acceptance criterion under D6.
  The bullet covers the empty status, `running`, `paused`, and `awaiting` with an escalation the arbiter would admit.
  In those states the check files none and removes none, and its notice names the task, its state, its hand step, and that its landing starts the next check.
  - The proposed hand step for `paused` was `lym tender resume <slug>`. That lifts a pause-record entry, not a `paused` run state.
    The bullet names `lym batten run <slug>` from the prime instead, which resumes a paused run silently (`internal/battencli/arm.go:586`, and the `lym batten run` resume text in `haltedOverDoneChild`).
  - The empty status is added because it is the other open state the removal bullet does not reach.
  - Bare `awaiting` is left out because F4 makes it halted.
- **F3** (fixed differently in part): D6 names the store of the fix-try count, `fix-tries.json`.
  It sits beside the holds file below `fabricengine.AdoptScratchDir` and is written through `internal/state` under its own lock, and the check record's writes never touch it.
  A filing adds one between D8's create and release writes.
  A hand `lym adopt` resets the count before starting the check, so the check never has to tell a hand-started check from one a landing started.
  D5 line 97 and the D6 acceptance criterion now match, and there is a new auto-mode assumption.
  - The proposal said "only a hand `lym adopt` resets it". D5's green-end reset is kept as well: without it, a check started by a hand removal after a green end inherits a spent count (self-check, fix-proposal finding 3).
  - The proposal said the add happens "before the task is released". The add is placed after the create write, so a create that is refused spends no try (self-check, fix-proposal finding 2).
- **F4** (fixed differently in part): D6's halted removal, D7's `halted` state and the D7 criterion now count `awaiting` as halted.
  This holds only when no review escalation the arbiter would admit is pending. `decideDrive` in `internal/sheddriver/driverules.go` drives an awaiting stop with `EscalationPending` on through the arbiter, so that case stays under F2's bullet (self-check, fix-proposal finding 1).
- **F5** (fixed as proposed): the D3 acceptance criterion now matches D4's bound.
  Held runs start by themselves only at a green check. The only hand lifts are `lym tender resume`, bare or by slug, and `lym batten cancel <slug>`. No release verb and no reed button exists.

## Self-check

- fix-claims, MEDIUM: the `paused` way forward, "the pause reason its run record carries", names no step. Fixed with own change: the bullet names `lym batten run <slug>` from the prime.
- fix-way-forward, MEDIUM: duplicate of the fix-claims finding. Fixed by the same change.
- fix-consistency, LOW: D6's opening sentence and the tries paragraph disagreed on when the count is added. Fixed as proposed: both now say between D8's create and release writes.
- fix-proposal, MEDIUM: `awaiting` with an admissible escalation is still being driven by the arbiter. Fixed as proposed: it is halted only without one, and running-like with one.
- fix-proposal, LOW: an add before the create spends a try on a refused create. Fixed as proposed: the add comes after the create and before the release.
- fix-proposal, LOW: the removed green-end reset leaves a spent count after green. Fixed as proposed: the D5 green-end reset is restored, and D6 and the D6 criterion name both resets.

## Disputed

None.

## Deferred

None.
