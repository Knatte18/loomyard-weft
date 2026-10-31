```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff forces outside the lines it touches.

**Per change, the set found and whether it is covered**

| Change | Set found | Covered |
|---|---|---|
| `NewPawl`, `PawlRouting`, `ServedByLoom` removed | A grep of `*.go`, `*.md` and `*.yaml` outside `_lym`, `.lym` and `.scratch` finds no live reference. `arm.go`, `loomrecipe.go` and the `New` refusal text are updated. | Covered |
| Board entry snapshot removed (`DiscussionBoardEntry`, `BoardEntrySnapshot`, `WriteSnapshot`, `BoardEntryPath`, `board-entry.md`) | No reference remains in code or stencils. Hits remain only in `docs/research/...` benchmark reports, which are measurement records and not live consumers. The rubric's third form and the PR-Review stencil comment are updated. `envkit`, `entries_pawl.go`, `beamawait.go` and `deps.go` are updated. | Covered |
| Pawl recipe: `contract: loom`, Record-Write row, Code-Review perch removed | `LoomBuildEngines` and the `loomcontract.go` message are updated. `holdFreeProducers` and `recordFreeProducers` are updated, and the doc in `boardengine/doc.go` says the recipe's test pins them. The `pawl`-keyed branches I read in `arbiterprompt.go:20`, `landingdeps.go:81` and `:86`, `sharedbootstrap.go:341` and `:368`, and `drivewiring.go:115` are all still consistent with the new recipe. | Covered |
| `project_directive` marker | Both stencil banners, the discussion stencil, `darn-template-write.md`, `DiscussionTable`'s `Optional` and `DarnTable`'s `Optional` list it. The chair and advisor stencils are not changed, which matches the summary that only the lead and the darn writer are told. | Covered |
| `runRecordOf` sites | Site by site: `wireLightweight`, `wire` (`PatternRecordPath`, `DecisionRecordPath`, `SupportLogPath`, `DiscussionHistory`, `RecordWrite`), `arbiter.go`, `landingdeps.go:255`, `decision.go` `preRun` and `beamwiring.go` all use the record. Two sites still read the discussion path directly. `wire`'s `CommitDiscussion` is the discussion-commit seam, and `RelevantPath` is a discussion-only file. `darnwiring.go:164` hardcodes `RecipeDarn`, which is correct there. The `ValidateDiscussion` call and `DiscussionTable` at `wiring.go:~678` still use discussion paths, and are not reached by a beam run. | Covered |
| Orch role stencil, `PATTERN.md`, package docs, `docs/overview.md` | The orch stencil, the `PATTERN.md` entries (`PATTERN-loom-contract` loses "bar pawl" and `PATTERN-project-record` is added) and the docs for `boardengine`, `loomcli`, `loomrecipe`, `beamcli`, `landingshed` and `pawlengine` are updated. The decision record declares no PATTERN changes, so there is no declared-change landing to check. | Covered |

**Findings**

1. **LOW / consistency**
   - **Location:** commit `3c8d5a3c`, message trailer.
   - **Summary:** the commit ends with a `Co-Authored-By: Claude Opus 5.5` trailer, but the previous commit on this branch, `f666ba87`, is titled "Landing commits end with a plain credit block, not Co-Authored-By". I did not open that commit's diff or any rule document to confirm the rule applies to this commit.
   - **Evidence:** `git log` output in my context (the trailer of `3c8d5a3c`) and the title of `f666ba87` in the gitStatus snapshot.

2. **LOW / consistency**
   - **Location:** `internal/loomcli/landingdeps.go:78-82` and `internal/loomcli/arbiterprompt.go:20-21`.
   - **Summary:** these two spots still key on `recipe == shedrun.RecipePawl` for behavior, such as the nil `reviewFollowsReroute` seam and the arbiter's "pawl run" description. The rest of the diff moves pawl to recipe-row-based selection (`runRecordOf` via `recipeHasEngine(..., BeamAwait)`). I judged the rows `RecipePawl` and `BeamAwait` equivalent today, so this is not a defect. The two mechanisms should not drift.
   - **Evidence:** `landingdeps.go:78-82`, `arbiterprompt.go:20`, and `projectrecord.go:607-609` (the row-based check).

I found no other member of the affected set left wrong.

**Tool calls (3)**
- `grep` for removed symbols and `board-entry.md` references across the repo: the diff alone shows what was deleted, not what still references it.
- `grep` for `RecipePawl`, `project_directive` markers and `pattern/` and `docs` stat: to find other pawl-keyed branches and marker sites outside the diff.
- `sed` of `arbiterprompt.go`, `landingdeps.go`, `sharedbootstrap.go`, `drivewiring.go`, plus the `discussiontable.go` grep and diff: to judge the remaining `RecipePawl` branches, which the diff does not show.
