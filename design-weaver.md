# Weaver: a cold-start implementer beside Webster, with model escalation

`weaver` · Note · feature, webster

## Idea (operator, 2026-10-11)

Webster stays fork-based: its implementer and verify-fix forks start at once with Master's context and run on Master's model.
Weaver is a second implementer beside it, built on Millhouse's "Builder" (skill `mill-go`), that runs a cold-start agent per batch, so each agent can take its own model, and escalates the model when an agent fails.

## Seen (research, 2026-10-11)

- Webster spawns one cold agent today: the recovery strand.
  When a fork reports `stuck` or writes no report twice, Master runs `lyx webster recover-batch` (`contracts/stencils/webster/webster-template-master.md`), which starts a fresh shuttle/reed strand with a full cold-start prompt (`internal/websterengine/recoverbatch.go`).
  Its model is `webster.yaml` `recovery` (default `opus[high]`), against `master` at `sonnet[medium]`; that is Webster's only model escalation, and a batch gets at most two recoveries.
- Builder (`plugins/mill/skills/mill-go-base/SKILL.md` in the Millhouse worktree) is a lean orchestrator that never reads cards or diffs.
  Per batch it starts a fresh, non-fork agent with a model from config (`roles.implementer.model`), and the agent reads only its batch file and the overview's Shared Decisions.
  It has no automatic model escalation: a stuck batch gets one fresh retry on the same tier, then a plan edit with a `## Prior failure` note, then a question to the parent.
- Millhouse's cold starts were slow: each agent read much background and loaded its tools before it began work.
  Likely further costs visible in the skill: a review loop per batch, finalize replaying every batch's verify in turn, and a state commit on every status change.

## Shape

Weaver is, in effect, Webster's recovery strand used as the normal path.

- From Webster: the Go bracket verbs (begin-batch, record-batch, recover-batch), the report contract and classification, suspect paths, partition and sequencing, the plan-level verify gate and failure ladder, the shuttle/reed spawn path, and `RenderRecoveryPrompt` with `uncommitted_paths` and the failure digest.
- From Builder: a cold, scoped agent per batch that reads only its cards and Shared Decisions, a model and effort per role per spawn, and a one-shot self-resolve before halting.
- New in neither: an escalation ladder per failed attempt, for example `sonnet` → `opus` → `opus[high]`, set in config.
- Open: how to keep cold-start cost down (a kick-start pack per batch, see `kick-start-pack`), and how a run chooses Weaver over Webster (recipe row or config).

## Idea: Weaver as a Shed run (operator, 2026-10-11)

Loom's driver now does little: most of a loom run is driven mechanically, and the driver acts only when something goes wrong.
Weaver gets the same split: a Weaver run is a Shed-like run in which each batch is a producer, and a Master Weaver drives it the way a driver drives loom, stepping in only when a batch fails.

Open against the code:
- A Shed recipe's rows are fixed, while a plan's batch count is not: either the recipe is generated from the plan, or one batch producer loops over the batch index, as the Burler round loops over rounds.
- The escalation ladder could be Shed rows: a batch's `OnStuck` points at the same batch on the next model tier, and the last tier's `OnStuck` escalates to Master Weaver.
- Whether Master Weaver is an LLM at all on the happy path, or only the escalation target, as the hub orchestrator is for loom.

## Idea: batches formed as the run goes (operator, 2026-10-11)

Batches are not fixed up front.
The run starts from the plan's linear card list, and a batchifier forms the first batches: the cards that can be spawned right now.
Batches, or single cards, whose glyphs do not overlap are spawned in parallel, each in its own worktree.
Each time a batch finishes, the batchifier runs again over the remaining cards and forms the next batch or batches.
The plan is updated as the run goes, since an implementer may have to deviate from it, and the next batchifier pass reads the updated plan.

Open against the code:
- This answers the fixed-rows question above: the run is a frontier of ready batches, not a fixed row list.
  The recipe can still be fixed if the batches are data, not rows: a Batchify row forms the frontier, a Batch-Run row runs it and loops back to Batchify while cards remain, then the run goes on to the plan-level verify, the way the Burler round loops over rounds.
  New for Shed: one row that runs several spawns in parallel and finishes when all of them have.
- Parallel worktrees need a merge back into the task branch after each batch; glyph disjointness rules out textual conflicts but not semantic ones, so the verify gate runs on the merged result.
- Who writes an implementer's deviation into the plan (the implementer through its report, Go from the report, or Master Weaver), and how the batchifier tells a deviated card's new glyphs from its planned ones.
- What the existing batcher module (`batcher` config) and Webster's partition and sequencing already give a batchifier.

## Was `webster-parallel-execution`

From `manifest/roadmap.md` Someday.
Give each DAG-independent group its own `fabric`-spawned worktree, so concurrent cards stop sharing one git index.
A speed optimization over an already-correct sequential system.
The loom plan spec and `internal/websterengine`'s package documentation describe the sequential model it would extend.

## Design

> **Status: parked, not rejected — one shape was rejected, a second is open.** What two explorations rejected is *concurrent forks inside one working tree*, which race on a single shared git index; the 2026-08-20 worktree-per-group shape below does not share that index and is not covered by that rejection. Read past this banner before concluding the idea is dead — the earlier wording said "rejected for now both times" and, until the 2026-08-29 designs audit, sent readers away before the open shape, the measured case study, and the decision gate.
> Still parked, though one of its original three reasons has since fallen: the **Adopt quarry's glyph alphabet as the plan alphabet** item shipped (`go.mod` now requires `github.com/Knatte18/quarry`), so the DAG source this doc used to wait on exists. What still parks it: it is a speed optimization over an already-correct sequential system, and webster's landed card-list rewrite (`internal/websterengine`) should get real running mileage first. Reopen it on the measured evidence the Decision gate section names, never on estimate.
> Per the documentation lifecycle, if this is ever picked up the durable parts fold into `internal/websterengine`'s package doc and this file is deleted; if abandoned, this file is simply deleted.

## Why it's parked

loom-plan-spec.md is deliberately a flat, sequential card list with **no DAG wired into scheduling**.
Running cards as *parallel* forks would require reintroducing a DAG and worktree isolation, which reopens exactly the problem sequential execution avoids: git's index/staging area is a single shared file per working tree, so two forks concurrently committing — even to fully disjoint files — race on the same lock.
Current (2026) ecosystem guidance treats worktree isolation as effectively required for concurrent subagents on the same repo.
A declared-disjoint card pair that turns out (via deviation) to actually overlap is a **live corruption risk** in a concurrent-no-worktree model, not just a bookkeeping problem to fix after the fact as it is sequentially;
quarry would also see other forks' uncommitted, potentially syntactically-broken in-flight edits while serving a concurrent fork's query, since there's no filesystem isolation between them.

**A possible middle ground, if revisited:** let forks edit concurrently (the LLM-thinking-dominated part) but serialize the actual `git add`+commit+verify step through a mutex in webster's Go orchestration ("edit in parallel, land sequentially").
Even this requires *strictly enforced* file-disjointness (not just DAG-edge-absence) to be safe.
Not built.

## A possible unblocking shape (2026-08-20)

A structural alternative to the rejected concurrent-forks-in-one-tree shape above: each DAG-independent group (a batch, possibly one card) gets its own `fabric`-spawned worktree — genuinely its own git index/HEAD, which is what the rejected shape lacked — running the existing `Preflight → Webster → Finalize` row set unchanged, with `Webster`'s `Geometry.PlanDir` (already told, not derived) pointed at the source plan and a new batch-filter selecting the one group to run.
Merge-back reuses `fabric`'s existing merge machinery, not new infrastructure, and a group's completion gates on a build+test of the *merged* result rather than of the group's own worktree in isolation.
The ready set recomputes wave to wave rather than being precomputed upfront, so a group that turns out to depend on another's output is simply not ready yet instead of being mis-scheduled.

This shape is what makes the rejection at the top of this doc specific rather than general: that rejection was about concurrent forks sharing one checkout's git index, which worktree-per-group does not do.

Grouping granularity (one card vs. several) is orthogonal to safety — the original rejection above conflated it with the working-tree-sharing hazard, but genuine worktree-per-lane isolation is what actually matters.
Not yet a plan: the DAG source exists now that the **Adopt quarry's glyph alphabet as the plan alphabet** item has shipped,
but it still needs a writeup reconciling this with the still-open questions elsewhere in this doc (typical-plan wave-width evidence, the batchifier/planner change needed to emit groups).

The per-lane mechanics have since shipped elsewhere: batten's recipe (create, seed, run, teardown as ordinary producers) and its run-id addressing carry N concurrent lane runs side by side, so a lane is a batten-style run whose child seed names a webster-lane recipe — none of that machinery is this item's to build.
What remains genuinely webster's: the wave scheduler (event-driven, recomputing the ready set on each lane's merge-back — never a continuously maintained DAG), the plan group-filter, and a non-forking webster variant.
That last one is the real structural change: today's webster runs forks inside the Master session's own context, which binds execution to one session in one worktree;
a lane needs agents spawned into the lane worktree's own reed session instead, scoped to just that lane's group.

## The case study (from an earlier, more detailed design draft, `websterv2.md`, now retired)

A card-level dependency analysis of the 42-card plan that built webster v1 overturned the naive "linear chain" assumption:

| Metric | Value |
|---|---|
| Cards | 42 |
| Batches (sequential) | 9 |
| True card-DAG depth (critical path) | 7 |
| Peak wave width | 10 |
| Cards off the critical path | 83% (35 of 42) |
| Wave widths (1→7) | 10, 9, 7, 7, 6, 2, 1 |

- **The batch DAG over-constrains.**
  Sequential batching's own dependency declarations were largely spurious at card granularity — ~26 cards (waves 1–3) could have run as three parallel waves instead of spread across four sequential batches.
- **File-conflicts barely bind** when the plan is create-then-extend — nearly every file-conflict pair is already dependency-ordered into different waves.
- **The tail is the real ceiling, not dependencies** — a hard funnel near the end of a plan (e.g. final registration → sandbox validation) crashes wave widths regardless of fork budget;
  speedup is front-loaded.
- **Honest speedup estimate: ~2–3× wall-clock**, discounted from a naively-computed 3–5× because a wave's wall-clock is its *slowest* card,
  and the heaviest implementation cards tend to sit on the critical path.
  Two caveats push it down further: semantic edges were *inferred* from card descriptions (real edit-time dependencies only add edges and shrink waves), and 42 cards is an atypically large plan — a routine 5–10-card task has little fan-out headroom and would show a speedup near 1×, dominated by warm-context, not parallelism.

## Decision gate, if ever revisited

Run the card-DAG width analysis across several *real* completed plans, weighted for **typical** task size, not this one outlier.
Wide (fat waves, short critical path, low file-conflict) → an executor might pay off.
Narrow (long critical path, most cards chained, or simply few cards) → parallelism won't materialize;
sequential is the complete correct design, not just the MVP.

## The separable, cheap win already taken

A planner that emits true card dependencies (`depends-on`) instead of an over-constrained batch line recovers most of the *width* insight with **no worktrees and no concurrent execution** — this is exactly what loom-plan-spec.md already does.
Only the *executor that actually runs the width* (this entry) remains parked.

## Relationship to quarry (Part B of the retired draft)

The retired `websterv2.md` draft also had a Part B — structured impact lookup via `go/packages`/`gopls` (find-all-references as a Go verb instead of LLM-driven grep).
That idea is superseded, not lost: it's the direct ancestor of [quarry](https://github.com/Knatte18/quarry), which generalizes it to a multi-language, daemon-based design.

## Related

- `internal/websterengine`'s package documentation — the sequential model this would extend.
- loom-plan-spec.md — already captures the cheap win (`depends-on`).
- [quarry](https://github.com/Knatte18/quarry) — Part B's successor.
