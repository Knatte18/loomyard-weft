```yaml
lens: existing-paths
outcome: reported
```

**Existing-paths lens:** I traced each state, key and file the record introduces or changes through the code that already reads, writes or resets it.

**Paths traced, per decision**

- D1, `computeScope` early return for scopers other than `go`, read by `planTierRuns` and `runTest`: survives. The guard scan runs only under `go`, and `--guards` is refused under other scopers (plan-writer note).
- D1, `openGreen(..., scoperGo)` feeding `subtract` and `recordPass`: survives. Under `dotnet`, package subtraction and package entries stay off (`green.go` `!g.scoperGo` branches).
- D4, `recordPass` case "command lacks `{targets}` → Whole" with a `{filter}`-only tier under a narrowed `dotnet` scope: survives. D4 runs that tier whole with the tier filter only, so the whole entry is true.
- D4/Constraints, `recordPass` case "targets == all_targets → Whole" under a narrowed `dotnet` scope: covered. The record's Constraint overrides this case explicitly.
- D4, the explicit-targets and darn tree-scope `recordPass` whole case with `{filter}` set to the tier filter: survives.
- D4, `vetStepRun` and the guard step building their own `Tier`: survives. Both stay go-only and carry no `filter`.
- D5, the inert-only empty scope under `go` against the guard scan: survives. Guards stay nil, so `runTest` scans the module and every guard test counts as outside. That equals today's Markdown-only result, where `impactset` returns no module and guards outside an empty set (`internal/impactset/doc.go:23,28`).
- D5, `with()` pruning against the carried whole entry for the current tree: survives. The entry keeps HEAD's tree and commit, so `newestWholeAncestor` finds it later.
- D5, `openGreen` returning nil on a dirty tree: survives, matching the record's "a dirty worktree reads no record".
- D6, `runGreenAware` ordering (subtract, then `runWithSlots`) against "the first opted-in tier whose command will run": survives. Carry is known before the hold.
- D6, `exe.inherited` and the unslotted case outside a hub: survives. The build runs in whatever hold the tier run has.
- D6, `WriteRunRecord` stem `tier%d` and `ReadRunRecords`' one production reader, `report.go`: survives as a named change. D6 states the new naming and the separate report total.
- D6, `FailedTier`'s one reader, `landingshed.recordFailure`: survives as a named change. D6 states `build: true`, and a build timeout read as timed out writes no record (`verifygate.go` `result.TimedOut` early return), which is consistent.
- D6, `printFailedRuns` and `haltExitCode`: survives. A build failure doesn't halt the call, and the closing line names `build`.
- D7, the CPU watchdog over the tied child: survives. It signals only binaries ending `.test` (`internal/gatecli/watchdog.go:31-32`), so MSBuild nodes, VBCSCompiler and testhost are never sent SIGQUIT.
- D8, `dropCovered` and `verifytree.Verify`'s verified-tree skip: both are named in D8's Bound.
- D10, Finalize carrying after Merge-In with no commit between: survives. `_lyx` is a link to the weft worktree and is untracked in the code tree, so run-record writes between the two gates don't change HEAD's tree.
- Constraints, `build: {}` in the template against `configengine`'s shape check and `hubreconcile` reconcile: **breaks** (Finding 1).

**Findings**

1. **MEDIUM** · design · `decision-record.md` Constraints, the `PATTERN-config-layers` bullet (line 189), and Decision 6 (line 109)
   - **Summary:** the claim that the template's `build: {}` stops reconcile from pruning a hub's `build` values depends on `build` being an open map. The record never says so.
   - **Why it breaks:** `configengine` checks a layer's shape against the template. Only a spec's `OpenMaps` are "mappings whose keys the user owns" (`internal/configengine/doc.go:71`, `:93-95`). `tiers: {}` is safe because it is an open map.
   - **Counterexample:** a hub writes `build={command: ..., timeout_min: 5}`. The template's `build` mapping has no `command` key, so the shape check refuses it or reconcile prunes it, unless `build` joins the gate spec's `OpenMaps` or the template spells out its sub-keys.
   - **Fix:** the Decision or Constraint states which of the two mechanisms carries `build`'s keys, with `LoadConfig`'s refusals (D6 line 125) as the validator either way.

Tool calls:
1. `internal/gatecli/watchdog.go` header: which processes the watchdog signals, to judge D7's warm nodes against an existing path. The manifest lacked it.
2. Run-record readers and the `WriteRunRecord` stem: who reads the D6 build record. The manifest named only the writer.
3. Other parsers of the tier summary line and the closing line, outside `gateslot`: whether a reader besides `FailedTier` misreads the new build line.
4. `_lyx` link and tracked status: whether run-record commits change the code tree between Merge-In and Finalize (D10).
5. `internal/impactset/doc.go` guard and Markdown rules: how the guard step behaves on an inert-only empty scope (D5).
6. `internal/configengine/doc.go` shape and open-map rules: whether `build: {}` keeps a hub's sub-keys (Constraints).
