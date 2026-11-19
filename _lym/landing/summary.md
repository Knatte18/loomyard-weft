# Commit a docs darn's report and flag an empty landing

A darn run on the `docs` profile could land with an empty diff.
`comment-trim-second-pass` wrote its measurements only into the change description, which stays in the run's records and never reaches the parent branch.
Land then landed nothing, and dobby reported the parent's unchanged tip as "landed as 64e58795".

Two changes close this.

## Report committed under `docs/research`

- `darn.yaml` gains a `report_dir` key: a worktree-relative directory the writer commits the task's report under.
  An empty value asks for no report file, which is the default.
- The `docs` run profile sets `report_dir: "docs/research"`.
- When `report_dir` is set, `darnengine.DarnTable` appends a new stencil passage, `darn-fragment-report.md`, to the writer prompt's Step 4 list.
  The passage tells the writer to commit the report at the path the task names under `report_dir`, or else at `<report_dir>/<slug>.md`.
- `LoadConfig` refuses a `report_dir` that is not worktree-relative, and one with a records directory (`_lym`, `.lym`) in any segment,
  because a report there would never reach the parent branch.

## Empty landing flagged in dobby's line

- `landingshed.Finalize` now reads whether the landing changed any code path against the parent head merged into the task worktree,
  through a new `Deps.ChangedCodePaths` seam backed by `fabricengine.Fabric.ChangedCodePaths`.
  The seam drops records paths, so the parent pair's own records commit does not hide an empty landing.
  A failed path read records the landing as not empty and logs a warning.
- `shedrun`'s landed-commit file keeps the flag as a second line, `empty-diff`, next to the commit.
  `ReadLandedCommit` and `WriteLandedCommit` now carry a `LandedCommit{Commit, EmptyDiff}`.
  A file holding the commit alone reads as not empty, so records written before this change stay valid.
- `RecordLandedCommit` keeps the first landing's flag when the same commit is recorded again,
  so a landing step re-run after a failed push does not reclassify it.
- Dobby's landing notice reads "landed with an empty diff: no file of its parent branch changed, which stays at `<commit>`"
  in place of naming the parent's tip as the landing.

## Design decisions

- Land is flagged, not refused: an empty landing is sometimes legitimate, and the flag puts it where the operator reads.
- "Empty" means no code path changed.
  Judging by all paths would let the records commit the parent pair adds count as work.

## To check by hand

No regression check ran, and none of this was run against a live hub.

- Run a `docs` darn end to end: the report lands under `docs/research/`, and dobby's line names the landing commit.
- Land a run that changes no code path: dobby's line shows the empty-diff wording and the parent's tip.
- Confirm `landingshed`'s empty-landing test runs in parallel like its neighbours in `internal/landingshed/finalize_test.go`.
- Confirm a landing whose only parent-side change is the parent pair's records commit is judged empty, in `landedDiffEmpty` in `internal/landingshed/finalize.go`.
- Confirm a landing step re-run after a failed push keeps the first landing's empty-diff flag, in `RecordLandedCommit` in `internal/shedrun/landed.go`.
