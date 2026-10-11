# Model and effort set per role and per review row

`per-role-models` · Note · enhancement, config, loom, burler

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
  - `landing.yaml` `describe` and `darn.yaml` `writer` are set hub-wide only, so a trial cannot be scoped to one task pair.

## Scope (operator, 2026-10-11)

- In scope: a model key per Bouncer row's judge and per row's seed pass, with today's `judge` key as the fallback, and a key of its own for PR-Rework.
- Out of scope: forks keep the model of the session that forks them, by design.
  Webster's implementer and verify-fix forks, the escalation fork and the lens forks stay forks, because a fork starts at once with its parent's context.
  Millhouse ran a fresh agent per implementer under a separate orchestrator ("Builder"), and each cold start spent long reading background and loading tools before it began.
  A different implementer model therefore means a different `master` model.
- Out of scope: `describe` and `writer` scoped per task pair belong to `config-overlay`.
- The Haiku judge trial is dropped (see `haiku-roles`); the keys are wanted on their own merits.
