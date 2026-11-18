# Card 1 — status-strand-spec-builder

**Create:**
- `plan:internal/loomengine#StatusStrandPlacement` -> `type StatusStrandPlacement int`
- `plan:internal/loomengine#StatusStrandOverview` -> `const StatusStrandOverview StatusStrandPlacement = 0`
- `plan:internal/loomengine#StatusStrandBesideRuns` -> `const StatusStrandBesideRuns StatusStrandPlacement = 1`
- `plan:internal/loomengine#StatusStrandAddSpec` -> `func StatusStrandAddSpec(l *lymcwd.Location, exe string, placement StatusStrandPlacement) reedengine.AddSpec`
- `plan:internal/loomengine#statusSidecarFile` -> `const statusSidecarFile = "status-strand.json"`
- `plan:internal/loomengine#StatusSidecarPath` -> `func StatusSidecarPath(l *lymcwd.Location) string`
- `plan:internal/loomengine#statusStrandCmd` -> `func statusStrandCmd(sh shell.Shell, exe string) string`
- `plan:internal/loomengine#TestStatusStrandAddSpec` -> `func TestStatusStrandAddSpec(t *testing.T)`
- `plan:internal/loomengine#TestStatusStrandCmd` -> `func TestStatusStrandCmd(t *testing.T)`

**Edit:**
- `internal/loomengine#LoomStatusStrandName`

**Uses:**
- `internal/loomengine#LoomScratchDir`

**Intent:**
D2's one builder of the loom status strand's pane command and add spec, and the one declaration of its sidecar path, in `internal/loomengine`, which both `internal/loomcli` and `internal/orchcli` already import.
Declare them beside `LoomStatusStrandName` in `internal/loomengine/driver.go`.

`StatusStrandPlacement` is the caller's choice of where the strand goes: `StatusStrandOverview` or `StatusStrandBesideRuns`, declared as one iota block whose zero value is the overview.
`StatusStrandAddSpec` builds the spec from the worktree location `l`, the running lym binary `exe` and the placement:
- `NameOverride` is `LoomStatusStrandName`, `IfAbsent` stays false, and no size, segment or color is set.
- `Cmd` is `statusStrandCmd(shell.ForGOOS(), exe)`.
  `statusStrandCmd` moves here from `internal/loomcli` unchanged: `sh.Invoke(exe)`, then the quoted `loom` and `checklist`.
- `StatusStrandOverview` gives `render.Display{Anchor: render.AnchorOverview}`, with no window group.
- `StatusStrandBesideRuns` gives `render.Display{Anchor: render.AnchorOwnWindow, KeepViewer: true, Window: <recipe>}`, the recipe read through `shedrun.ReadSeed(l, shedrun.SelfRunID)`.
  A read error or an absent seed leaves `Window` empty, so reed names the window by the role, `loom-status`, as today.
  The group holds this strand alone, so `WindowSize` stays 0.

`StatusSidecarPath` returns `filepath.Join(LoomScratchDir(l), statusSidecarFile)`, the path `statusSidecarPath` in `internal/loomcli/statusbuild.go` returns today; card 2 deletes that copy.
The recipe is read from the seed and never named in code, per `PATTERN-no-hard-coded-choice`.
`LoomStatusStrandName`'s doc comment, which says the orch bring-up reads the role only to take the overview window from that strand, is rewritten:
the bring-up takes that window and adds the strand back beside the runs strand, and both it and `lym loom start` pass the role as the unless-name of reed's `AddStrandUnless`.

`TestStatusStrandAddSpec` is a tier 0 table test over a `locationkit` worktree in `t.TempDir()`, seeding through `shedrun.WriteSeed` with `shedrun.RecipePawl` and `shedrun.DriverGo`.
Every row asserts `NameOverride`, `IfAbsent` false and `Cmd` equal to the shell-composed `<exe> loom checklist`, and the row's exact `Display`:
- overview with a pawl seed: the overview anchor, no `KeepViewer`, no group;
- beside-runs with a pawl seed: the own-window anchor, `KeepViewer` and `Window` `pawl`;
- beside-runs with a seed of `shedrun.RecipeTapestry`: the own-window anchor, `KeepViewer` and `Window` `tapestry`, so a builder that names a recipe in code fails;
- beside-runs with no seed: the own-window anchor, `KeepViewer` and no group;
- beside-runs with garbage bytes at `shedrun.SeedFile(l, shedrun.SelfRunID)`: the own-window anchor, `KeepViewer` and no group.
It also asserts `StatusSidecarPath(l)` equals `filepath.Join(LoomScratchDir(l), "status-strand.json")`.

`TestStatusStrandCmd` is a tier 0 table test that carries the rows of `internal/loomcli#TestStatusStrandCmd` onto the moved function, `shell.Posix()` and `shell.Pwsh()` with `C:\lym.exe` included, so the pwsh composition stays covered on a POSIX gate.

**ImpactSummary:** only `LoomStatusStrandName`'s doc comment changes among existing members; its value and every reader stay as they are, and the new builder has no caller until cards 2 and 3.

**Commit:** 1: loomengine: one builder of the loom status strand's spec, naming its own window for the recipe
