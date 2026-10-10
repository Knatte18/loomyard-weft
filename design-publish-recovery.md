# Publish and loom recovery: a fix route from Publish, resume past its own merge, clean re-steps

`publish-recovery` · Task · bug, loom, burler, fabric, config, stencil · running · Webster-Bouncer

Depends on: [`gate-test-scope`](design-gate-test-scope.md), [`git-robustness`](design-git-robustness.md)

## Goal

- A blocked Publish has a route that hands its failure to a fixer as a finding (#462), so the hub never falls back to `goto Plan-Bouncer` for one fix.
- `lyx loom resume` at Publish recovers from the staged merge-in a failed merge-in left behind (#471), and the dirty-tree refusal names `lyx fabric merge --abort` when a merge is in progress.
- Finalize warns only of per-worktree config changes the task made, never of changes that came from main.
- A loom re-step is not refused while a stopped review strand's run dir is younger than 3 minutes (#452).
- A plan agent cannot record a decision as `--by parent` (#453).

The merged notes keep their own Seen sections below.

## From `publish-fix-route`

Imported from [issue #462](https://github.com/Knatte18/loomyard/issues/462).

Seen 2026-10-08 in `shuttle-send`: Publish blocked three times on the same `publish_verify` failure (three `//go:build tmux` loomcli tests), and the two fix routes the hub has from a blocked Publish never reached the defect.

The defect arrived through the merge of main (reed-view typed `/color` raw into the pane), so it is outside the task's diff.

- `lyx loom decision add --by parent` plus `lyx loom goto --to Webster-Burler`: the Webster-Review rubric measures against the plan, not the decision record, and the reviewer reads only `git diff $(git merge-base main HEAD)..HEAD`, so both rounds approved with unrelated NITs and the decision was never acted on.
- The Webster-Burler verify gate compiles the `tmux` tier and never runs it (`internal/loomshed/doc.go`), so the fixer never saw the failing tests either.
- `lyx loom reject` works only at PR-Gate or PR-Rework, and `goto` refuses PR-Rework from Publish because it lies past the current row, so the hub cannot hand the failure to a fixer as a finding.

The hub fell back to `goto Plan-Bouncer`, so Plan-Review raises the uncarried decision against the plan, which re-runs Plan-Burler, Batchifier and Webster for one fix.

Expected: a blocked Publish (or any blocked row after Webster) has a route that hands a named finding to a fixer round, e.g. `lyx loom reject` accepted at a blocked Publish, or a Webster-Burler round that takes the publish_verify failure and post-Discussion parent decisions as its focus.

## From `publish-resume-own-merge`

Imported from [issue #471](https://github.com/Knatte18/loomyard/issues/471).

## What happened

After a `Publish` step failed mid merge-in (merge staged, conflict resolved by the agent, then a producer refusal), the loom driver re-stepped once.
The run then halted as `blocked` at `Publish`, anomaly `escalation-to-human`, with this reason:

`the task worktree has uncommitted changes before the merge-in: <a long list of files, plus "and 63 more">; commit or remove them on the task branch, then resume`

The listed files are the changes the staged merge-in itself brought in, not the task's own uncommitted work.

## Why it is abnormal

The merge-in precondition check treats the worktree state produced by the earlier, interrupted merge-in attempt as foreign uncommitted changes.
A re-step or resume after a partial merge-in can therefore never pass the check, and the run escalates to a human although all earlier segments (Discussion, Plan, Webster, Describe) were `done`.
The advice in the message ("commit or remove them") would either commit an unfinished merge or discard the resolution.

## What to do

- Detect an in-progress merge (`MERGE_HEAD` present, or merge-staged state recorded in the trace/history) in the `Publish` precondition and resume it, instead of applying the clean-worktree check.
- Alternatively provide a `lyx` repair verb that completes or aborts a staged merge-in, and name it in this refusal message.
- Make the loom driver's repair step aware of this state, so it does not re-step into a guaranteed refusal.

Related: the transient `object not found` report, which is what first interrupted the merge-in.

Provenance: task `binary-change-and-rules`, from `loom-halt.md` and `driver-repair-1.md`.

## From `finalize-config-warn-from-main`

Imported from [issue #450](https://github.com/Knatte18/loomyard/issues/450).

At Finalize, loom warns: `task branch "<slug>" changed per-worktree config files since it forked from "main": _lyx/config/batcher.yaml (base <sha>, tip <sha>); landing does not carry them to the parent branch, so re-apply each change meant for the parent on the parent's copy`.

Seen for `shuttle-state` and `plan-glyph-precision` on 2026-10-08.
In both, the pair's `batcher.yaml` was byte-identical to the prime's: the change was main's own commit (`batcher: cautious is the template default`, c0bf19c54) arriving in the pair through the weft sync, not an edit the task made.

Expected: the check compares the tip against the parent's current copy (or excludes changes the weft sync brought in), and warns only when the task's copy differs from the parent's.
As it is, the warning fires on every run that forked before a config change on main, and the hub must diff each time to rule it out.

## From `restep-young-run-dir`

Imported from [issue #452](https://github.com/Knatte18/loomyard/issues/452).

## What happened

After a driver repair, the re-step of the Discussion burler segment failed with a non-transient `producer` error.
The trace line: `shed: step refused kind=producer transient="" error="... probe the burler-review half: shuttle: attach: 1 candidate run dir(s) ... cannot be confirmed dead or alive (untracked or with a cleared pane binding, younger than 3m0s) ..."`.
The run dir had been left by the review strand that the previous step itself stopped.
`lyx reed status` listed no such strand, while the burler-fix strand was still live, and no fabric mutation was half-finished.
The driver's only remedy was to wait until the run dir passed the 3m age and re-step, which cost a repair round (history_length 6).

## Why it is abnormal

The strand was stopped by lyx moments earlier, so its run dir is known dead, yet the burler probe treats it as unclassifiable and refuses the step.
A refusal that resolves itself with time should be a transient error, not a `producer` error that consumes a repair.

## What to do

- Have the stop path record or clear the stopped strand's pane binding, so the probe can classify its run dir as dead immediately.
- Alternatively, report the young-run-dir refusal as transient so the loom retries instead of halting for repair.

Provenance: task `plan-glyph-precision`; note `driver-repair-20261008-discussion-burler-2.md`.

## From `plan-agent-parent-decision`

Imported from [issue #453](https://github.com/Knatte18/loomyard/issues/453).

## What happened

In the `shuttle-send` run, the Plan-Write agent recorded a decision as the parent:

```
lyx loom decision add --by parent --title "PATTERN-shuttle-stop names the loom driver exception" ...
```

The Plan role writes its own output files only; a design call outside the decided scope goes to the parent.
`lyx loom decision add` accepted `--by parent` from inside the plan agent's session, so the record reads as the hub's decision though the hub never made it.

## Expected

`decision add --by parent` refuses when it runs inside a lyx-spawned agent session other than the parent's, or the Plan stencil's deny hooks block it, so a plan agent's scope call reaches the parent as an escalation or an open question in the plan.

## Context

The decision itself was narrow and the hub would have taken it (it names loomcli's raw removal of the loom driver's strand as an exception in the new PATTERN entry).
The defect is the forged authorship, not the content.
