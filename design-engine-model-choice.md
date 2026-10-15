# Every role spec carries its resolved engine

`engine-model-choice` · Task · enhancement, shuttle, loom, webster, burler, shed · running · Webster

## Goal

A task of the Codex-readiness series (see `engine-registry` for the operator decision): every role's spawn spec carries the engine its model alias resolves to, not only the model.

## Seen (2026-10-15)

- No consumer reads `modelspec.Resolved.Engine`; about 17 places copy `Model`, `Params["effort"]` and `Params["version"]` by hand: `darnengine/spec.go`, `landingshed/describe.go`, `frictionengine/spec.go`, `mergeresolve/spec.go`, `loomengine/{review,driver,rework,discussiontable}.go`, `websterengine/{runlevel,recoverbatch}.go`, `sheddriver/config.go`, `shedrecipe/entries_multillm.go`, `shuttlecli/open.go`.
- Separate model carriers have no engine field: `burlerengine.ModelChoice` (also `shedadapters.BouncerModels`), treadle's model strings, `seatengine`'s `seat.Model`, alignment's `s.Model`.
- `websterengine/merriambase.go` counts `CLAUDE.md` and `CLAUDE.local.md` by name.

## Scope

- Replace `burlerengine.ModelChoice` and the bare model strings with `engine-registry`'s `shuttleengine.ModelChoice`.
- Convert every hand mapping from `Resolved` to `Spec` through it.
- Per-round model lists keep the engine, and `round-N-usage.yaml` records it.
- Claude-only roles declare their requirements through `Spec.Requires`: Webster master and implementers, twill, seats with advisors, the Burler cluster fan, the Claude-strand loom driver.
- Merriam's base reads the engine's instruction files instead of naming `CLAUDE.md`.

## Out

The orch model (Claude-only for now), key renames (note `loom-model-key-names`, which should wait on this task, as should `shed-arbiter-all`), adding `codex` to the engine names.
