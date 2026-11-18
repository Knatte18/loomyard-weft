# Card 1 — checklist-strand-spec

**Create:**
- `plan:internal/loomengine#ChecklistStrandCmd` -> `func ChecklistStrandCmd(sh shell.Shell, exe string) string`
- `plan:internal/loomengine#ChecklistBesideSpec` -> `func ChecklistBesideSpec(cmd, recipe string) reedengine.AddSpec`
- `plan:internal/loomengine#checklistSidecarFile` -> `const checklistSidecarFile = "status-strand.json"`
- `plan:internal/loomengine#ChecklistSidecarPath` -> `func ChecklistSidecarPath(l *lymcwd.Location) string`
- `plan:internal/loomengine#ChecklistRecipe` -> `func ChecklistRecipe(l *lymcwd.Location) string`
- `plan:internal/loomengine#TestChecklistStrandCmd` -> `func TestChecklistStrandCmd(t *testing.T)`
- `plan:internal/loomengine#TestChecklistRecipe` -> `func TestChecklistRecipe(t *testing.T)`

**Edit:**
- `internal/loomengine#LoomStatusStrandName`

**Uses:**
- `internal/loomengine#LoomScratchDir`

**Intent:**
Give the gate checklist strand one spec that `internal/loomcli` and `internal/orchcli` both build from (D1).
`internal/orchcli` cannot import `internal/loomcli`, which imports it, so the shared pieces live in `internal/loomengine` beside `LoomStatusStrandName`, in a new file `internal/loomengine/checklist.go`.
`internal/loomengine` may import `internal/reedengine`, `internal/reedengine/render` and `internal/shell`: none of them imports `internal/loomengine`, directly or transitively.

- `ChecklistStrandCmd` composes the checklist strand's pane command through the shell seam: `sh.Invoke(exe)`, then `sh.Quote("loom")` and `sh.Quote("checklist")`, space-separated.
  It is the body of today's `internal/loomcli#statusStrandCmd`, moved; card 2 deletes the old one.
- `ChecklistBesideSpec` returns the spec of the checklist strand placed beside the beam overview: `NameOverride` `LoomStatusStrandName`, `Cmd` cmd, and `render.Display{Anchor: render.AnchorOwnWindow, KeepViewer: true, Title: recipe}`, with no segment, no size and `IfAbsent` false.
  An empty recipe gives an empty `Title`, which reed reads as the role's name (card 4).
- `checklistSidecarFile` and `ChecklistSidecarPath` move today's `internal/loomcli/statusbuild.go` pair: the path is `filepath.Join(LoomScratchDir(l), checklistSidecarFile)`, so the sidecar stays under `.lym` at the path it has today (`PATTERN-durable-vs-ephemeral-state`).
- `ChecklistRecipe` returns the `Recipe` of the seed `shedrun.ReadSeed(l, shedrun.SelfRunID)` reads, and the empty string when the seed is missing or the read fails.
  It never defaults to a recipe: `loomCLI`'s own recipe field defaults to tapestry before a seed exists, and a strand its sidecar keeps is never re-specced, so a defaulted title would stick.
  A failed read is logged at Debug through `internal/logger` and is no error, since a missing title costs display only.
- `LoomStatusStrandName`'s doc comment says a beam's orch bring-up reads it to move that strand out of the overview window.
  The edit sits in this card, the first to touch the constant, because card 3 only uses it.

Tests, all tier0 in the `internal/loomengine` package:
- `TestChecklistStrandCmd` carries over `internal/loomcli#TestStatusStrandCmd`'s rows unchanged, against `ChecklistStrandCmd`.
- `ChecklistBesideSpec` gets no test of its own: card 2's `internal/loomcli#TestStatusStrandAddSpec` asserts the whole spec it returns, for recipe `pawl` and for the empty recipe, and card 3's `internal/orchcli#TestStart_EnsuresTheRunsStrandBeforeTheOrch` asserts it on the move.
- `TestChecklistRecipe` is table-driven over a `locationkit.Location` in `t.TempDir()`:
  a seed written with `shedrun.WriteSeed` with recipe `pawl` answers `pawl`;
  no seed answers the empty string;
  an unreadable seed, the bytes `{not json` written at `shedrun.SeedFile(l, shedrun.SelfRunID)`, answers the empty string.

**ImpactSummary:** `LoomStatusStrandName` changes in its comment only, so no reader of the constant is affected; the new file adds symbols nothing yet calls.

**Commit:** 1: Share the checklist strand's command, beside spec, sidecar path and recipe read in loomengine
