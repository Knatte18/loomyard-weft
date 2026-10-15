---
verdict: APPROVED
---

## Alignment at pr-gate — `engine-config-seams`, PR 656 at `5c7313a66`

The diff against `main` (merge base `530c339e9`) carries every goal point of board entry 548 and keeps to the decision record's D1–D10.

- `claude.yaml` is a `claudeengine` module with its own template, spec and lazy `LoadConfig`, and it reaches `configreg` through `shuttleengines.ConfigModules()`, so `configreg` imports no concrete engine (D1, D2).
- `shuttle.yaml`'s template drops every Claude key and keeps the python switch as the neutral `deny_python` (D3).
- `configreg.KeyMoves()` declares the five moves; `configsync/movekeys.go` applies them per layer, and `PrepareRunSettings` applies them to run settings (D4).
- `internal/shellguard` holds the shell rows, `claudeengine/settings.go` renders them against a byte-identical golden (`testdata/settings-golden.json`), and `cmd/lyx/norecoverydeny_test.go` builds its forms from `shellguard.Rows` plus the operator's list (D5–D7).
- `reedengine.CleanClaudeEnv` is gone; `reedengine.New` takes the engines' declared scrub from `shuttleengines.EnvScrub()` (D8).
- Strands carry `Provider`, set from the resolved engine name in `Runner.start` and `Runner.Open`; `WatchNames` takes a per-tag namer map, and `shuttleengines.SessionNamers()` maps the empty tag to the `claude` namer (D9).
- PATTERN gains `PATTERN-shell-guard-table`, and the leaf, strictness, provider-seam, no-denied-recovery and test-isolation backgrounds are updated, with package docs and `docs/overview.md` (D10).

No change falls outside the goal.
Commits `10c4a59bf` and `5c7313a66` shorten six existing `PATTERN.md` overview lines so the new entry fits under the headroom line.
Each trimmed clause is still stated in its background file (`PATTERN-hub-suffix`, `-config-layers`, `-batten-bookend`, `-parent-directive`, `-edit-directive`, `-spawn-color`), so no invariant is lost.
Comment-only edits in `burlerengine/doc.go`, `loomengine/plan.go`, `fabriccli/clone.go` and `shuttleengine/spec.go` follow the retired key names and the removed `CleanClaudeEnv`.

The open risks in the decision record are accepted bounds of D4 and D5: a Codex payload shape for `codex-engine`, stale `claude_*` keys ignored until the first reconcile walk, and board run settings rewritten only at seeding.
The Plan-Review carry-over lists two MEDIUM plan findings (per-row guardrail comments in card 2, a steer line-break check mismatch) that the fixer addressed with no fresh review afterwards.
Neither changes the goal or a decision; the code keeps the rows' patterns, steers and notices, which the golden test pins byte for byte.

## Overlaps

`lyx loom peers` lists no parent commits and one peer.

- `engine-model-choice` (row 12, status Webster) against this run's `own_row` 22.
  Shared files: `internal/shuttleengine/spec.go`, `run.go` and `enginedispatch_test.go`, `claudeengine/command_test.go` and `prepare_test.go`, `shuttlecli/smoke_interrupt_test.go`, `treadleengine/smoke_judge_test.go`, `burlerengine/doc.go`, and `cmd/lyx/testdata/test-budget.yaml`.
  The overlap is textual: this run edits `Spec`'s comments on the deny keys and adds `Provider: engineName` to the strand spec in `start` and `Open`, while the peer replaces `Spec`'s `Model`, `Effort`, `Version` and `Engine` fields with an embedded `ModelChoice`.
  The peer's decisions read `shuttle.yaml`'s `engine:` key, which this run leaves unchanged, and neither record contradicts or duplicates the other; the peer's record already expects whichever lands second to adapt.
  No PATTERN entry is changed by both: the peer adds `PATTERN-model-choice-mapping`, this run adds `PATTERN-shell-guard-table`, and the `PATTERN.md` index lines are shared bookkeeping.
  Landing order: this run is further along (row 22 against 12), so it lands first and `engine-model-choice` takes its changes at Merge-In.

`cmd/lyx/testdata/test-budget.yaml`, `docs/overview.md`'s module-table line and the new `PATTERN.md` index line are shared bookkeeping files, so they raise no escalation.

## Superseded

No board entry is made obsolete: `codex-engine`, `engine-usage-readers`, `engine-model-choice`, `spawn-audit-log` and `shuttle-open-permission` each keep work this run leaves out of scope.
