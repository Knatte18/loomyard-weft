# Card 9 — sampler-wiring

**Create:**
- `plan:internal/loomcli#samplerStamp` -> `func samplerStamp(l *lymcwd.Location, runID, recipe, row string) func() (int, error)`
- `plan:internal/loomcli#samplerDeps` -> `func samplerDeps(l *lymcwd.Location, runID, recipe, row string, spec func() (shuttleengine.Spec, error)) loomshed.SamplerDeps`
- `plan:internal/loomcli#openRecordWatch` -> `func openRecordWatch(path string) <-chan struct{}`
- `plan:internal/loomcli#TestWire_SamplerRecipe` -> `func TestWire_SamplerRecipe(t *testing.T)`
- `internal/loomcli/samplerwiring_test.go`
- `plan:internal/loomcli#TestOpenRecordWatch` -> `func TestOpenRecordWatch(t *testing.T)`
- `plan:internal/loomcli#TestWireLanding_FillsSamplerSessionSeams` -> `func TestWireLanding_FillsSamplerSessionSeams(t *testing.T)`

**Edit:**
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireLanding`
- `internal/loomcli/doc.go`

**Extends:**
- `docs/overview.md`

**Uses:**
- `internal/loomcli#sessionLiveness`
- `internal/loomcli#sessionRegistrar`
- `internal/loomcli#sleepContext`
- `internal/loomcli#pauseCovered`
- `internal/loomcli#loomProgram`
- `internal/loomcli#openLandRequestWatch`
- `internal/loomcli#hubLocation`
- `internal/loomcli#seedDarnHubConfig`
- `internal/loomcli#writeHubGateConfig`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#intentionallyNil`
- `plan:internal/loomcli#discussionRecordCheck`
- `plan:internal/loomrecipe#EngineRow`
- `plan:internal/samplerengine#SessionSpec`
- `plan:internal/samplerengine#EntryStamp`
- `plan:internal/shedrun#SamplerRecord`

**Intent:**
D3, D5 and D6: a sampler run's Sampler-Session row gets its seams.
`samplerStamp(l, runID, recipe, row)` reads the run's status file under its lock and the recipe's rows through `loomrecipe.Rows`, and returns `samplerengine.EntryStamp(status.History, row, predecessors)` with `predecessors` the rows whose `OnDone` is `row`; an absent status file answers 0.
`samplerDeps` fills `loomshed.SamplerDeps` with `shedrun.SamplerRecord` and its lock, `spec`, the stamp above, `DirtyPaths` over `verifytree.DirtyPaths(l.WorktreePath())`, `CheckRecord` from `discussionRecordCheck(l, recipe)`, `MailAgent` posting as the `LOOM` program through creel as `pullRequestDeps` does (`PATTERN-agent-messaging`), `PauseCovered` over `pauseCovered(l)` for the slug, `Now` `time.Now` and `Sleep` `sleepContext`; every closure reads on demand and opens nothing at build time, since `wire` also runs for status and pause.
`wire`, for a recipe with a SamplerSession row (`loomrecipe.EngineRow`), loads the hub's `sampler.yaml` strictly through `samplerengine.LoadConfig`, refusing a malformed file before any row builds with the error and the way forward `fix the key with lym config sampler (the editor) or lym config sampler --set <key>=<value> from the prime, then run lym loom resume`, since a value the hub tuned survives `lym config reconcile`, and sets `c.env.Sampler` from `samplerDeps` with a `spec` closure that builds `samplerengine.SessionSpec` at call time from the stencils dir, `c.parentName`, the slug, the worktree, the decision record path `runRecordOf` gives and the loaded config resolved against the registry `wire` holds.
`wireLanding` sets `c.env.Sampler.Sessions` to `c.runner`, `SessionLive` to `sessionLiveness(c.reed.Status)` and `RegisterSession` to `sessionRegistrar` registering `runproc.RoleSession` under the slug (`PATTERN-run-process-owner`), as it does for PR-Review, when the recipe has the row.
`wireLanding` also sets `c.env.Sampler.Watch` to `openRecordWatch(shedrun.SamplerRecord(...))` for such a recipe, since both building paths pass through it, the `run` verb's `loomPreRun` and the `step` verb's `buildLoomShed`, and `status` and `pause` never call it; `openRecordWatch` creates the record's directory, watches it through `fswatch` for the record's base name, and returns nil (leaving the row on its timer) with a Warn when the watch cannot open, in the shape of `openLandRequestWatch`.
`buildLoomShed` needs no change of its own, so it is not edited.
`TestOpenRecordWatch` (`//go:build tier0`, in a new `samplerwiring_test.go` that carries only that line's tag) writes the record through `samplerengine.UpdateRecord` after opening the watch and asserts a delivery within the test's deadline; a path whose directory cannot be created (under a regular file) returns nil.
`TestWireLanding_FillsSamplerSessionSeams` (in `wiring_publish_integration_test.go`, `//go:build tier1`, beside `TestWireLanding_TellsLanguageSkills` and over its real hub and pair) wires a sampler seed and asserts after `wireLanding` that `Sampler.Sessions`, `SessionLive`, `RegisterSession` and `Watch` are set, and on a darn seed that they stay nil (D5 acceptance on the session's registration and the wake).
`TestWire_SamplerRecipe` (in `wiring_darn_test.go`, `//go:build tier0`, carrying the same `//lym:seam` marker lines as `TestWire_DarnRecipe` plus `//lym:seam internal/loomshed#SamplerDeps`) wires a sampler seed over a hub holding darn.yaml (`verify_attempts` 5), gate.yaml and the sampler template, and asserts: the sampler seams `wire` owns are set and the four `wireLanding` owns are nil; `VerifyTreeScope` and `VerifyMergeBase` are set and `DarnVerifyAttempts` is 5 (D6 acceptance: Code-Burler-Fix gets darn's tree scope and budget); `Sampler.CheckRecord` is set and `DarnTable` nil; `Sampler.Stamp()` answers 1 after a status file with a Record-Write Done is written; and a hub `sampler.yaml` with `agent: "opus[[bad"` fails `wire` naming the key and `lym config sampler`.
`loomcli/doc.go` gains a paragraph on a sampler run's wiring, and `docs/overview.md` a sentence that the row's session is kept like PR-Review's and wakes on the record's file event.

**ImpactSummary:** new sampler-only wiring branches in wire, wireLanding and buildLoomShed taken only for a recipe with a SamplerSession row; tapestry, darn and pawl wiring is unchanged.

**Commit:** 9: loom wiring fills the Sampler-Session row's seams
