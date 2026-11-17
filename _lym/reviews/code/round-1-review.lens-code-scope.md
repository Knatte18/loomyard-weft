```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens — map each diff change to the decision it builds, and each decision to its changes.

**Per change → decision**

| Change | Traces to |
|---|---|
| `heddleengine` `Passage`, `Passages` and `FileEntries` returning slugs and engine keys, `Target.Engine` on the file targets, `keyedEngine`, doc.go; `shuttleengine.Passage`/`Standing`/`StandingPassages`; the `cmd/lym/main.go` reader adapter | D1 (shared targets keyed and compared, single-engine targets keyed to one engine), card 1 |
| `shuttleengine/manifest.go` (record type, `WriteManifest`, `ReadManifest`, `MissingItems`, `copyManifest`, `WebsterForkDenyKey`), the `run.go` copy call, the `rundir.go` skips of `manifests`, doc.go | D1 (record, copy under the root's `manifests` directory, sweep and strand-lookup skips), tolerant read through `internal/state` |
| `claudeengine/manifest.go` and the `Prepare` hook-up, `usersetup.go` `ListedSkills`, `pluginhook.go` `Plugin`/`Hook`, `shuttleengine/usersetup.go` fields | D1 (read back from artifacts, native items, omitted plugin contexts counted missing in the verdict) |
| `codexengine/manifest.go` and the `Prepare` hook-up, `developerinstructions.go` slug-named passages, `usersetup.go` silent on an unrun hook | D1 and D10 (an unrun hook is listed, logs no fix-it warning) |
| `shuttleengines.NewWithSeams`, `LaunchSeams`, `registeredConstructors`, the `keepwarm.go` refactor | D1 and D10 (injectable hook runner and sandbox probe, taken from codex-standing-instructions) |
| `seatengine.Specs`, `burlerengine.SeatTables`, `landingshed.FixReviewSeatTable` | D1 (every seat of the tables gets a per-engine spec); forced by the seat catalog |
| `shuttleparity` (`prepare.go`, `seats.go`, `report.go`, the tests, `doc.go`) | D1 (parity test, producer table, scan, the `Compare` fork-deny gate, every seat compared) and D10 (shared `UnrunHooks`/`ConfiguredSandbox`, temp-dir `Report`) |
| `shuttlecli/manifest.go`, the `cli.go` wiring, the `cli_test.go` row | D10 |
| `PATTERN.md`, `pattern/PATTERN-engine-parity.md`, the `PATTERN-shuttle-provider-seam.md` addition | PATTERN changes entry; the seam addition is forced by `NewWithSeams` |
| `docs/overview.md` | Forced doc update for the new package and verb |
| `cmd/lym/testonlycode_linux_test.go` (`seatengine.Prompts` entry removed) | Forced: `Specs` now calls `Prompts` from production code |
| `cmd/lym/treesitterlink_integration_test.go` (new allowed link) | Forced: the producer test imports `loomcli` |

**Per decision → builds**
- D1: all the changes above except the `shuttlecli` verb, the `shuttleengines.NewWithSeams` hook-runner/probe injection that D10 also names, and the doc and PATTERN edits.
- D10: `shuttlecli/manifest.go`, `shuttleparity/report.go` and `prepare.go`, `NewWithSeams`.
- D11: no change in the diff. This is expected, because the arbiter and parent entries record that the orch wrote it on the board and card 10 is dropped.
- Added entry on assist-trial risk: no code, nothing to build.
- Added entry on codexengine as the bench-rule home: the board, no diff.
- Added entry that card 10 drops: consistent, nothing in the diff writes the board.
- Added entry on every seat compared: built. `TestSeats_CodexManifestHoldsClaudes` and `TestProducers_CodexManifestHoldsClaudes` have no `Unmet` skip.
- Added entry on the fork deny gate: built in `Compare`. `TestCompare/Compare` rows `ForkDenyIsNotAskedOfAnEngineWithoutForks` and `ForkDenyIsAskedOfAnEngineWithForks` pin the tripwire, and no Codex fork deny was added.
- Added entry on the plan-review round-3 close: the `Seat.Specs` model choice is the engine alone (`seats.go:105`), `testmain_test.go` exists, and the interim allowlist entries net to zero.
- Added entry on the round-3 Discussion findings: attach-probe literals are skipped by `wayForwardOf` (no `Prompt` key). The `judgeCall` and `runSeedSpawn` renderers (`RenderJudgePrompt`, `RenderSeedPrompt`) already exist in `shedadapters/bouncer.go`, which this diff does not touch, so they are not a change here.

**Findings:** none. Every hunk traces to a decision or is forced by a traced change, and every decision with a code effect has a building change.

**Tool calls:** none; my context already held the diff, the decision record and the plan overview.
