## Goal

```markdown
## Seen (orch, 2026-11-01)

`Parts-Write` and `Parts-Plan-Write` landed behind the tapestry flag `plan_split`, off by default: `Parts-Write` splits a task into parts with contracts and shared spots, and `Parts-Plan-Write` writes one plan part per fresh session under a joining chair.
Nothing has benched them against `Plan-Write`; note `plan-writer-context` holds the background (the Plan writer is the costliest session, about 83 turns at a mean context of 183k tokens).

## The study

A darn with profile docs writes the plan of the same input twice, once with `Plan-Write` and once with `Parts-Write` then `Parts-Plan-Write`, and compares the two.
- Inputs: two landed tapestry runs' approved decision records and discussions, `warp-visibility` and `run-verbs-shared`, each replayed at the start of Plan on a scratch branch at its own base commit.
- Per plan: cost and wall time of the writing sessions and their forks, turns and context per session, whether the joined plan passes the ordinary `plan` gate first time, its card count, and the cards that read the same files.
- Quality: one round of the hub's Plan-Review per plan, its finding count and severity.
- Cap: 2 inputs times 2 arms, one repeat, the models and efforts the hub's config gives each producer today: 4 plan writings and 4 Plan-Review rounds, no other agent or model run beyond the darn writer.
- The report goes to `docs/research/` with its scripts; it recommends, it decides nothing.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-01): Scratch worktrees for the plan replays

Decision: The writer may create up to 4 scratch worktrees with lym fabric add (never --adopt-local), one per input and arm, drive the Plan producers and one Plan-Review round in each within the entry's cap, and remove every one with lym fabric remove when its numbers are taken. tapestry.plan_split is set only in each scratch worktree's own config layer, never in the hub-wide layer, since other runs read that; if the setting cannot be set per worktree, ask the orch before changing anything hub-wide. The replays get no board entry and no batten run.

Rationale: The entry's study replays two landed runs at the start of Plan on scratch branches; worktrees are the only way to run them, and the cap bounds them.

### Added after Discussion (parent, 2026-11-01): The orch creates the scratch worktrees

Decision: The orch created four local scratch worktrees from the prime with lym fabric add --local: psb-wv-plan, psb-wv-split, psb-rv-plan and psb-rv-split. The writer resets each to its input's base, checks out the records, sets plan_split in each worktree's own tapestry config and runs the Plan producers there. When the numbers are taken, the writer reports and the orch removes the four with lym fabric remove; the writer does not remove them.

Rationale: lym fabric add is refused from a task session, and its refusal names the orch as the one to run it.

### Added after Discussion (parent, 2026-11-01): The failed rv split arm stops; no extra writing

Decision: psb-rv-split failed at Parts-Plan-Write when reed lost the chair's strand. The orch touched no reed state. The arm stops there: the report gives its partial cost, its interventions and the failure, marked as an infrastructure failure (reed strand loss) rather than a Parts-Plan-Write result, and compares the split arm on the wv input alone. No rerun beyond the entry's cap of 4 writings unless the operator approves one.

Rationale: The entry caps the study at 4 writings; extra spend is the operator's call.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
