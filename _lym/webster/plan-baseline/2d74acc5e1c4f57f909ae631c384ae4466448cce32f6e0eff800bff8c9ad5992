---
format: 5
approved: true
language: go
---

# Plan: sampler, an interactive loom recipe

Add `sampler`, a loom recipe whose build segment is one interactive agent session in the run's worktree that works with the operator, commits as it goes and ends the build through `lym loom sampler-done`; the run then goes on through the Code-Review perch (off by default, `sampler.code_review`), Describe, PR-Review, Finalize and Friction-Reflect.
The cards build the hub-wide `sampler.yaml`, the row's record, stencil and producer, its registry entry and recipe, the recipe name's reach into the vocabularies, the rows-keyed loom wiring, the verb, the widened `decision add` caller rule and one list of rows that wait on the operator.

## Card Index

1 — sampler-config — a hub-wide sampler.yaml module with agent and code_review, registered in configreg
2 — sampler-record — the Sampler-Session record, its done flag and the entry stamp, with shedrun paths
3 — sampler-stencil — the sampler-template-agent stencil and the session's spec builder
4 — sampler-session-row — loomshed's Sampler-Session producer: its session, wait and done judgement
5 — sampler-registry-entry — register SamplerSession as a loom build engine with Env.Sampler, and embed the sampler recipe file
6 — sampler-recipe — the sampler recipe's name in the recipe vocabulary, lym shed and loomrecipe's accessors
7 — sampler-vocabulary — sampler in batten seeding, the board, run profiles and run descriptions
8 — rows-keyed-wiring — tree-scope, record and perch-off wiring keyed on the recipe's rows
9 — sampler-wiring — wire the Sampler-Session row's seams for a sampler run
10 — sampler-done-verb — lym loom sampler-done
11 — decision-sampler-caller — decision add admits the sampler session with --by operator
12 — operator-wait-rows — one list of operator-wait rows read by dobby and the step watchdog

## Shared Decisions

- The Sampler-Session producer lives in `internal/loomshed` beside Darn-Write and copies the shape of PR-Review's session code rather than sharing it, since the two rows keep different records; `internal/samplerengine` owns the config, the record, the entry stamp and the session spec, as `darnengine` owns darn's config and table.
- The record and done flag share one never-tracked file, `sampler.json` under the run's scratch, as PR-Review's record holds its session and continue flag.
- The operator-wait list lives in `internal/loomshed`, which both `battencli` and `loomcli` already import and which lies outside `boardengine`.
