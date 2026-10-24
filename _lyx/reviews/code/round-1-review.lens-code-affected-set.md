```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: for each change in the diff, the readers, callers, docs and tests it forces, and whether the diff left any of them wrong.

**Per-change sets (covered or not)**

| Change | Set found | Result |
|---|---|---|
| `OutcomeSkipped` history entry | Every `History` reader outside `shedengine`: `sheddriver/relaunch.go:41` (`verdictInEffect`), `sheddriver/drive.go:760`, `loomshed/loompreflight.go:165` (`isGotoReentry`), `loomcli/darnwiring.go:128` (`latestDarnOutcome`), `battenshed/classify.go:216` (`countDone`), `reportengine/rounds.go:337` (`countHandSteps`), `battenshed/notice.go:373`, `battenshed/innerrun.go`. | Covered. None treats `skipped` as a stop or a call. `latestDarnOutcome` reads only Darn-Write, which is ungated. `countDone` serves episode rows, which carry no entry gate. `verdictInEffect` can't meet a skip, since it needs only goto entries added since the stop key. Inside `shedengine`, `gotoAdmitted` and `episodeStuckCount` handle it. |
| `EnteredRow` on `Status` | `RenameProducers` and `namesRetiredRow` (updated), `Goto` and `Relaunch` (updated), `persistRouted` (clears it), `rerouteStuck` (moved to `persistRouted`). | Covered. |
| `guard_tier` retired | A grep of non-test Go, yaml and md found no leftover reader. | Covered. Only the load-drop, the prune and their tests remain. |
| `tree_scope_verify` and `ErrTreeScopeWithoutTrees` retired | Only the migration, the load-drop and the tests remain. The `lyx config darn` refusal sits in `configcli/hubwide.go` (−20 lines). | Covered. |
| `CommandDeps.Forced` and the switch read | `battencli/wire.go`, `battencli/reopen_window_test.go:57`, `testkit/envkit/envkit.go:441`, `shedadapters/command_test.go:119`. | Covered. `Forced` appears nowhere in the grep. |
| Retired gate keys `landing`, `final`, `round` | Non-test Go, md and yaml under `internal`, `contracts`, `docs` and `cmd`. | Covered. Only the migration table and its test rows name them. |
| Package-load tag lists | `impactset/derive.go:59`, `impactset/guards.go:110`, `planglyph/typesload.go:82`. | Covered. All three list `tier0,tier1,tier2,tier3,integration,tmux,llm`. |
| `lyx gate test` manual flags | `contracts/specs/loom-plan-spec.md:322` and `:537` keep `--tier`. | Covered. Those are manual or card-verify forms, which D3 keeps. |
| `gatecli` doc examples | `internal/gatecli/doc.go:200` and `:210` still show `-tags integration`, `-tags tmux` and `-tags llm` in example `gate.yaml` commands. | Finding 1. |
| `docs/overview.md` gate line (461) | It names `--gate` and `--guards`. | Finding 2. |

**Findings**

1. LOW, `consistency`: `internal/gatecli/doc.go:200` and `:210` show example tier commands tagged `integration`, `tmux` and `llm`.
   - The task retags every test file to `tierN`, and the D13 transition commands carry both spellings.
   - Under the new disjoint tiers, the `tmux`-only example at line 210 compiles no `tier2` file.
   - It also trips the new empty-tier failure (D7).
   - Evidence: `internal/gatecli/doc.go:200` and `:210`.

2. LOW, `consistency`: the `docs/overview.md:461` gate entry omits the green-record carry.
   - It also omits `--gate` and the `diff` scope's base chain.
   - `gate report` has one hit in the file.
   - The CLAUDE.md "Docs land in the same commit" rule asks `overview.md` to follow observable CLI behavior. `gatecli/doc.go` was updated and the overview was not.
   - Evidence: `docs/overview.md:461` (the grep for `carried` found nothing in the file).

**Tool calls (5 of 6)**

1. Grep for `guard_tier`, `tree_scope_verify`, `CommandDeps` and `Forced`, and for history readers of `Outcome`, because the manifest did not show the readers outside `shedengine`.
2. Grep for the retired keys and for old tag spellings in non-test code.
3. Grep for `History` readers, plus `sed` of `sheddriver/relaunch.go` and `loomshed/loompreflight.go`, to check whether skipped entries break them.
4. Grep for `entered_row` and `skipped` in the specs, and for `--tier` and `--all` in stencils, recipes and docs.
5. `sed` of `battenshed/classify.go`, `reportengine/rounds.go` and `loomcli/darnwiring.go`, to confirm the gated rows are not read by those counters.
6. Grep count of `gate report`, `entry gate` and `carried` in `docs/overview.md`, to confirm finding 2.
