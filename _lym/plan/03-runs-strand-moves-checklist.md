# Card 3 — runs-strand-moves-checklist

**Create:**
- `plan:internal/orchcli#reedStrandOps.AddStrandUnless` -> `func (o reedStrandOps) AddStrandUnless(spec reedengine.AddSpec, unlessName string) (reedengine.Strand, bool, error)`
- `plan:internal/orchcli#fakeStrands.AddStrandUnless` -> `func (f *fakeStrands) AddStrandUnless(spec reedengine.AddSpec, unlessName string) (reedengine.Strand, bool, error)`
- `plan:internal/orchcli#orchCLI.addChecklistBesideRuns` -> `func (c *orchCLI) addChecklistBesideRuns(exe string, build statuspane.BuildIdentity) string`
- `plan:internal/orchcli#selfRunUnfinished` -> `func selfRunUnfinished(l *lymcwd.Location) bool`
- `plan:internal/orchcli#wayForwardWarning` -> `func wayForwardWarning(what string, cause error, verbs ...string) string`

**Edit:**
- `internal/orchcli#strandOps`
- `internal/orchcli#reedStrandOps`
- `internal/orchcli#newReedStrandOps`
- `internal/orchcli#orchCLI.ensureRunsStrand`
- `internal/orchcli#runsStrandWarning`
- `internal/orchcli#fakeStrands`
- `internal/orchcli#fakeAdd`
- `internal/orchcli#fakeStrands.ReplaceStrand`
- `internal/orchcli#TestStart_EnsuresTheRunsStrandBeforeTheOrch`
- `internal/orchcli/doc.go`
- `docs/overview.md`

**Uses:**
- `plan:internal/loomengine#StatusStrandAddSpec`
- `plan:internal/loomengine#StatusStrandBesideRuns`
- `plan:internal/loomengine#StatusSidecarPath`
- `internal/loomengine#LoomStatusStrandName`
- `internal/orchcli#newStartHarness`
- `internal/orchcli#startHarness.run`

**Intent:**
D1: when `ensureRunsStrand` takes the overview window from the loom status strand, it adds the checklist back in a window of its own, in the same call.

Strand seam: `strandOps` gains the interface method `AddStrandUnless(spec reedengine.AddSpec, unlessName string) (reedengine.Strand, bool, error)`.
`reedStrandOps` gains an `addUnless` field of that function type, `newReedStrandOps` sets it to `reed.AddStrandUnless`, and the new `reedStrandOps.AddStrandUnless` calls it.
`fakeAdd` gains an `unless string` field, empty for a plain add.
`fakeStrands` gains `addedUnless []fakeAdd`, each recording its unless-name, a `held bool`, an `addUnlessErr error` and a `replaceErr error`.
`fakeStrands.AddStrandUnless` appends `add-unless` to `calls`, then answers `addUnlessErr` when set, `(Strand{GUID: "held-" + unlessName}, true, nil)` recording nothing when `held` is set, and otherwise records the spec and answers guid `added-` plus the spec's `NameOverride` with false.
`fakeStrands.ReplaceStrand` answers `replaceErr`, recording nothing, when it is set.

Warnings: `wayForwardWarning` logs `orch: <what>` with the cause at Warn, as `runsStrandWarning` does today, and returns `<what> (<cause>); way forward: fix the cause it names, then run ` followed by each verb in backticks, the verbs joined by `, then `.
`runsStrandWarning` becomes `return wayForwardWarning(what, cause, "lym beam start")`, so every existing warning's text stays byte for byte.

`ensureRunsStrand`'s `loomStatus != nil` branch, the session holding a loom status strand and no runs strand:
1. A failed `ReplaceStrand` returns `wayForwardWarning("could not take the overview window from the loom status strand, so the beam overview and the loom checklist may be missing", err, "lym beam start", "lym loom start")`, since `ReplaceStrand` persists the old strand's removal even when its add fails.
2. After a successful replace it writes the runs sidecar as today, keeping a failure's warning instead of returning it.
3. Then, when `selfRunUnfinished(c.location)`, it calls `c.addChecklistBesideRuns(exe, build)`, whatever the runs sidecar write returned.
4. It returns the non-empty warnings of steps 2 and 3 joined by `; `, empty when both are.
The other branches (current runs strand kept, stale one replaced, none added) are unchanged and add no status strand.
`ensureRunsStrand`'s doc comment, which says it replaces a loom status strand instead of adding beside it and touches no other strand, is rewritten to say it takes that strand's window and adds the strand back in a window of its own while the self run is not done.

`selfRunUnfinished` polls `shedrun.StatusFile(l, shedrun.SelfRunID)` once through a fresh `statuspane.NewStatusCache()` and reports `Found` with a `State` other than `shedengine.StateDone`, the finished test `runStatusUnfinished` in `internal/fabriccli/anchordeps.go` makes.
An absent, unreadable or done status is false, so a done run's leftover checklist is not brought back.
`c.location` is the session's own worktree, which `wireCLI` sets from the location every `BeamUp` caller passes, never the caller's.

`addChecklistBesideRuns` builds `loomengine.StatusStrandAddSpec(c.location, exe, loomengine.StatusStrandBesideRuns)` and calls `c.strands.AddStrandUnless(spec, loomengine.LoomStatusStrandName)`:
- an error returns `wayForwardWarning("could not add the loom checklist in a window of its own beside the runs strand, so the checklist is missing", err, "lym loom start")`;
- a skipped add returns "", writing no sidecar, since a concurrent `lym loom start` added a strand holding the full `loom-status` name first;
- otherwise it writes `statuspane.WriteSidecar(loomengine.StatusSidecarPath(c.location), strand.GUID, build)`, so the next `lym loom start` keeps that strand, and a failure returns `wayForwardWarning("could not record the loom checklist strand's build, so the next start replaces it", err, "lym loom start")`.
No failure removes the runs strand or its sidecar, and none fails `lym beam start`.
On a blocked or failed self run, the named `lym loom start` also spawns a driver and resumes the run, as any start of a halted run does; the record names that verb as the way back to the checklist.
Bound: the re-add names only the fixed `loom-status` role, only right after this call took that strand's window, and only for a self run not done.

`TestStart_EnsuresTheRunsStrandBeforeTheOrch` (tier 0) gains row fields: `selfState shedengine.State`, which, when set, writes a self seed (`shedrun.WriteSeed` with `shedrun.RecipePawl`, `shedrun.DriverGo` and a `parent` param) and a status of that state at `shedrun.StatusFile(h.cli.location, shedrun.SelfRunID)`, inlined in the row loop through `json.Marshal`, `os.MkdirAll` and `os.WriteFile`; `selfStatusGarbage`, which writes the seed and garbage bytes as that status; `replaceErr`, `addUnlessErr` and `held` for the fake; `blockRunsSidecar` and `blockStatusSidecar`, which create a directory at the runs sidecar path or at `loomengine.StatusSidecarPath(h.cli.location)` so the write fails; `wantStatusAdd bool`; `wantStatusSidecarGUID`; and `wantNoWarnings`.
The loop asserts the `addedUnless` specs apart from the runs specs: when `wantStatusAdd`, exactly one, with unless-name `loom-status`, `NameOverride` `loom-status`, `Cmd` the shell-composed `<exe> loom checklist` and `Display` `{AnchorOwnWindow, KeepViewer: true, Window: "pawl"}`; otherwise none.
It asserts the status sidecar records `wantStatusSidecarGUID` with the current build, or is absent when that is empty; a blocked sidecar path is not read.
`wantNoWarnings` asserts the report carries no warning.
The existing rows "no runs strand adds one", "a live runs strand of this build is kept", "a dead runs strand is replaced" and both "stale runs strand ... loom status strand precedes/follows" rows gain `selfState` running, and their unchanged `wantCalls`, holding no `add-unless`, pin that they add no status strand.
The existing row "a loom status strand holding the overview is replaced by the runs strand" keeps no `selfState`: it is the absent-self-run case, calls `replace` and no `add-unless`, and writes no status sidecar.
New rows, each with the live strand `{GUID: "r1", Name: "loom-status", Live: true}` and no runs strand, so the loop's replaced-guid check holds:
- running self run: calls `replace`, `add-unless`, then the orch's; runs sidecar `replaced-runs`; `wantStatusAdd`; status sidecar `added-loom-status`;
- blocked self run: the same, so a stuck pawl's checklist comes back too;
- done self run: calls `replace` and no `add-unless`; no status sidecar;
- `selfStatusGarbage`: calls `replace` and no `add-unless`; no status sidecar;
- `blockRunsSidecar` and `addUnlessErr` with a running self run: both the runs sidecar warning naming `lym beam start` and the re-add warning naming `lym loom start`, joined by `; ` in one report warning;
- `addUnlessErr` with a running self run: the warning `could not add the loom checklist in a window of its own beside the runs strand, so the checklist is missing (<err>); way forward: fix the cause it names, then run ` + "`lym loom start`"; runs sidecar `replaced-runs`; no status sidecar;
- `blockStatusSidecar` with a running self run: a warning opening `could not record the loom checklist strand's build` and naming `lym loom start`; `wantStatusAdd`; runs sidecar `replaced-runs`;
- `blockRunsSidecar` with a running self run: the runs sidecar warning naming `lym beam start`, and still `wantStatusAdd` with status sidecar `added-loom-status`;
- `replaceErr` with a running self run: the warning `could not take the overview window from the loom status strand, so the beam overview and the loom checklist may be missing (<err>); way forward: fix the cause it names, then run ` + "`lym beam start`, then `lym loom start`"; calls `replace` and no `add-unless`; no sidecars;
- `held` with a running self run: calls `replace`, `add-unless`; no `addedUnless` spec; no status sidecar; `wantNoWarnings`.

Docs: `internal/orchcli/doc.go`'s runs-strand sentence and `docs/overview.md`'s orch and batten entries, which say ensuring the runs strand replaces a `loom-status` strand, say instead that it takes the overview window from that strand and, while the worktree's own run is not done, adds the checklist back in a window of its own named for the run's recipe, a failure being a warning naming `lym loom start`.

**ImpactSummary:** `strandOps` gains one method whose only implementers, `reedStrandOps` and the test fake `fakeStrands`, are on this card; `ensureRunsStrand` keeps its signature and its one caller in `start.go`, and the existing runs-strand warnings keep their text.

**Commit:** 3: beam start: move the loom checklist to its own window instead of replacing it
