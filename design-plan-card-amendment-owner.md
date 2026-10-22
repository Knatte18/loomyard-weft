# No role may carry out a parent decision that amends a plan card mid-run

`plan-card-amendment-owner` · Note · enhancement, webster, orch, loom

Imported from [issue #806](https://github.com/Knatte18/loomyard/issues/806).

## What was seen

In the quarry hub, run `csharp-alphabet` blocked at Webster on a plan defect (see #805).
The arbiter handed on, and the orch recorded a parent decision to amend card 9 and rebaseline.
No role could carry the decision out: the driver edits no plan files, Webster's master edits only its contract files, the recovery strand will not edit the approved plan, the orch role forbids editing `_lyx/plan` in a task pair, and `lyx webster rebaseline` says the operator names the cards.
The run sat parked until the operator was asked, and the operator objected that the agents should have fixed it themselves.

## Workaround

On the operator's order the orch edited card 9 and ran `lyx webster rebaseline --card 09` and `lyx loom resume`.
A hub orch rule now lets the orch make such an edit without asking.

## Expected

A parent decision that amends a plan card has an owner inside the run, such as the arbiter, a plan-fix row or the orch by role, so the run goes on without the operator.
