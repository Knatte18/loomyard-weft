# A beam member's run names its beam orch as parent, which may amend the beam's record

`beam-member-parent` · Task · bug, batten, landing, orch, stencil, shed, loom, pawl · running · Describe

Bundled from the notes below; each part keeps its note's text.

# A beam member's run records the prime orch as its parent orch

Imported from [issue #232](https://github.com/Knatte18/loomyard/issues/232).

## Seen

In the DrillOpPlan hub on 2026-11-24, the run `models-stick-slip-map`, a member of the beam `models-torque-drag` and spawned from the beam's worktree, records the prime's orch `dop:orch` as its parent orch.
Its PR-Review sent the `[pr-review request]` for PR #52 (into the beam branch `hanf/models-torque-drag`) to `dop:orch`, as ask 1 from the PR-Review session and as ask 2 from `dop:LOOM`.
The beam's orch `dop:models-torque-drag:orch`, which owns and released the run, reviewed the PR, and its answer was refused:

```
caller dop:models-torque-drag:orch is neither the run's parent orch nor the strand the ask went to
```

The beam's dobby sends its notices for the same run to `dop:orch` too.
Knatte18/loomyard#231 shows the same gap in the tackler and driver prompts of a beam member.

## Expected

A beam member's run records the beam's orch as its parent orch, so its parent-review asks, notices and escalations go to the beam's orch and that orch's answer is accepted.

## Workaround

The prime orch forwards each ask and notice to the beam's orch, and runs `lym loom parent-review answer <slug> --ask <n> --satisfied` in the task worktree on the beam orch's verdict.


# Tackler and driver prompts of a beam member say no parent is recorded

Imported from [issue #231](https://github.com/Knatte18/loomyard/issues/231).

## Seen

In the DrillOpPlan hub on 2026-11-10, the shed tackler of run `models-backend`, a member of the beam `adopt-models-sim`, got a prompt whose section "Your parent" read:

```
No parent is recorded for this run.
When you are blocked, write your report and stop.
```

The run had a parent: the beam's orch `dop:adopt-models-sim:orch`, which owned and released it.
Other tackler and driver prompts in the hub carried the same text, in the form "No parent is recorded for the run, so the question is written for whoever owns it."
The tackler escalated on a merge conflict and stopped; the prime orch saw the stop and passed it to the beam orch by hand.

## Expected

The tackler and driver prompts of a beam member name the beam's orch as the parent, so a blocked tackler or driver reaches it directly.

## Workaround

The prime orch forwards a beam member's stop to the beam's orch by creel.


# Beam orch cannot amend its own beam's decision record

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




## From issue #252

Imported from [issue #252](https://github.com/Knatte18/loomyard/issues/252).

A beam orch received a PR-Review pause notice for a task of its beam, naming 'lym tender resume <slug>' as the way forward. Run from the beam orch's session, 'lym tender resume <slug>' was refused: 'pause: an agent may pause, resume or revive only its own pair; way forward: ask your parent session'. The beam orch is the task's parent. Workaround: the prime orch ran the resume. Expected: the parent orch of a task, beam orch included, may resume its tasks' tender pauses, or the notice names a way forward open to it.

## From issue #251

Imported from [issue #251](https://github.com/Knatte18/loomyard/issues/251).

A task's PR-Review agent paused its own pair with lym tender pause before a push. The beam orch, which parents that task, then could not lift the pause: lym tender resume <task-slug> from the beam orch's session is refused with 'an agent may pause, resume or revive only its own pair'. Mail from the beam orch to the paused agent is not delivered while the pause holds, so the run stays stuck until the prime orch runs the resume.

Expected: the orch that parents a run may resume that run's pair, as it is the session that settles the run's halts.