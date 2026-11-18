# Card 2 — loom-start-shared-spec

**Edit:**
- `internal/loomcli#statusStrandAddSpec` -> `func statusStrandAddSpec(cmd string, strands []reedengine.StrandStatus, l *lymcwd.Location) reedengine.AddSpec`
- `internal/loomcli#loomCLI.ensureStatusStrand`
- `internal/loomcli#TestStatusStrandAddSpec`
- `internal/loomcli#TestSmokeStatusStrandAcrossDriverSeeds`
- `internal/loomcli/doc.go`
- `docs/overview.md`

**Delete:**
- `internal/loomcli#statusStrandCmd`
- `internal/loomcli#TestStatusStrandCmd`
- `internal/loomcli/statusbuild.go`

**Uses:**
- `plan:internal/loomengine#ChecklistStrandCmd`
- `plan:internal/loomengine#ChecklistBesideSpec`
- `plan:internal/loomengine#ChecklistSidecarPath`
- `plan:internal/loomengine#ChecklistRecipe`

**Intent:**
`lym loom start` builds the checklist strand from card 1's shared pieces, so the checklist it places beside a beam's runs strand is named for the run's recipe (D1).

- `statusStrandAddSpec` gains the worktree's location `l` and keeps its placement rule.
  With no strand matching `orchcli.RunsStrandName` it returns today's overview spec, `render.Display{Anchor: render.AnchorOverview}` with no `Title`, and reads no seed, since a normal task run's checklist keeps the overview window named as today.
  With a runs strand it returns `loomengine.ChecklistBesideSpec(cmd, loomengine.ChecklistRecipe(l))`.
  Taking the location rather than a recipe puts the seed read inside the function a test drives, so the title can never come from `loomCLI`'s recipe field, which defaults to tapestry before a seed exists.
  Its doc comment says the beside window is named for the seed's recipe, empty when the seed is missing or unreadable.
  The header comment of `internal/loomcli/bootstrap.go`, which names the command-string builder it holds, drops that clause, since the builder moves to `internal/loomengine`.
- `ensureStatusStrand` reads the sidecar at `loomengine.ChecklistSidecarPath(c.location)`, composes the command with `loomengine.ChecklistStrandCmd(shell.ForGOOS(), exe)`, and passes `c.location` to `statusStrandAddSpec`.
  The seed is written before this call in `start`, so a pawl run's start reads `pawl`.
- `statusStrandCmd`, its test `TestStatusStrandCmd` and the whole `internal/loomcli/statusbuild.go` (`statusSidecarFile`, `statusSidecarPath`) are deleted; card 1 carried the command's rows over to `internal/loomengine#TestChecklistStrandCmd`.
- `TestSmokeStatusStrandAcrossDriverSeeds` passes a nil location at both `statusStrandAddSpec` calls; with nil strands no seed is read and they still get the overview spec.
- `TestStatusStrandAddSpec` (tier0) gets a row field for the seed: a recipe to write with `shedrun.WriteSeed(l, shedrun.SelfRunID, …)`, or the bytes `{not json` to write at `shedrun.SeedFile(l, shedrun.SelfRunID)`, over a `locationkit.Location` in `t.TempDir()` per row.
  It asserts the whole returned `AddSpec` by equality: `NameOverride` `loomengine.LoomStatusStrandName`, `Cmd` `watch-cmd`, no segment, `IfAbsent` false and the display.
  The rows:
  - the two existing overview rows, now with seed recipe `pawl`, still want `render.Display{Anchor: render.AnchorOverview}` with no `Title`;
  - the existing runs-strand row, with seed recipe `pawl`, wants `render.Display{Anchor: render.AnchorOwnWindow, KeepViewer: true, Title: "pawl"}`, which fails if the beside case stops reading the seed;
  - "a runs strand with no seed leaves the beside title empty" wants the same display with an empty `Title`;
  - "a runs strand with an unreadable seed leaves the beside title empty" writes `{not json` and wants the same.
- `internal/loomcli/doc.go`'s status-strand sentence and `docs/overview.md`'s two bootstrap lines (the `start` paragraph under `lym loom` and "the bootstrap" bullet) say the window of its own is named for the run's recipe.

**ImpactSummary:** Local to `internal/loomcli`: one production caller of the re-signed `statusStrandAddSpec` (`ensureStatusStrand`) and two test callers; the deleted `statusStrandCmd`, `statusSidecarPath` and `statusSidecarFile` have no caller outside this card.

**Commit:** 2: loom start builds the beside checklist from loomengine's spec, titled by the seed's recipe
