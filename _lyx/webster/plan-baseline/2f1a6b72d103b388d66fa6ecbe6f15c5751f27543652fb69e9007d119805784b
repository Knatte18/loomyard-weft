# Card 12 — overview-entry-numbers

**Create:**
- `internal/battencli#overviewStage`
- `internal/battencli#overviewSource.resolveChildPair`
- `internal/battencli#overviewSource.resolveChildRecipe`

**Edit:**
- `internal/battencli#overviewRun`
- `internal/battencli#overviewLineOf`
- `internal/battencli#overviewSource`
- `internal/battencli#newOverviewSource`
- `internal/battencli#overviewSource.resolveChildStatus`
- `internal/battencli#overviewSource.poll`
- `internal/battencli#TestRenderOverview`
- `internal/battencli#TestOverviewSource_ReadsThePrimesRunsFromDisk`
- `internal/battencli/cli.go`
- `docs/overview.md`

**Uses:**
- `internal/loomrecipe#EntryNumbers`

**Intent:**
The prime runs overview, the one pane a batten run has, prefixes the stage cell with the stage's entry number (D4).
`overviewStage` returns `<n>. <row>` for a row of the named recipe, numbered by `loomrecipe.EntryNumbers` over `loomrecipe.Rows(recipe)`, and the row alone when the recipe is empty, unreadable or does not hold the row.
`overviewLineOf` draws a batten stage as `overviewStage(run.Recipe, row)`, for example `7. Land`.
While batten is in `Run-Shed` and the child's status is found, it draws the child's row as `<Run-Shed's number>.<child number> <row>`, for example `6.5 Plan-Write`, the child number being the one the child's checklist gives the row (card 9), so a child row inside a review segment takes its segment's number (`6.6 Plan-Burler-Review`).
The dotted form keeps a batten number and a child number from reading alike.
A stage whose number cannot be read shows the name alone: a batten row of a run whose own seed was unreadable, a batten row not in the batten recipe, and a child row with no child recipe known or not in it.
With batten in `Run-Shed` and the child's status not found, the stage is batten's own row, `6. Run-Shed`.

`overviewRun` gains `Recipe`, the recipe the run's own seed names, which `poll` sets once the seed is read (a shown run's read seed is always batten's), empty while it is unreadable; and `ChildRecipe`, the recipe the child's seed names, empty when unknown.
`poll` fills `ChildRecipe` only while batten is in `Run-Shed`, through `resolveChildRecipe`.
`resolveChildPair` is `resolveChildStatus`'s worktree resolution moved out, resolving the task worktree of a slug once while its directory exists; `resolveChildStatus` calls it.
`overviewSource` gains the seam field `childPair`, which `newOverviewSource` sets to `resolveChildPair` and a test replaces, and a by-slug cache of the child recipes read.
`resolveChildRecipe` takes the pair from `childPair` and reads the child's seed through `shedrun.ReadSeed(pair, shedrun.SelfRunID)` (PATTERN-shed-run-directory), caching a decoded seed's recipe by slug, since a seed is write-once; a pair or read error returns the error and an absent seed an empty recipe, which `poll` logs at Debug and draws as the name alone.

The time columns keep `FormatMinutes`.

`TestRenderOverview`'s rows set `Recipe: shedrun.RecipeBatten` and gain the numbers: the `Land` row reads `7. Land`, the `Worktree-Create` row `1. Worktree-Create`, the Run-Shed row with a loom child recipe reads `6.5 Plan-Write`, and stages not in the batten recipe (`Publish`, `Plan`) keep the name alone.
New rows: a child row in a review segment reads `6.6 Plan-Burler-Review`; a Run-Shed row with no child recipe, and one whose child row is not in its child recipe, show the child's row alone; a Run-Shed row whose child status is not found reads `6. Run-Shed`; a run with no `Recipe`, its seed unreadable, shows `Land` alone.
`TestOverviewSource_ReadsThePrimesRunsFromDisk` keeps the real `resolveChildRecipe`: it replaces `childPair` with a `locationkit.Location` under a temp directory where it writes the child's seed through `shedrun.WriteSeed(pair, shedrun.SelfRunID, ...)` naming the loom recipe, so the first frame's run-a reads `6.5 Plan-Write`.
The frames after the child's status file is removed keep their `Run-Shed` stage, now `6. Run-Shed`.
It removes the child seed file after the first frame and restores the child status, and the frame still reads `6.5 Plan-Write`, from the cached recipe.
A second Run-Shed run whose child pair holds no seed reads its child row alone, and one whose `childPair` answers an error reads its child row alone too.

The package doc in `cli.go` and `docs/overview.md`'s sentence listing the overview's cells say the stage cell carries the entry number.

**ImpactSummary:** the overview's stage cell text and one seed read per Run-Shed run in `internal/battencli/overview.go`, plus one extracted helper `resolveChildStatus` now calls; the time columns and the orch's overview are unchanged.

**Commit:** 12: batten overview: entry numbers in the stage cell
