# Card 2 — loom-start-shared-builder

**Create:**
- `plan:internal/loomcli#statusStrandOps` -> `type statusStrandOps interface`
- `plan:internal/loomcli#ensureStatusStrandOn` -> `func ensureStatusStrandOn(ops statusStrandOps, location *lymcwd.Location, executable func() (string, error), currentBuild func() (statuspane.BuildIdentity, error)) error`
- `plan:internal/loomcli#fakeStatusStrandOps` -> `type fakeStatusStrandOps struct`
- `plan:internal/loomcli#fakeStatusStrandOps.Status` -> `func (f *fakeStatusStrandOps) Status() (reedengine.StatusResult, error)`
- `plan:internal/loomcli#fakeStatusStrandOps.ReplaceStrand` -> `func (f *fakeStatusStrandOps) ReplaceStrand(guid string, spec reedengine.AddSpec) (reedengine.Strand, error)`
- `plan:internal/loomcli#fakeStatusStrandOps.AddStrandUnless` -> `func (f *fakeStatusStrandOps) AddStrandUnless(spec reedengine.AddSpec, unlessName string) (reedengine.Strand, bool, error)`
- `plan:internal/loomcli#TestEnsureStatusStrandOn` -> `func TestEnsureStatusStrandOn(t *testing.T)`

**Edit:**
- `internal/loomcli#loomCLI.ensureStatusStrand`
- `internal/loomcli#TestSmokeStatusStrandAcrossDriverSeeds`
- `internal/loomcli/doc.go`
- `docs/overview.md`

**Delete:**
- `internal/loomcli#statusStrandAddSpec`
- `internal/loomcli#statusStrandCmd`
- `internal/loomcli/statusbuild.go`
- `internal/loomcli#TestStatusStrandAddSpec`
- `internal/loomcli#TestStatusStrandCmd`

**Uses:**
- `plan:internal/loomengine#StatusStrandAddSpec`
- `plan:internal/loomengine#StatusStrandOverview`
- `plan:internal/loomengine#StatusStrandBesideRuns`
- `plan:internal/loomengine#StatusSidecarPath`
- `internal/loomcli#resolveStatusStrandAction`
- `internal/orchcli#RunsStrandName`

**Intent:**
`lym loom start`'s status-strand step builds its spec through card 1's builder and adds the strand through reed's `AddStrandUnless`, so a status strand added concurrently, as by D1's re-add in `lym beam start`, is kept (D1, D2).

`statusStrandOps` is the slice of reed the step uses: `Status() (reedengine.StatusResult, error)`, `ReplaceStrand(guid string, spec reedengine.AddSpec) (reedengine.Strand, error)` and `AddStrandUnless(spec reedengine.AddSpec, unlessName string) (reedengine.Strand, bool, error)`, which `*reedengine.Engine` satisfies as it stands.
`ensureStatusStrandOn` holds today's body of `ensureStatusStrand`, with these changes:
- The placement is `loomengine.StatusStrandBesideRuns` when the strands `Status` returns hold one that `agentname.Matches` addresses as `orchcli.RunsStrandName`, else `loomengine.StatusStrandOverview`, the choice `statusStrandAddSpec` makes today.
  The spec is `loomengine.StatusStrandAddSpec(location, exe, placement)`, so the beside-runs window carries the recipe's group.
- The add is `ops.AddStrandUnless(spec, loomengine.LoomStatusStrandName)` in place of `AddStrand`.
  A skipped add, true as its second result, means a strand holding the full `loom-status` name appeared between the `Status` read and the add: it counts as kept, returns nil and writes no sidecar.
  A strand under the legacy bare name never reaches the add, since `findStatusStrand` finds it first and the step keeps or replaces it.
  An add error is returned as today.
- The replace path, its warning on failure and the keep path are unchanged.
- The sidecar path is `loomengine.StatusSidecarPath(location)`.
`loomCLI.ensureStatusStrand` becomes `return ensureStatusStrandOn(c.reed, c.location, os.Executable, statuspane.CurrentBuild)`; its one caller in `start.go` is unchanged.

Delete `statusStrandAddSpec`, `statusStrandCmd` and `internal/loomcli/statusbuild.go` (`statusSidecarFile`, `statusSidecarPath`), now declared once in `internal/loomengine`, with their tests `TestStatusStrandAddSpec` and `TestStatusStrandCmd`.
Card 1's `TestStatusStrandCmd` carries the command rows, card 1's `TestStatusStrandAddSpec` the display per placement, and this card's `TestEnsureStatusStrandOn` the placement chosen from the strand list, the old `OtherStrandsLeaveItTheOverview` row included.
`TestSmokeStatusStrandAcrossDriverSeeds` calls `statusStrandAddSpec("sleep 3600", nil)` twice: both become `loomengine.StatusStrandAddSpec(loc, "lym", loomengine.StatusStrandOverview)` with `Cmd` then set to `"sleep 3600"`, the test's `loc` being the fixture location already in scope.

`TestEnsureStatusStrandOn` is a tier 0 table test, added in `sharedbootstrap_test.go`, over a `locationkit` worktree in `t.TempDir()` and `fakeStatusStrandOps`.
The fake answers `Status` from a strand list, records each `ReplaceStrand` and `AddStrandUnless` call with its spec and unless-name, answers a replace with guid `replaced-status` and an add with guid `added-status`, and, when its `held` field is set, answers the add with `(Strand{GUID: "held-status"}, true, nil)`; its `addErr` field, when set, is the add's error, and its `replaceErr` field, when set, is the replace's error.
The executable seam answers `/fixed/lym` and the build seam a fixed identity, or, when the row sets `buildErr`, that error.
Rows:
- no strands and no seed: one add, unless-name `loom-status`, the overview display; the sidecar at `loomengine.StatusSidecarPath` records `added-status` and the build;
- only a live orch strand `tst:main:orch`: one add with the overview display, so a strand list that holds no runs strand keeps the overview placement;
- a live `loom-status` strand whose guid and the current build the sidecar records: no replace, no add, nil, and the sidecar unchanged;
- `buildErr` set and no strands: nil, one add, and no sidecar file;
- `replaceErr` set and a dead `loom-status` strand: nil, one replace, no add and no sidecar file;
- a live `runs` strand and a pawl seed: one add with the own-window anchor, `KeepViewer` and `Window` `pawl`, the D2 criterion for `lym loom start` with a runs strand present; the sidecar records `added-status`;
- `held` set: nil error, no replace, the add called once and no sidecar file, the D1 criterion for the role taken between the read and the add;
- a dead `loom-status` strand beside a live `runs` strand and a pawl seed: one replace of the dead strand's guid with the beside-runs spec and no add; the sidecar records `replaced-status`;
- `addErr` set: the error returned and no sidecar file.

Docs: `internal/loomcli/doc.go`'s status-strand paragraph and `docs/overview.md`'s `start` bootstrap lines (the loom module's four steps and the bootstrap entry) say that the window of its own, in a beam whose `runs` strand holds the overview, is named for the run's recipe.
They also say that the add is skipped when a strand holding the role appeared since the read.
The header of `sharedbootstrap_test.go`, which says no test there drives `ensureStatusStrand`, and the line of `smoke_drivingsurface_test.go`'s header saying its branches are pinned only through `resolveStatusStrandAction`, are rewritten to name `TestEnsureStatusStrandOn`.

**ImpactSummary:** `ensureStatusStrand` keeps its signature and its one caller in `start.go`; the deleted helpers' only other callers are their own tests and `TestSmokeStatusStrandAcrossDriverSeeds`, all on this card.

**Commit:** 2: loom start: build the status strand's spec once and keep a concurrently added one
