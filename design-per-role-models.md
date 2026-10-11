# Model keys per judge row, seed and PR-Rework; judge verdict and usage fixes

`per-role-models` · Task · bug, enhancement, config, loom, burler · running · Loom-Preflight

## Seen

The haiku-roles analysis (`docs/research/haiku-judge.md`, 5459376bb) found that one `judge` key in `loom.yaml` sets the model for all three Bouncer rows and their seed passes; a per-row model exists only as `model`/`effort` in the recipe, which is built into the binary.

## Idea (operator, 2026-10-10)

Most agents a run spawns should take a model of their own in config.

- Give each Bouncer row's judge its own model key in config, so Discussion-Bouncer can stay on Sonnet while Plan-Bouncer and Webster-Bouncer try Haiku 5.5.
- Give the seed pass (role `bouncer-seed`, writes only `round-1-focus.md` before round 1, never gates) a model key separate from the judge's; the analysis rates it Haiku-safe on every row, so it moves to Haiku 5.5 on all three rows once the key exists.
- Find every other role in the loom and darn recipes, batten and webster whose model is shared with another role or settable only in the recipe, and give it its own key in the same task.
- A role survey (2026-10-10) found these without a key of their own:
  - PR-Rework reuses `loom.yaml` `plan` (`loomengine/rework.go`).
  - Webster's implementer and verify-fix forks inherit `webster.yaml` `master`.
  - The Bouncer escalation fork and the fan's lens forks inherit the parent session's model.
  - `landing.yaml` `describe` and `darn.yaml` `writer` are set hub-wide only; the operator ruled (2026-10-11) that a per-worktree value is not needed for these simple roles, so they are out of this task.

## Webster stays fork-based (operator, 2026-10-11)

Webster's implementer and verify-fix forks stay forks and run on `master`'s model; they are out of this task.
A cold-start implementer with a model of its own is the separate feature `weaver`.
The same holds for the Bouncer escalation fork and the fan's lens forks: a fork always runs on its parent session's model, so they are out of this task; the lens forks' model is set through the fanned reviewer's own key (`fan_review`).

## Judge: CONVERGED with an open BLOCKING finding (was `judge-converged-open-blocking`)

The haiku-roles analysis (`docs/research/haiku-judge.md`, 5459376bb) found that Go accepts a CONVERGED verdict even when a BLOCKING finding is open, and that the findings carried into the decision record come only from the judge's own ledger.

Idea: refuse CONVERGED in Go while any BLOCKING finding is open, and check that every finding key from the previous ledger is still present, so a weaker judge model cannot drop one silently.

## Judge and seed usage unrecorded (was `judge-usage-unrecorded`)

The haiku-roles analysis had to measure judge and seed cost from session transcripts, since the per-round usage files have no judge entry.

Idea: record judge and seed passes in the usage files like other roles, so the token report per role shows them without reading transcripts.
