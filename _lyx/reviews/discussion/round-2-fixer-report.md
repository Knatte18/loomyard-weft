# Fixer report — discussion, round 2

## Changes

- F1, fixed as proposed, with the owner moved into a new decision:
  D7's third bullet now says the #673 entry is removed with the fix of #673 as D9 assigns it.
  New D9 gives the Merge-Review perch the check after every merge-in, raising a #673 entry in `ORCHESTRATOR.md` as a finding when the parent's `CLAUDE.md` no longer holds the workaround.
  Its rationale notes the perch's overlap skip does not apply, since the parent's removal touches `CLAUDE.md`, which this task changes.
  For the other landing order, the board-note follow-up leaves Open risks and becomes a D9 card that writes the note with `lyx board upsert`, worded for either order.
  Scope gains the board note; the Acceptance criteria name the perch as owner and add the note; the Open risk keeps one line pointing at D9.
- F2, fixed as proposed: D1's first bullet defines a recorded operator decision (the task's board entry, an operator's board note, a `--by operator` decision-record entry, a Discussion answer the support log's `## Interview` shows the operator gave) and states that `--by arbiter` or `--by parent` entries, auto-picks and Auto-mode assumptions never count.
  The brief's settle condition states the definition, D1's Bound repeats the exclusions, and the brief's Acceptance criterion names the definition.
- F3, fixed as proposed: D1's list of brief changes gains a bullet rewording step 1 to record the applied operator decision, citing its source, with `--by parent`, or to name it and its source in the report when no decision record exists.
  Scope and the brief's Acceptance criterion now include step 1.
- `support-log.md`: a `### Round 2` entry under `## Review rounds`.

## Disputed

None.

## Deferred

None.
