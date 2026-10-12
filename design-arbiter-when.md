# Arbiter spawns only on stops that need it

`arbiter-when` · Note · enhancement, loom, shed

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
