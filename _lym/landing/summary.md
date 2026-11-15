# Report: Codex astra[high] as the darn writer, replayed on a landed task

Adds `docs/research/codex/darn-writer/codex-darn-writer-replay-neve-2026-11-15/`, a report and its data on one Codex session that did the darn writer's work for `ide-beam-readme-preview` (landed as `824eb2b6`), for level 2 of `codex-support-levels`.
No production code and nothing under Bolt changed.

The session finished the work and its tree passes the gate on `./internal/ideconfig`.
Its `template.yaml` and `config_test.go` match the landed change byte for byte.
It also added a documentation page the entry did not ask for, and its change description names no design decisions.
It took about 4 minutes against 5 minutes 41 seconds for the landed writer, whose figure includes a 2 minute 17 second verify gate; its dollar cost is unknown.
The report leaves the verdict on level 2 to the operator.

## Design decisions

- The prompt is the Darn-Write stencil filled for the landed run's record and a scratch worktree at the parent of `824eb2b6`, with the no-parent directive and a two-line preface naming the worktree.
  The report lists each deviation from a production render.
- The scratch worktree is no `lym` worktree, so the session ran no board, drift or loom check; the report names this as a limit.

## To check by hand

- Read the report's "Diff against the landed one" against `data/codex.diff` and `data/landed.diff`.
