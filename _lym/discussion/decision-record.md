## Goal

```markdown
## Why

`plan-split-tune` landed the first round of tuning of the plan-split writers (Parts-Write, the part writers and the joining chair) without replaying it.
A replay must run the tuned lym and stencils, and stencils live only in the hub's shared Bolt copy, which takes only what lands on `main`, so the replays run here, after that landing, on the installed lym.

## Goal

Replay the landed tuning, then tune and replay again, until a split plan joins and passes its gates with no hand edit.
Start from the `plan-split-tune` report in `docs/research/` and the `plan-split-bench` report it builds on.

## Loop and cap

- Inputs: the two replays `plan-split-bench` used, `warp-visibility` (code at `d407ef69`, `_lym/discussion` and `_lym/reviews/discussion` from weft `576c4b3a31`) and `run-verbs-shared` (code at `52b69848`, the same directories from weft `b8a86e5c59`), each worktree with `plan_split: true` in its own `_lym/config/tapestry.yaml`.
- The first round replays the landed tuning with no further change.
- A later round tunes the instructions or gates, then replays Parts-Write and Parts-Plan-Write on both inputs.
- A tuning made in a later round reaches the replays only after it lands, so a later round that needs it ends this task with its change landed, and the report names what the next replay should check.
- Record per input and round whether the parts joined, which gates refused, and every hand edit needed.
- At most 2 rounds, so at most 4 split writings, and one Plan-Review round per input on the last round's plans only, 2 in all.
- The loop ends early at the first round where both inputs join and pass their gates with no hand edit.
- The orch creates and removes the scratch worktrees on the writer's request, since `lym fabric add` is refused from a task session.
- The results go into the same `docs/research/` report, round by round, with the cost.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
