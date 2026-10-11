---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/fabricengine/junction.go:572
    summary: unwire dry run still takes the exclude-file write lock and creates the info dir through mutateGitExclude, so a bare run is not strictly write-free
---

### [LOW] Unwire dry run takes the exclude lock and may create files

**Location:** `internal/fabricengine/junction.go:572` (`unseedGitExclude` with `dryRun`), through `internal/fabricengine/gitexclude.go:52` (`mutateGitExclude`).
**Issue:** The dry run answers the exclude question by calling `mutateGitExclude` and returning the original content from the rewrite closure.
`mutateGitExclude` runs `os.MkdirAll` on the exclude directory and acquires the write lock on a lock file there before it calls the closure.
Nothing in the exclude file changes, but a bare `lyx fabric unwire` still creates the info directory and the lock file when they are absent, and it blocks behind a concurrent exclude writer.
The card's bound says the dry run "reads state only", and the verb's help says it "changes nothing".
`lyx board prune` and `lyx board retire-legacy` have the same shape through `boardCriticalSection`, which runs `os.MkdirAll` on the board directory and takes the board lock before the `noWrite` return.
**Fix:** Optional.
Either state in the `unseedGitExclude` and `Board.Prune` doc comments that a dry run takes the lock and may create the directory and lock file, or read the exclude file without `mutateGitExclude` on the dry path.

## Verdicts on the focus questions

- Card 2: `git mv` shows as a pure rename with no content change in the commit stat.
  Only surgical edits follow, and a grep for `final-summary-spec` finds no stale old-path reference in code, docs, stencils or the embed and registry lists.
  Not a defect.
- Cards 6 and 7: every mutation is behind `apply`.
  Unwire's dry run skips `UnwireJunctions` and calls `unseedGitExclude` with `dryRun` true, which returns the original content so `mutateGitExclude` writes nothing.
  `Board.Prune` and `Board.RetireLegacy` return `noWrite` when `apply` is false, which skips the save, the retire-legacy callback and the render.
  A bare run prints the applied run's fields plus `"applied": false`.
  Only F1 applies.
- Card 4: `ClearTerminalOpened` is called only in `battenPreRun` and `battenPreStepUnmarked`, after the found-status early return.
  The clearing block in `innerRunProducer.Call` is deleted, so a re-spawn and a resume keep the marker.
  Not a defect.
- Card 5: `RaiseIntFloor` leaves absent keys, env markers, non-integers and at-or-above values untouched, raises below-floor and null values, and reports the rewrite.
  The three hooks wrap their parse error with `configengine.MarkInvalid` and are registered on the loom, orch and shuttle rows.
  Not a defect.
- Card 9: `replaceFile` compares the on-disk bytes to `prior` before the rename and to `data` after it.
  The post-rename compare cannot misfire on the writer's own write, since `data` is exactly what was renamed into place.
  The refusal-spec row is present and the test reaches both paths.
  Not a defect.
- Card 8: `withNote` prints a note only under the audience its annotation names.
  The plan changes the unannotated case to never print, which `indexNoteProblems` now guards in the live tree; the focus line saying unannotated notes keep their prior behavior disagrees with the plan, and the plan governs.
  The board group is the only noted command and now names the operator audience.
  Not a defect.
- Test economy and budget: the new top-level tests are `TestRaiseIntFloor` (card 5) and `TestWriteCarryOver_DetectsConcurrentWriter` (card 9), and `test-budget.yaml` raises `internal/yamlengine` and `internal/discussionparser` by one each.
  Every other card extends an existing test or table.
- Docs in the same commit: `doc.go` files for `yamlengine`, `shuttleengine`, `orchengine`, `configcli`, `fabricengine`, `boardcli`, `battenshed` and `clihelp`, `docs/overview.md`, the sandbox suites, the board skill and `refusal-spec.md` are updated where the cards require.
- Verification run: `lyx gate test` over the touched packages, `./cmd/lyx` and the `integration` tier for `fabriccli` and `configcli` all pass.
