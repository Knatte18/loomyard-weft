```yaml
lens: code-affected-set
outcome: reported
```

Task: for each change the diff makes to a signature, output, message or file shape, list the callers, tests and docs it forces, and flag any the diff left wrong.

**Set per change**
- `Deps.Log` (`func(line)` to `func(path, line, rotate)`): the only production wiring is `internal/beamcli/dobbydeps.go:156`, and `passes.go:322` is its only call site. The test fakes in `loop_test.go:137`, `passes_test.go:209`, `tidy_test.go:73` and `loop_worktreegone_test.go:53` use the new signature. Covered.
- `Deps.CheckBinary` (new `beforeExec`/`failed` params, `selfexec.CheckWith`): the only wiring is `dobbydeps.go:101`, and `loop_test.go:132` uses the new signature. `Watcher.Check` is kept as a wrapper. Covered.
- `placeDobby` (returns `location` as well): the only caller is `beamcli/cli.go:395`. Covered.
- `battencli.DobbyLandings` (new `machine` param): the callers are `dobbydeps.go:154` and `dobbypairanchor_test.go:145`. Covered.
- `hubgeom.DobbyGeometry` (new `OutPath`, `PrimeLogPath`): `OutPath` and `PrimeLogPath` are read by `start.go` and `loop.go`, and `LogPath` by `dobbytrace.go`. The hubgeom test changed. Covered.
- `dobby.log` now holds only the dobby's own lines: the docs in `dobbyengine/doc.go:20`, `beamcli/doc.go:103-130` and `docs/overview.md:448` are updated. No other production reader of `LogPath` or `dobby.log` turned up. Covered.
- `State.StartTicks`, `State.LandingsMachineOnly`, `HistoryEntry.Machine`: these are tolerant `omitempty` fields. `Machine` is stamped at all three append sites in `shedengine` (`entry.go`, `reroute.go`, `run.go`). The two `goto.go` appends stay unstamped, as the field doc says. Covered.
- Gate key `adopt`: `template.yaml`, `goReadGates` (`tiers.go:87`), the `gateslot` and `gatecli` docs, `PATTERN-test-speed.md` and `envkit` all carry it. `contracts/specs/loom-plan-spec.md:318,326` list gate keys but only for the plan's sites, which `adopt` is not. Covered.
- The `lym adopt` spawn (`finalize` gate replaced by the hidden check): the `adopt.go` warning, the `cli.go` Long text, `docs/overview.md` and `adoptcli/doc.go` are updated. One stale header remains (finding F1 below).
- `suiteStart` gains `Record`: the envelope text in `cli.go` and `doc.go:116` name it. Covered.
- `MigrateHubBase` and `ReconcileHubWideAt`: `configreg` sets the hook for `gate` only, and the `configsync` doc and tests are updated. Covered.
- `selfexec` `CheckWith`: `selfexec/doc.go` and the test are updated. Covered.

**Findings**

- F1, LOW, consistency, `internal/adoptcli/adopt.go:1`: the file header still ends "...runs the beam steps of each adopted beam and starts the test suite". The diff changed what `adopt.go` starts, which is now the adopt check running the `adopt` gate, and updated the `deps.go` header, the `adoptDeps.StartSuite` comment and `startSuite`'s doc, but not this line. Evidence: `adopt.go:1`, set against the new `deps.go:1` header in the diff. Other names still say "suite" (`suiteStart`, `suiteLogName`), but those are identifiers and not wrong.

No other set member found wrong.

**Tool calls**
1. Grep for `gate finalize` and stale gate-key references, the `DobbyLandings`, `CheckBinary` and `placeDobby` callers, `dobby.log` mentions in docs and code, and `HistoryEntry{` appends without `Machine`. My context did not hold the callers or the repo-wide reference sets.
2. Grep for `LogPath`, `LogFileName` and `OutFileName` users, adopt/finalize wording, gate-key lists and `Deps.Log` users. These were needed to check other readers of `dobby.log` and stale text.
