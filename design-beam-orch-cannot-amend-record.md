# Beam orch cannot amend its own beam's decision record

`beam-orch-cannot-amend-record` · Note · bug, loom, orch, pawl

Imported from [issue #236](https://github.com/Knatte18/loomyard/issues/236).

## What happened

In hub `dop`, beam `models-torque-drag` (P1) holds a stored decision record.
On 2026-11-24 the operator made a decision that supersedes two of its decisions: a new member task maps the well's formation strength into the Models bit-rock law, where the beam record's D1 and D2 keep Models' bit-rock defaults.
The member's Discussion saw the conflict and asked the beam orch, which owns the beam record, to settle it.

The beam orch could not amend its own beam's record:

- `lym loom decision add models-torque-drag --by parent ...` from the beam orch refused: `the caller "dop:models-torque-drag:orch" is a task session; way forward: a design call after the Discussion is the parent's to record; ask dop:orch to run the verb`.
  The beam orch is an orch, not a task session, and it is the record's owner.
- `lym board upsert --record-file` refuses an entry a run holds, and a beam's project is always held by its pawl run (`running · Beam-Await`), so that way is closed for the beam's whole life.

## Expected

The beam's own orch can append a decision to, or replace, its beam's decision record while the beam lives, as a run's parent can for a task's record.
The caller check counts a beam orch (`<hub>:<beam>:orch`) as an orch, not as a task session.

## Related

The same misclassification of the beam orch shows in parent-review: `lym loom parent-review answer` refuses the beam orch for its member's pull request (`neither the run's parent orch nor the strand the ask went to`), filed by the hub orch as Knatte18/loomyard#232.
