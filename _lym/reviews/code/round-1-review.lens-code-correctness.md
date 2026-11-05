```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness** (function-by-function cases for the motif board-fixes diff)

| Function or branch | Case run | Result |
|---|---|---|
| `battenrecipe.RowsBeforeRunShed` | The batten recipe | Correct. It returns the five rows before Run-Shed, with the gated Command rows included. |
| `battenrecipe.RowsBeforeRunShed` | A recipe with no Run-Shed row, or one that fails to parse | Panics by design, as `RecipeEngines` does. |
| batten seam (`commitstatus.go` `newCommitStatusSeam`), producer in the pre-run rows | `Seed-Child` with state `blocked` | Writes `blocked · Seed-Child`. The write runs before the marker skip. A failed write only warns and the commit goes on. Correct. |
| batten seam, producer `Run-Shed`, `Land` or `Worktree-Teardown` | Each of the three | Writes nothing. The board keeps the last pre-run status until the inner run's start write. Matches the design. |
| `battenCommitStatusDeps.SetBoardStatus` | `OpenHub` fails | The error is returned to the seam, which warns. Correct. |
| `Board.SetRunStatus` | Entry already carries the status; entry done; entry absent | Writes nothing in each case. Correct. |
| `Board.SetRunStatus` | Entry carries `discussed` or `draft` | Overwritten by a run status. This is existing behaviour. |
| `shedengine.stepLocked` start write | Fresh seed, `CurrentProducer==""`, state running | `started=false`, so it persists the first row as running and calls `CommitStatus` before `Call`. Correct. |
| `stepLocked` start write | Resume with the row named and state running | No extra write. Correct. |
| `stepLocked` start write | First row gated and held | `enter` only marks it entered, and the re-read keeps `started=false`, so the start write fires. Correct. |
| `stepLocked` start write | First row gated and skipped | `persistRouted` has named the next row, so `started=true` and there is no double write. Correct. |
| `stepLocked` start write | `pause_before` on the first row, or a cancelled context | These exit at step 3 before the start write. Correct. |
| `stepLocked` start write | `persistMarked(true…)` with a `ResumedBy` set on a fresh running seed | `resumableHalt(running)` is false, so `PendingBy` is not set. Correct, but I did not read `resumableHalt` itself. |
| `finishedBoardStatus` and loom seam | Recipe with no Finalize row, state done | `boardStatus` returns `discussed`. `SetRunStatus` overwrites `running · Discussion-Bouncer`, and `lastBoardStatus` makes a re-fire a no-op. Correct. |
| `finishedBoardStatus` and loom seam | Recipe with a Finalize row | Writes nothing. Correct. |
| `ComputeLayers` | A task with status `discussed`, with and without dependencies | Layer is `Discussed` in both cases. A dependent of it lands in `A`. Correct. |
| `RenderOrder` | Bucket order | Letters 2–26, Z=27, Discussed=28, Draft=29, done=30, so the order is right. |
| `entryCell` | Draft and discussed | The `switch` is exclusive and correct. |
| `layerSection` | `discussedLayer` | Returns its own heading. Correct. |
| `prepareRunSettings` | Motif with no profile, with the template | `Default("motif")` is `solid` and the motif block is applied. Correct. |
| `prepareRunSettings` | Motif naming a profile with no motif block | Refused, wrapped in `ErrInvalidRunSettings`. Correct. |
| `prepareRunSettings` | Empty recipe name | The old `ProfileRecipe("")` mapped to tapestry. `Recipe` now comes from the seed through `seamchild.go:140`, so an empty name should not reach it. Not provable from what I read. |
| `template.yaml` motif blocks | Compared with each profile's tapestry Discussion values | They match (0/1, 1/2, 2/3, 3/4) and `defaults.motif: solid` is set. Correct. |

**Findings**

1. **MEDIUM, class `design`: a `discussed` entry can still be started by hand.** `internal/battencli/startguard.go:96` and `:164-170` refuse a run or step with no status file only when the board status equals `StatusDraft`.
   - A finished motif used to be `draft`, so `lym batten run <slug>` was refused with the release command named.
   - It is now `discussed`, so the guard does not fire, and the run claims the entry again.
   - This contradicts the README text the diff adds (`render.go:700-701`, "nothing starts it until its status is cleared") and the record's "Like a draft" intent.
   - Dobby does not start it, but the hand path is open. The guard's `Draft` probe and its refusal text need to cover `discussed`.

2. **LOW, class `design`: existing hubs change behaviour silently.** The template is seed-only (the writer's summary acknowledges this).
   - A hub whose `profiles.yaml` has no `defaults.motif` now runs a motif task naming no profile on the hub's `loom.yaml`. Before, it ran on tapestry's `solid` block.
   - A task naming a profile that has no motif block is refused at Seed-Child.
   - The refusal names the way forward, per the summary, but the diff adds no automatic migration. Judge it against PATTERN `config-reinterpretation` and `sparse-hub-base` if the handler considers this a meaning change.

I found no other defects on the start-write ordering, the board-status failure paths, or the layer and render changes.

Tool calls (all read-only):
1. `grep` of `render.go` for layer constants, and `sed` of `runsettings.go` with `grep` for `prepareRunSettings` callers. My context did not show whether the Discussed table gets the Recipe and After columns, or who calls `prepareRunSettings`. The columns turned out to be unconditional (no layer branch beyond Running and Ready).
2. `sed` of `wire.go`, `grep` of `ValidateRecipe`, and `sed` of `runprofile.Default`. I needed to know where `Recipe` is filled and what `Default` returns for a missing entry.
3. `grep` of `seamchild.go` for `settings.Recipe`. I needed to see where `RunSettings.Recipe` is set.
4. `sed` of `seamchild.go` lines 110-142. It confirmed `Recipe` comes from the seed's resolved recipe.
