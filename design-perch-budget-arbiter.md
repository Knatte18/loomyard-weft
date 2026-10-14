# Perch budgets and stops: darn budget per run like loom, arbiter only when needed

`perch-budget-arbiter` · Task · enhancement, config, loom, shed · running · Batchifier

These two notes are one task because both concern how a review perch's budget and stops are configured and settled in shed.

## Darn review budget overridable per run (darn-run-budget)

## Problem

Darn's `review_max_bounces` and `review_hard_max_bounces` are `CategoryGuard` keys (`internal/darnengine/config.go`), so `internal/configengine/layers.go` refuses them in a `.local` override and a board entry's `run_settings` cannot set them for one darn run.
The only way to change a darn run's review budget is hub-wide, in the hub's `darn.yaml`, which changes every darn run.
Loom's `review_max_bounces` is category none (minimum 1), so a loom run takes a per-run budget through `run_settings`, and loom has a separate guard key, `skip_merge_review`, for switching a perch off.

Darn guards the budget keys because `review_max_bounces: 0` skips Darn-Review and Describe: the budget key and the off switch are the same key.

Seen on 2026-10-14 with `tokencount-fork-output`: the operator wanted no review round on a small darn run, which no per-run setting allows, and `lyx loom goto` cannot move a run past a review row.

## Direction

The operator finds the difference wrong: darn and loom are both shed recipes and should read their config the same way.

- Darn's budget keys become ordinary keys like loom's, overridable per run through `run_settings`, with a minimum of 1.
- Switching the Darn review perch off becomes its own guard key, like loom's `skip_merge_review`, so the guard stays on the off switch and not on the budget.
- Check whether darn and loom can share one config shape for perch budgets and off switches, so the two cannot drift apart again.

## Arbiter spawns only on stops that need it (arbiter-when)

The arbiter (`lyx loom arbiter`, `opus[high]`, cold start) settles a stop the run's driver cannot.
On 2026-10-12 it spawned five times across three runs, and none of the five stops needed it.

## Operator's view (2026-10-12)

The arbiter is a strong agent and should spawn only when it is really needed, which should be rare.
Its purpose is to take load off the orch.
This is a view to design against, not a decision.

## Current triggers

Two filters decide a spawn.

The verb's closed eligibility list, `internal/loomcli/doc.go:98`, implemented in `internal/loomcli/arbitereligibility.go:38`:

> A stop is eligible only if the verb's closed list admits it: an error stop of kind `producer` or `bootstrap`, an `interrupted` stop under the `reinvoke` policy or with a transient class, a halted `blocked` stop other than a Publish halt that names a target, and a halted `awaiting` stop whose Bouncer row has a pending escalation.

The driver stencil, `contracts/stencils/shed/shed-template-driver.md:106-113`:

> Run it at exactly these stops, and never run it at any other:
> - a `blocked` stop;
> - an `awaiting` stop whose envelope carries a non-empty `parent_notice`;
> - an `error` or `interrupted` stop, after the fork's action is an escalation;
> - a transient stop that failed twice.
>
> Never run it at an error stop whose fork named a repair verb or a re-launch.

Lines 60, 63 and 80 of the same stencil route every `blocked` stop, every `awaiting` stop with a `parent_notice`, and every twice-failed transient through `## Arbiter` first.
The driver's own judgement is thus not a filter: the stencil makes the arbiter mandatory at each listed stop, and only the fork's choice of a repair verb or a re-launch keeps an error stop away from it.
In practice the gate is the verb's list, and that list admits every `blocked` stop and every budget escalation.

## Attempts on disk

| Run | Attempt | Stop key | Verdict | Needed? |
|---|---|---|---|---|
| convergence-plan | 1 | Discussion-Bouncer, history 10, awaiting | resume, `circling continue` | No: a BLOCKING design-wrong finding (D10's join) was fixed and not re-reviewed; `judge-cap-call`'s soft-cap rule gives "one more round" from the finding text alone |
| convergence-plan | 2 | Discussion-Bouncer, history 14, awaiting | resume, `circling accept` | No: every finding fixed as prescribed, none design-wrong; the soft-cap rule gives "accept unconverged" |
| operator-followups | 1 | Discussion-Bouncer, history 10, awaiting | resume, `circling accept` | No: no BLOCKING finding, review APPROVED, one MEDIUM refinement fixed; `discussion_auto_accept` or the soft-cap rule settles it in shed |
| operator-followups | 2 | Merge-In, history 24, blocked | resume, goto `Webster-Burler-Review`; never applied | No: the stop envelope's own reason already named the way forward (`lyx loom goto --to Webster-Burler-Review`, then `lyx loom resume`); the run was moved there 17 seconds after the spawn, and the relaunch was refused as moved (`arbiter-relaunch-moved`) |
| cleanup-1012 | 1 | Discussion-Bouncer, history 10, awaiting | resume, `circling accept` | No: cause `budget`, all four findings settled, nothing disputed; auto-accept or the soft-cap rule settles it |

Records live under `<slug>-weft/_lyx/shed/<slug>/arbiter/attempt-<N>/`.
Four of five were budget escalations at a Bouncer, which the arbiter settled from the round files with its fixed rule ("accept unless a finding makes the design wrong, and hand on a blocker that remains after round 3"), the same rule `judge-cap-call` hands to the judge.
The fifth was a `blocked` stop whose way forward Go had already computed: a deterministic goto, not a judgement.

## Proposal: a tight trigger list

The arbiter is worth an opus spawn only where a judgement is needed that neither Go, the Bouncer's judge nor the driver's fork can make, and that the orch would otherwise spend a turn on.

### Arbiter spawns

1. A Bouncer at the **hard cap** of `judge-cap-call` with a design-wrong blocker still standing, or a blocker that returned after the soft-cap round (circling).
   The arbiter decides continue, accept or hand on, and may record a design call.
2. A `blocked` stop with **no computed way forward**: a halt whose envelope names no goto target and no retry, such as a gate that concluded a human is needed without naming why the task cannot fix it.
3. An `error` stop of kind `producer` or `bootstrap` whose fork **escalated after the repair cap was spent**, where the fork's report leaves a choice between re-launching, a goto and a hand-back.

### Every other stop

| Stop | Today | Instead |
|---|---|---|
| Bouncer budget escalation at the soft cap | arbiter | The judge settles it under `judge-cap-call`: accept unconverged on mechanical findings, one more round on a fixed-but-unreviewed design-wrong finding, circling on a returning blocker |
| Bouncer budget escalation with no open BLOCKING finding | arbiter | Shed accepts it, as the per-perch `*_auto_accept` flag already does where set |
| `blocked` stop whose envelope names a goto way forward (Merge-In verify failure) | arbiter | Shed performs the goto itself, as a bounded edge in the recipe, or the driver runs the named `goto` and `resume` without a spawn; the arbiter's verdict only restates it |
| `blocked` Publish halt that names a target | refused already | Batten's retry, then the orch |
| `awaiting` stop with a `parent_notice` and no Bouncer escalation | refused by the verb, though the stencil sends it | Straight to the orch; the stencil line 109 is narrowed to match the verb |
| transient stop that failed twice | arbiter | The driver's fork reads the trace and re-launches under the repair cap; past the cap it is case 3 above |
| `interrupted` under `reinvoke` | arbiter after a fork escalation | The driver re-launches under the repair cap; past the cap it is case 3 |
| `busy`, `ownership`, no-kind refusal, `paused`, `awaiting` at PR-Gate | hand back | Unchanged: straight to the orch (PR-Gate also to the alignment agent) |

### Fit with `judge-cap-call`

`judge-cap-call` moves the soft-cap call to the judge and leaves "the arbiter, or the parent" open at the hard cap.
This note answers that open point for the arbiter: the hard cap and a returning blocker are the arbiter's only review triggers, and the verb's `EscalationPending` check narrows to an escalation whose cause is the hard cap or circling, never `budget` at the soft cap.

## Changes this implies

- `internal/loomcli/arbitereligibility.go`: an `awaiting` stop is eligible only at the hard cap or on circling; a `blocked` stop whose envelope names a goto way forward is refused like the Publish halt that names a target.
- `contracts/stencils/shed/shed-template-driver.md` lines 60, 63, 80 and 106-113: the stencil's list matches the verb's narrowed list, and a stop the verb refuses goes straight to its own handling without a spawn attempt.
- `contracts/stencils/loom/loom-template-arbiter.md` line 59: the fixed review rule moves to the judge; the arbiter keeps only the hard-cap call.
- `internal/loomcli/doc.go` "# Arbiter" and `contracts/specs/refusal-spec.md` rows 250-258 follow.

## Open

- Whether a Merge-In verify failure becomes a recipe edge (`on_stuck`-like, budgeted) or stays a driver-run goto.
- Whether case 3 is worth an opus spawn at all, or goes straight to the orch; no attempt on disk exercised it.

## Later: batten stops

The operator's view (2026-10-12), to think about later: a batten stop such as a Worktree-Teardown failure could be settled by an arbiter-like LLM instead of the orch.
Example: `operator-followups` teardown failed because reed recreated two empty lock files in the pair during removal; the fix was to delete them and re-run `lyx batten run <slug>` (finding recorded in `run-window-bugs`).

## Operator's thinking: a general helper (2026-10-12, open)

The arbiter is not loom-specific at heart: a cold agent given a stop, an instruction and a set of allowed actions, answering with a verdict.
Today only its verb (`lyx loom arbiter`), its stencil wording and its eligibility list tie it to loom.
The operator sees it becoming a general "we do not know what to do here, something is wrong, call a general helper" for any shed run, batten included, and is still thinking about it.
Open: today it only judges and the driver acts; a helper that fixes things (deleting stray lock files, say) must act itself, bounded by the allowed actions in its instruction.

## Arbiter and parent double notice (seen 2026-10-13/14)

When a review segment circles or hits its hard cap, the escalation goes to the arbiter first.
The parent (the hub orch) also gets a notice while the arbiter is still deciding.
Seen in the runs `logging-cleanup` and `tender`.

The orch must check the round's `round-N-circling-decision.md` and `lyx loom status` before acting, and does nothing if a decision is already written.

The parent should be notified only on the arbiter's `hand-on` or when its spawns are spent.
