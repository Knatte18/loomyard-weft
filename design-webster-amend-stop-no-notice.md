# Webster stops after an amended plan send the parent no notice, and webster reset leaves the run tied to the dropped code tip

`webster-amend-stop-no-notice` · Task · bug, webster, loom · running · Describe

Imported from [issue #207](https://github.com/Knatte18/loomyard/issues/207).

## What happened

Run `models-torque-drag-adapter` (tapestry, under autonomy) went `blocked` at Webster after a parent decision with `--amends-plan` and a replan.
`lym loom status` gave:

> webster: the auto-rebaseline of the changed plan blocks this run: webster: the edited plan changes the cards of a batch this run already begun: 00-overview.md changed outside its Card Index; ...

The parent orch got no creel mail for this stop.
Its last mail from the run was the arbiter's "settled the review escalation at Plan-Bouncer and resumed the run" (12:21 UTC); the block came later, and only the operator noticed it.
Every other stop of the same run (question stops, batten failures) was mailed by `dop:LOOM` or `dop:BATTEN`.

## Two defects

1. A Webster auto-rebaseline block sends no stop notice to the run's parent. Under autonomy the orch must hear of every stop, so this one stalls the run silently.
2. The Plan rows that apply an `--amends-plan` decision edit `00-overview.md` outside its Card Index (here, the plan's opening summary) even when batches have already begun, and the plan gate accepts that, but Webster's rebaseline then refuses it. The plan rows should keep to the Card Index after a begun batch, or Plan's gate should refuse the edit, so the two agree.

3. After that, the run blocked again because batch 03 was dead from before the amendment. It named `lym webster reset --to batch-start --batch 03` as the way forward.
   That reset moved the code branch and origin back to e1ab782 and committed `webster: reset` to the records with `Warp-SHA` but no `Run-Transition` trailer.
   The run's last transition still named the dropped tip 9fff222, so `lym loom resume` refused ("the code tip 9fff222 the run's last transition names is not on origin yet"), and `lym loom goto --to Webster` wrote no transition either.
   Past `runfsm.CodeTipGrace` the dobby would have returned the run to its previous transition.
   `webster reset` (and `goto`) should commit a transition naming the new tip.
   None of these stops sent the parent a notice either.

## Fix applied by hand

The orch restored the summary paragraph in `00-overview.md`, kept the Card Index line, committed it, and ran `lym loom resume`.
After the reset it committed an empty records commit carrying `Warp-SHA: e1ab782…` and `Run-Transition: models-torque-drag-adapter`, pushed it, and ran `lym loom resume`; Webster then ran.
