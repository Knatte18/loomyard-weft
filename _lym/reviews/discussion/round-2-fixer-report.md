---
dispositions:
  - id: F8
    disposition: fixed
---

# Round 2 fixer-report

## Changes

F8, fixed as proposed with option (a), in `_lym/discussion/decision-record.md`:

- D5 gains a Decision line: every lym verb outside `lym config` that loads the loom or tapestry config runs `hubreconcile.EnsureOrWarn` before that load, as the start verbs do today.
  The `lym config` verbs are left out because they are D4's ways forward, and `lym config reconcile --apply` reconciles its own layer.
  `EnsureOrWarn`'s warning names re-running the verb once the walk ends.
- D5's rationale no longer claims the reconcile runs "before any command runs".
  It now says the walk runs before the first load on a new build.
- D5's second Bound line now says the walk runs at the first verb that loads either config.
  It covers files the walk has not reached on a new build: a mid-merge worktree, and an `EnsureOrWarn` that failed or waited out `DefaultLockWait`, where re-running the verb loads the moved file.
  It still covers files that gain an old key after the walk.
- A new D5 Bound line names the cost while the stamp stays stale: every such verb reruns the walk.
- A new D5 acceptance criterion: after a rebuild that leaves the stamp stale, a non-start loom verb whose hub base and local `loom.yaml` hold `code_max_bounces: 2` resolves `loom.code.max_bounces` to 2 and does not halt.
- D7's body and its criterion say "the next walk" instead of "the next start verb's walk".
- Two plan-writer notes: the widened `loomcli` `arm` gate and the other loaders it reaches, and the change to `reconcileWayForward`.

## Self-check

- fix-consistency, LOW: D7 criterion still said "next start verb's walk". Fixed as proposed.
- fix-consistency, LOW: "the first verb of any kind" overstated the rule. Fixed as proposed ("the first such verb"); merged with fix-claims' twin.
- fix-claims, LOW: "the call costs one stamp read" was inaccurate. Fixed differently: the line now says the call costs the stamp read and board file checks a start verb pays today, instead of listing each check.
- fix-claims, LOW: "of any kind" overstated the rule. Merged with fix-consistency's twin; fixed.
- fix-proposal, MEDIUM: a stale stamp makes every loading verb rerun the walk. Merged with fix-way-forward 1. Fixed: a D5 Bound line names the cost and when it ends.
- fix-proposal, MEDIUM: a lock timeout behind another verb's walk still halts. Merged with fix-way-forward 2. Fixed: the bound names that case, and `EnsureOrWarn`'s warning now names re-running the verb once the walk ends, with a plan-writer note on `reconcileWayForward`.
  An unbounded lock wait was not chosen, since the walk's board pull has no time bound of its own.
- fix-proposal, LOW: the verb set reached `lym config` and the shuttle loaders. Fixed: the `lym config` verbs are excluded, and the note names `webstercli` and `shuttleparity`.
- fix-way-forward, MEDIUM: retry cost unbounded in the record. Merged with fix-proposal 1; fixed.
- fix-way-forward, MEDIUM: lock-timeout way forward unnamed. Merged with fix-proposal 2; fixed.
- fix-way-forward, LOW: `reconcileWayForward` names start verbs alone. Fixed with a plan-writer note and the D5 line on the warning's way forward.

## Disputed

None.

## Deferred

None.

## Rulings

None.
