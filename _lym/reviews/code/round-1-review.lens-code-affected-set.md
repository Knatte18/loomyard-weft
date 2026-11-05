```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I traced what the diff's changes force outside the lines it touches. I found no defects.

**Per change, the set found:**

1. **Removed `StatusDiscussed`, `discussedLayer`, `isDiscussedStatus`, the `Discussed` README subsection and the `discussed` title span.**
   - Repo-wide grep over code, specs, stencils, skills and docs finds no live reference left.
   - The remaining `discussed` hits are all the `Discussed` flag, the decision-record `get --record` and `seedDiscussedRecord` paths, and the migration's legacy constant.
   - The layer, render, cli help, startguard, doc.go and overview.md text all match the code. Covered.
2. **`FinishedStatus` field, `finishedBoardStatus` and `boardStatus` replaced by `Finish`, `finishesAsSketch` and `sketchFinish`.**
   - No stale reader is left outside the touched files. The only other `boardStatus` hit is an unrelated test helper in `pairteardown`.
   - The touched loomcli tests are the other consumers, and the diff updates them. Covered.
3. **Built-in `sketch` type, handle letter `S`.**
   - Consumers of the type list are `LoadConfig`, `validateTypeLetters`, `Vocabulary.IsType`, `typeHeadings`, `leadingNoteTypes`, `IsSketch` in `text.go`, and `lym board labels`.
   - The diff updates the `boardengine` doc.go, the `boardcli` doc.go, `docs/overview.md` and the board SKILL.md for them. No stencil or spec under `contracts/` names the type or the old status. Covered.
4. **Run-lock exemption (`endsAsSketch`).** Callers are `checkRunLocks` only. The replay shares it through `boardCriticalSection`, and the run-lock section of doc.go states the rule. Covered.
5. **`migrateDiscussedStatus` in `boardCriticalSection`.** Doc.go, the `board.go` comment and `sketch_test.go` all describe it. Covered.
6. **PATTERN entries and declared changes.**
   - `PATTERN-refusal-way-forward` binds `internal/loom*`, and the new `sketchFinish` warning is not a refusal.
   - `PATTERN-tolerant-records` is untouched, since no `DisallowUnknownFields` appears.
   - The directive declares no PATTERN changes, and none are required.
   - `PATTERN-stencil-ownership` and `PATTERN-lym-instructions-in-code` are not touched, because the diff changes no stencil. The SKILL.md edit is lym's own instruction in a plugin skill.
   - The commit message carries no `Pattern-Departure:` trailer, and none is needed. Covered.

**Findings:** none.

Tool calls (3 of 6):
1. Repo-wide grep for leftover discussed-status wording outside `_lym`. My context lacked the non-Go files and the untouched packages.
2. Repo-wide grep for `sketch`, `research first` and `researchHeading` outside the changed files. This confirmed no untouched consumer of the old research-only ordering.
3. Test-file grep for the `Discussed` layer or status, plus `contracts/stencils` and `contracts/specs` for `sketch`. My context lacked the untouched tests and stencils.
