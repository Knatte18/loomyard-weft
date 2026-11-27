```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens, covering what the diff's changes force outside the touched lines.

**Set found per change**

1. **New per-task key `beam.run_watch`** (beamsetting, `LoadConfig`, `CheckLocalValues`, `checkTaskSetting`, `LookupRunWatch`)
   - Covered: `pattern/PATTERN-config-layers.md` (the declared entry landed), `boardcli` `run_settings` help and its test, `template.yaml`, `dobbyengine/doc.go`, the `beamsetting` package doc, the `tasksetting.go` and `Config` comments, and the `configreg` test.
   - Needs no change: `PATTERN.md` index line; `PATTERN-landing.md` (names `pull_request` only); `boardengine` and `hubgeom` beam modes (do not enumerate this key as shown); `plugins/ly/skills/board/SKILL.md:266` (describes only how to ask for a landing notice).
2. **Hub keys `gate_hold_factor` and `gate_hold_min` and their refusals**
   - Covered: `ConfigSpec` categories, `template.yaml`, `LoadConfig` doc, `defaultsWith`.
   - Other `Config` literals in harnesses call `d.deps.LoadConfig` stubs, so the range refusal does not reach them.
3. **`RunView` and `ChildReading` fields**
   - Covered: `DobbyRuns`, `readNoticeChild` and their tests.
4. **New `[run watch]` mail prefix and gate-hold `[dobby notice]`**
   - Covered: `contracts/orchrole/orch-role.md` Notices (line 73's "start no Monitor" stays consistent) and `docs/overview.md` dobby paragraph.
   - No Go code outside `dobbyengine` parses the prefixes in the searches run.
   - The hub's `orch-template-role.md` lives outside the repository, as the plan overview states.
5. **Done-run clear gains three holds**
   - Covered: `dobbyengine/doc.go` and the `clearDoneRuns` comment.
   - **Not covered:** the `lym batten cleanup` help (finding 1).
6. **`State.Watches`**
   - Covered: `state.go`. The tolerant rewrite is unaffected.
7. **`gateslot.Holder.Marker`**
   - Covered: `gateslot/doc.go` and tests.
   - `loomcli/reviewstatus.go` `formatHolders` takes `Holder` values and needs no field-specific change.
   - The other acquirers keep no marker, as the field doc says.
   - `gatecli/doc.go`'s `LYM_GATE_TEST_PID` passages stay true.
8. **`Deps.RunWatch` and `Deps.GateHolds`**
   - Covered: the hub wiring. Nil seams leave existing harnesses unchanged.
9. **Commit messages and departures**
   - The one `Loosened-Check` trailer names the allowlist entry that commit 4 removed again; no `dobbyengine` entry remains in `cmd/lym/testonlycode_linux_test.go`.
   - No `Pattern-Departure` trailer, and no stencil changed.

**Findings**

1. **LOW, consistency.** `internal/battencli/cleanup.go:682-684`.
   - The `lym batten cleanup` long help still says the dobby removes a done run "itself once it has counted its landing and decided its landing notice".
   - The diff adds more conditions: no watch line still owed, the run's `run_watch` read at least once when the watch is wired, and the tick's watch step recorded.
   - An operator with a done run held by an owed line (up to an hour of awake time) or by a level that never reads is told the clear should already have happened.
   - Evidence: `internal/dobbyengine/donerun.go:52-55` adds the holds; `internal/battencli/cleanup.go:683` has the unchanged sentence.

**Tool calls**
1. `git grep` for per-task keys and notice prefixes: failed on pathspec syntax; the manifest had no repo-wide list of readers.
2. `git grep` for done-run clear and holder-record mentions: same pathspec failure; the same gap.
3. Rerun of call 1 with working excludes: the manifest lacked the docs and help texts naming these keys and prefixes.
4. Rerun of call 2 with working excludes: the manifest lacked the other descriptions of the clear and the holder record.
5. `sed` and `git grep` over the `batten cleanup` help, the board skill and per-task or prefix mentions: those texts were not in context.
6. `git grep` for run-notice tags and `Monitor` mentions in contracts, package docs and patterns: not in context.
