```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each change the diff makes, I looked for what else must change outside the touched lines.

**List (per change, set found, covered or not)**
- Removed names (`skip_file`, `verify_max_runs`, `DriftRecheck`, the `drift-template-baseline` and `drift-template-verify` stencils, `baseline.md`, `verify-report.md`, `runs.json`, `CompareAccepted`, the verbs `accept`, `waive`, `withdraw`, `scope` and `compare`): covered. The grep over `.go`, `.md`, `.yaml` and `.json` outside tests and `docs/research` and `_lym` finds only intended hits. Those are doc and comment text on the retired keys, `{skip_file}` refusal code (`steprun.go:297-310`, `diffmode.go:27`), the old-to-new key renames in `diffmode.go:20-22`, and `NameDriftRecheck` in `internal/loomshed` and `internal/reportengine`. The row name `Drift-Recheck` stays by D4. `docs/research` files that name removed stencils are dated records, not subjects.
- Renamed config keys (`baseline_model`, `verify_model`, `verify_interactive` to `expect_model`, `judge_model`, `judge_interactive`): covered. Migration is in `driftengine/diffmode.go`. The `internal/driftengine/template.yaml` and `config.go` edits and the `internal/configengine` and `internal/configreg` `HubOnlyWayForward` wiring are all in the diff.
- `lym drift` verb set: covered. `docs/overview.md` lists `check`, `amend`, `capture`, `defer`, `status` and `capture-child`. The darn and tapestry row prose and the `Drift-Recheck` description are updated, and `docs/dotnet-hub.md` is touched.
- `drift-verify` gate key retirement: covered. The `gateslot` template, tiers and docs, `PATTERN-gate-slot.md`, `PATTERN-test-speed.md`, the `runprofile` template and the recipes' `gate_key` all changed. No recipe still carries a drift `gate_key`.
- `PATTERN.md` and `pattern/` declared change: covered. The `PATTERN-sole-parsers` line in `PATTERN.md` equals the declared text. The background file gains the format list and the drift expectation and judgement section. `PATTERN-daemon-wakeups.md` gains the current-capture lock wait. `PATTERN-stencil-ownership.md` replaces the Drift-Verify seat wording with the judge seat under `judge_interactive`, which matches `fcd194e0`. No reference to the deleted `PATTERN-parent-directive` remains outside `docs/research` bench prompt snapshots.
- Stencils (`drift-template-expect`, `drift-template-judge`, the drift paragraphs in the darn and Webster stencils): the files are in the diff. I did not re-read their text against D16, because that is a wording check for the handler.
- Commit messages and `Pattern-Departure` trailers: one finding below.

**Findings**

1. **LOW, consistency** — `Pattern-Departure` trailers name an entry that does not exist.
   - Where: commits `e2dcd747` (card 8) and `b7f922f6` (card 15) carry `Pattern-Departure: PATTERN-fabric-vocabulary`.
   - Evidence: `PATTERN.md` and `pattern/` hold no `PATTERN-fabric-vocabulary` (`ls pattern` shows only `PATTERN-fabric-boundary.md`).
   - The rule the trailers invoke, that a non-owner file carries no side-named spelling, is `PATTERN-fabric-boundary.md:24`.
   - Under the rubric's "trailer naming an unknown entry" rule, this counts as a finding.
   - The substance holds. The renames to `CodeHead`, `RecordsChangedSince`, `LoomRecordsDirs` and `RecordsPathsChangedSince` are what that real entry demands (`trailercommit.go:109`, `driftwiring.go:204-206`).
   - The commit message cannot be amended in this range. Suggested fix: record the departure in the fixer report or summary under the correct entry name `PATTERN-fabric-boundary`.

No other affected-set gaps found.

**Tool calls**
1. Grep for removed names across the repo, to find leftover readers outside the diff.
2. Grep for `PATTERN-parent-directive` and Verify-seat wording, plus the `docs/` diff, to confirm the docs updates and no dangling references.
3. Grep for stale prose about removed agents, verbs and acceptances in `contracts`, `internal`, docs and `pattern`, and for drift `gate_key` in the recipes.
4. Grep for `PATTERN-fabric-vocabulary` and the `HubOnlyWayForward` wiring in `PATTERN.md`, `pattern/` and `configengine`, to see whether the cited departure entry exists.
5. Grep for side-named identifiers and the `fabric-boundary` text, to confirm which entry actually forbids them.
