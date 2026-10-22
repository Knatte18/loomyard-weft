# Fixer-report, webster round 1

## Changes

- F1 (MEDIUM), commit `1c38c4e2c`: restored main's text for the nine `PATTERN.md` entries. Fixed as proposed, then cut back by the gate repair below.
- Gate repair (overview headroom: the overview grew 242 bytes past main's size, which is over the headroom line): the `PATTERN-run-process-owner` line is shortened, and four restored lines lose a clause their background file already holds: owner-machine ("bar a Finalize past its parent-side merge" and "verdicts in `ReconcileOrigin`"), glyph-conversion-chokepoint ("a ref's alphabet only `planparser`"), pause-holds-prompts ("only orch runner reads uncovered") and verified-tree ("told its scope's pause hold"). The overview is now 15664 bytes against main's 15666. This partly undoes F1's restore for those four entries, which the headroom rule forces.
- F2 (MEDIUM), commit `558b1d880`: `planRecords` gives a group unit every live process of the group as a member (`groupMembers`), so `runUnits` awaits, escalates to SIGKILL and reports them. `End` passes a `skip` predicate so the caller and its ancestors are never watched. `TestEnd` gains a row for a leader that exits on SIGTERM while a member ignores it, and a row for a zombie leader with a surviving member.
  Fixed differently inside the proposal: the `runUnits` rule "a signalled group with empty members counts as signalled" was not added. Members are empty only when every live process of the group is the caller or an ancestor, which `End` never watches, so the rule would await nothing.
- F3 (LOW), commit `558b1d880`: `signal` reports an attempted signal, failed or not, so the unit's members are awaited and a still-running one is reported. `TestEnd` gains a `signalProcess` error row.
- F4 (LOW): fixed differently. The overview stays an index and its two lines are unchanged, but the proposed header statement was not made: `PATTERN.md` is over the headroom line (`internal/pattern`), the added sentence was undeclared growth, and it overstated the overview as many entries have no background file.
- Self-check fixes, follow-up commit: the `PATTERN.md` header sentence removed; `end` reports a process that survives several passes once; `internal/runproc/doc.go` describes group-member watching.

## Self-check

- fix-consistency, LOW: `doc.go` omits group-member watching. Fixed as proposed.
- fix-code-correctness, LOW: a survivor of one pass can be listed again in a later pass. Fixed as proposed, by deduplicating survivors in `end`.
- fix-code-correctness, LOW: a failed signal on an exiting process waits the full grace. Not changed, for lack of a sound alternative: the wait is the intended path for a live process whose signal fails (EPERM).
- fix-code-correctness, LOW (in the same finding as the first): a live process with a stale record in a signalled group stays unwatched. Not changed: it predates this round and the group signal still reaches it.
- fix-proposal, MEDIUM: the `PATTERN.md` header sentence exceeds the overview headroom. Fixed as proposed, by removing the sentence.
- fix-proposal, LOW: the header sentence overstates the overview. Resolved by the same removal.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

Checks run: `lyx gate test ./internal/runproc ./internal/pattern` green. `lyx gate test --upto 2` was not run.
