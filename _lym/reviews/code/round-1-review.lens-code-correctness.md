```yaml
lens: code-correctness
outcome: reported
```

Task: lens `code-correctness`. For each changed function or branch I ran one concrete case, and the list comes first.

**Per function or branch, case run, verdict**

- `undiscussedBlockGate` (`entries_block.go`):
  - Seam returns (false, nil): the gate holds with an empty reason, so Discussion runs. Correct.
  - Seam returns (true, nil): the gate does not hold, so the row skips to `SkipTo` Drift-Baseline with `undiscussedSkipReason`. Correct.
  - Seam returns an error: the gate returns (false, "", err). `enter` (`shedengine/entry.go:38-41`) fails the run and leaves the row not entered, so a resume evaluates the gate again. Correct.
  - Nil seam: `requireSeam` refuses at build time. Correct. `wire` (`wiring.go:880`) is the only non-test `Env` that sets the seam, and the same Env already needs `ApproveDiscussion`, so no existing tapestry caller regresses.
- `seedDiscussedRecord` (`discussionrecord.go`):
  - Board entry flagged discussed with its file present: the function writes the record, writes the support log if it is missing and commits. Correct.
  - Absent entry, unflagged entry, or flagged entry with no file: `DecisionRecord` returns `found=false` and the function returns (false, nil) with nothing written. Correct. The flagged-entry-with-no-file case is silently treated as undiscussed, so Discussion runs. That is safe, not a defect.
  - `OpenHub` or `DecisionRecord` read error: returned, so the gate fails the run. Correct, and it matches `approveDiscussion`'s use of `OpenHub`.
  - `MkdirAll`, write or commit failure: each is returned and the gate fails. A resume re-evaluates the gate, and the writes are idempotent. The file is already written when the commit fails, but the resume rewrites it, so no state is left inconsistent.
  - Support log already present: left alone. `Stat` errors other than not-exist are returned. Correct.
  - Goto back onto Motif after the run's own approval: the working record is overwritten with the board copy. This is already disclosed in the landing summary. The board copy also holds the entries added by `decision add`, so nothing is lost. Acceptable, not a finding.
- `finishedBoardStatus` and `boardStatus` (`wiring.go`):
  - tapestry, darn, pawl and sampler recipes: all four carry a Finalize row (grep of `engine: Finalize` in `pawl-recipe.yaml`, `sampler-recipe.yaml` and `darn-recipe.yaml`, and tapestry per the existing Finalize-based design). The finished status is "" and nothing is written. Correct.
  - Motif recipe, with state `done`: writes `draft` once, because the `lastBoardStatus` dedupe holds. A failed write only warns. Correct.
  - Unknown recipe name: `recipeHasEngine` falls back to the tapestry recipe, so it writes nothing. Correct.
  - `c.recipe` is set at `arm.go:132` from the seed. Both commit-status call sites go through `c.commitStatusDeps`. Correct.
- `checkLoomContract` and `recordOnly` (`loomcontract.go`):
  - Motif: a record row only, with no build row, no Finalize row and no `verify_source`. It passes. Correct.
  - A recipe with a build row but no Finalize row, or the reverse: `recordOnly` is false and the old clauses refuse it as before. Correct.
  - A loom recipe with only a record row, plus a typo that drops the build and Finalize rows by accident: it now passes silently. This is the intended design, so I raise no finding.
- `Expand` (`expand.go`): the sub-recipe's `contract`, `verify_source`, `amendment_target` and `profile_recipe` are dropped and the parent's are kept. Case run: tapestry with `Motif` inlined. Tapestry's top-level keys are unchanged, and the snapshot test covers this. The only validation that disappears is the refusal that was removed on purpose. Correct.
- `arbiterRecipeDescription` (`arbiterprompt.go`): the Sampler, Plan-Write and Pawl branches come first, so motif (no Finalize) reaches the new branch. Darn and tapestry have Finalize and fall through to the code-change description. Correct.
- `Rows` and `RecipeEngines` (`names.go`): the `RecipeArras` entry in `embeddedRecipes` was removed, and `RecipeMotif` stays registered. `RecipeEngines` now iterates tapestry, darn, pawl, sampler and motif. Correct.
- `PastDiscussionApproval` (`discussionapproval.go`): only the comment changed. I did not run the expanded-row read; that is covered by my context only through the comment and the `discussionapproval_test.go` +1 line. No defect found.
- `RecipeNames` (`seed.go`): the sorted list is now batten, darn, motif, pawl, sampler, tapestry. Correct. The `"arras"` entry in `seed_test.go:174` is a deliberate "former name" refusal row.
- `reportengine` / `tokencount`:
  - Tapestry's phase table counts the `Motif` family as discussion.
  - The Discussion rows' block path is `Motif/…`. I assumed the report's `rowFamily` matching reads it as it read arras's. I did not verify this.
  - Case run: a tapestry run whose Discussion rows have BlockPath `Motif`. The family key is `rowFamily(NameMotif)`, and the rows are named `Discussion-*`, so the exact and family tables still match them through `loomPhases`. No defect found.

**Findings**

None.

**Unverified (not enough to raise as a finding)**

- Nothing in my context shows how batten's Land and teardown treat a child run that ends with no Finalize, and the writer lists this as a hand check.

**Tool calls**

1. `sed`, `grep` on `discussionrecord.go`, `loomcli/*.go` and the recipe yaml files: `approveDiscussion`'s `OpenHub` handling, where `c.recipe` is set, the other `ApproveDiscussion` Env builders, and which recipes carry a Finalize row. The manifest lacked all four.
2. `sed`, `grep` on `arm.go` and `shedengine/*.go`: `recipeHasEngine`'s fallback and the entry-gate and `SkipTo` mechanism.
3. `Read` of `shedengine/entry.go` lines 15-95: whether a gate error leaves the row not entered so a resume re-evaluates it. The manifest lacked this.
