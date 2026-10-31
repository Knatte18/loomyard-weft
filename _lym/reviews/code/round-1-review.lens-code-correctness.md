```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens over the diff. For each changed function or branch I ran one concrete case. I found no BLOCKING or MEDIUM defects, and four LOW ones.

## Per-function list

| Function or branch | Case run | Result |
|---|---|---|
| `ProjectRecordHeld` (`runlock.go`) | non-beam project; nil status; `Preflight`; `Worktree-Create`; `Record-Write`; `blocked · Beam-Await` | Correct. The status parser rejects nil and `Kind`/`Beam` gate the rest. A halt keeps the producer half, so it stays held. |
| `checkRunLocks` body check | beam at Preflight, body edit | Allowed, correct. |
| `checkRunLocks` body check | beam at Record-Write, body edit | Refused with `ProjectRecordLockedError`, correct. |
| `checkRunLocks` body check | beam at Beam-Await, title-only edit | Allowed, correct. |
| `checkRunLocks` body check | beam at Parent-Align | `RunHolds` is true, so the full lock applies first and the new branch is never reached. Correct. |
| `holdFreeProducers` with `Record-Write` | members joining during Record-Write | Allowed. Matches the summary, correct. |
| `runRecordOf` | recipe `""`, tapestry, darn, pawl | `""` and unknown names fall back to tapestry through `recipeHasEngine`, so they get the discussion record. Pawl gets the project record. Correct. |
| `commitRunRecord` | project record, repeat commit | Stages `_lym/project/&lt;slug&gt;`. The code comment says a repeat commit over an unchanged directory is a no-op. Correct. |
| `recordWrite.Call` with the new `record.dir` | `_lym/project/&lt;slug&gt;` absent | `removeTornFiles` tolerates `IsNotExist`, and `AtomicWriteBytes` runs `MkdirAll`. Correct. |
| `recordWrite.Call` resume | record present, `Record-Write` re-entered | Writes only missing files, so `decision add` entries survive. Correct. |
| `decisionParent` / `preRun` | beam target, caller strand equals the beam's nearest live orch | Passes. Correct. |
| `preRun` `filepath.Rel("_lym", record.rel)` | `_lym/project/s/decision-record.md` | Yields `project/s/decision-record.md`. Correct. |
| `decisionReceivers` | beam with members, a member pair worktree absent | Absent pairs are skipped in `memberPairs`. `liveRunStrands` errors propagate, which is acceptable. Correct. |
| `projectRecordPaths` | pair under a nested beam chain; a cycle; a beam whose worktree or record is absent | Nearest first. `seen` plus `ParentBeam`'s own cycle guard stop loops. `NamedWorktree` needs no worktree on disk, so absence returns found=false. Correct. |
| `ProjectRecordDirective` | empty records | Returns `""`, so the prompt bytes are unchanged. |
| `ProjectRecordDirective` | one record | Substitutes the token with a plain replace and wraps in newlines. Correct. |
| `DiscussionTable` / `DarnTable` `Optional` | empty marker value | Both lists include `project_directive`, so an empty value does not fail the fill. Correct. |
| `beamDescription` | no decisions | No extra section, so the output is unchanged. Correct. |
| `beamDescription` | decisions present | Appended after the commit bullets. Correct. |
| `beamDescription` | decision text heading levels against `ValidateDescription` | Not run (see the last section). |
| `NewBeamDescribe` | `ProjectDecisions` nil | Refused by the seam check. Correct. |
| `Call` error path | `ProjectDecisions` returns an error | Reported by `fail`, with no write. Correct. |
| `AddedDecisionTexts` | record file absent | Returns nil, nil. A missing record yields no section rather than an error, which is acceptable. |
| `wire` / `wireLightweight` `record.project` | pawl vs. other recipes | Pawl gets no `PatternRecordPath`, and `BeamDescribe` and `Pawl` deps are wired on `record.project`. Equivalent to the old `RecipePawl` test for the embedded recipes. Correct. |
| `loomrecipe.New` / `Routing` for pawl | no review segments | No `segmentBudgets` entry needed, so no failure. Correct. |
| `BeamAwait.landChecks` | `WriteSnapshot` removed | Blockers path unchanged. Correct. |

## Findings

**LOW / design: `internal/boardengine/runlock.go:65-67`.** `recordFreeProducers` makes the body editable again if a run is sent back to `Preflight` with `lym loom goto --to Preflight`.
- Record-Write then keeps the existing record, because it writes only a missing file.
- A body edit made in that window is silently never reflected in the project record.
- The lock is judged from the status alone, not from whether the record file exists.
- Evidence: `runlock.go:60-67` and `loomshed/recordwrite.go:126-146`.

**LOW / design: `internal/boardengine/runlock.go:65-67` with `internal/loomcli/decision.go:87`.** A beam that halts transiently at `Record-Write` before the record is written has its body locked.
- `decision add` refuses with `decisionWayRecordStart` until the record exists.
- The only way forward is a re-step of Record-Write.
- Evidence: `ProjectRecordHeld` includes `Record-Write`, and `recordWrite.fail` says to re-step.

**LOW / consistency: `internal/loomcli/decision.go:842-848`.** `preRun` now calls `seedRecipe(target)` for every `decision add`.
- A seed that cannot be read now aborts the verb with "read the run's seed".
- Before, the seed was read only inside the `hasPlan` closure, and its error became a normal refusal with a way forward.
- The new abort names no way forward.

**LOW / consistency: `internal/loomcli/decision.go` caller refusal.** For a beam with no resolvable orch, `NearestLiveOrch` returns an empty `Parent`, and the refusal reads "the caller is a task session". That is misleading wording for a beam, though the refusal itself is correct.

## Not verified
- Whether `summaryparser.ValidateDescription` accepts the added-entry headings that `AddedDecisionTexts` returns, under the new `## Project decisions` heading. `beamdescribe_test.go` in the diff may cover it.
- `memberPairs` depth coverage inside `battencli.ReadBeamLandBlockers`. I read only its call shape.

## Tool calls (6 allowed, 5 used)
1. Read `recipeHasEngine`, `AddedDecisionTexts`, `NearestLiveOrch` and `ReadBeamLandBlockers`: the manifest held only their call sites.
2. Read `seedRecipe`, `resolveSeedRecipe`, `NamedWorktree`, `ParentBeam` and the `discussiontable.go` diff: to check the absent-worktree, nearest-beam and `Optional` marker behaviours.
3. Read `recordWrite.Call`, `removeTornFiles` and `AtomicWriteBytes`: to check the new project directory case on a fresh beam.
4. Grepped `decision.go` for darn and loom references: to see how the verb picks its record checks per recipe.
5. Read `decision.go:600-700`: the `decisionVerb` flow, to check the caller guard and `rowWritten` handling for a beam.
