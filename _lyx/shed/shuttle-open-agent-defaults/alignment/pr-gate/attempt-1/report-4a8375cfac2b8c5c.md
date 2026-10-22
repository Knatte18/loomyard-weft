---
verdict: APPROVED
---

Alignment at `pr-gate` for `shuttle-open-agent-defaults`, pull request 813 at `19ccaf0eb88491607a7b60ca2873f8d4412f0a1d` against `main`.

## Measure

The decision record holds no `Added after Discussion` entries, so the board entry's body is the measure unamended.
Its points and where the diff meets them:

- Every agent session lyx launches into reed starts in its engine's bypass mode with no flag, `lyx shuttle open` included: `validatePermissionMode` in `internal/shuttleengine/claudeengine/command.go` resolves an empty mode to bypass for autonomous runs, interactive runs and opened sessions, inside the engine.
- Per engine, not claude-only: the default stays in `claudeengine`, and the `Spec` and `OpenSpec` docs say each engine maps it onto its own bypass flag; no Codex engine is registered, so its `--yolo` lands with it.
- `lyx reed add --help` sends an agent launch to `lyx shuttle open` and no longer shows the broken `%SID%` example: `internal/reedcli/add.go`.
- `lyx shuttle open --help` documents the model-spec grammar, aliases and effort: `internal/shuttlecli/open.go`.
- The output says which engine, model and effort the session got: the success envelope adds `engine`, `model` and `effort`.
- A way to list the registry's aliases with engine, model and default effort from the command itself: `--list-models`.

The diff changes nothing the entry does not call for; the doc edits in `internal/shuttleengine/doc.go`, `claudeengine/doc.go`, `engine.go`, `spec.go` and the shuttle row of `docs/overview.md` follow the behavior change.

## Open risks

- The record's own risk, interactive runs no longer prompting by default, is the behavior the entry asks for; `prompt` stays the opt-out.
- The Darn-Review carry-over `claudeengine-doc-open-bypass-stale` names no parent correction; commit `0a7cf9ab4` rewrites the open-launch sentence of `claudeengine/doc.go` to say bypass is carried unless the mode is `prompt`, which the diff shows.

## Overlaps

`lyx loom peers` lists no parent commits.

- `decision-add-notifies` (row 16, own row 21): both change `docs/overview.md`, at different module rows (its `lyx loom decision add` paragraph, this run's shuttle row). Text-only. This run is further along and lands first; `decision-add-notifies` takes the change at Merge-In.
- `gate-tier-verbs` (row 0): both change `docs/overview.md`, at different rows. Text-only. Unknown order.
- `darn-verify-fixes`, `machine-handover`, `quarry-csharp`: no changed paths yet, and their decision records touch neither permission modes, `shuttle open` nor `reed add`. `darn-verify-fixes` D4 changes `internal/shuttleengine`'s gate outcome only, apart from this run's permission and open paths. No overlap.

No design conflicts.

## Superseded

No board entry is made obsolete by this run; `codex-engine` stays open, since its bypass equivalent lands with that engine.
