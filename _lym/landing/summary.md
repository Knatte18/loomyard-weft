# Commit a docs darn's report, and flag an empty landing in dobby's line

A darn run on the `docs` profile could find something, write it only into its change description, commit nothing, and still "land".
The change description stays in the run's records, so the finding never reached the parent branch.
Dobby then reported the parent's unchanged tip as the landing commit.
This change closes both gaps.

## A docs run commits its report

- `darn.yaml` gains `report_dir`, a worktree-relative directory; empty, the template's default, asks for no report file.
  `LoadConfig` refuses a value that leaves the worktree, and keeps the value in clean slash form.
- When `report_dir` is set, `DarnTable` appends the new stencil `darn-fragment-report` after the writer prompt's Step 4 list.
  The passage has the writer commit the task's report under that directory, at the path the task names there, or else at `<report_dir>/<slug>.md`.
  The change description then summarizes the report and names its path.
  The marker shares `make_change`'s line, so a run without `report_dir` gets a prompt with the same bytes as before.
- The `docs` run profile's darn block sets `report_dir: "docs/research"`.
  A hub's `profiles.yaml` takes the new key at its next reconcile, where its docs block still holds lym's earlier value.

## An empty landing is flagged

- After the parent-side merge, Finalize reads the paths that differ between the parent head it merged into the task worktree and the landing commit, through the existing `Deps.ChangedPaths`.
  `Deps.RecordLanding` now takes whether that list is empty, and Finalize logs such a landing at Warn.
  With no merged head held, as in a Finalize resumed past the merge, or when the path read fails, the landing is recorded as not empty.
- The prime's landed-commit file (`shedrun.LandedCommitFile`) gets a second line, `empty-diff`, for such a landing.
  `ReadLandedCommit` now returns a `LandedCommit` struct holding the commit and the flag.
  A file written before this change holds the commit alone and reads as not empty.
- `battencli` carries the flag into `dobbyengine.RunView.EmptyLanding`.
  Dobby's landing notice then reads `[dobby notice] <slug>: landed with an empty diff: no file of its parent branch changed, which stays at <commit>`, where it used to say `landed as <commit>`.
  A beam member's follow-up still ends the line.

## Design decisions

- **Flag, not refuse.** The entry offered either.
  A refusal at Land would stop a run after its review had already passed, and would need a refusal way forward of its own.
  With docs runs now committing a report file, an empty landing is the rare case, and naming it in the notice the orch reads is enough to catch it.
- **The report path is configuration, not a hard-coded docs rule.** `report_dir` is an ordinary darn key, so a hub can point it elsewhere or set it in another profile.
  A task that names its own path under the directory, such as the retro task's `docs/research/retro/...` report, keeps that path.
- **Emptiness is the landed diff's path list**, not the landing commit being equal to the parent's earlier tip.
  It catches a landing that made a merge commit with no change as well as one that moved nothing.
  Merges in a one-repository hub restore the receiving side's records paths, so a run's records alone never make a landing count as non-empty.

## Tests

- `TestFinalize_RecordsWhetherTheLandedDiffIsEmpty` (`internal/landingshed`) pins the flag for each case: no path changed, a path changed, a failed path read, and no merged head held.
- `TestLandNotices` (`internal/dobbyengine`) gains the empty-landing line.
- `TestLandedCommit` (`internal/shedrun`) round-trips the flag and reads a commit-only file as not empty.
  `TestLandingDeps_RecordLandingWritesWhereTheDobbyReads` (`internal/loomcli`) checks the seam writes the flag where dobby reads it,
  and the `DobbyRuns` test in `internal/beamcli` checks it reaches the run view.
- `TestDarnSpec` (`internal/darnengine`) gains a row for the report passage and its default path, and checks the passage is absent without `report_dir`.
  `TestLoadConfig` covers a clean-form value and the refusal of one outside the worktree.
  `TestTemplateDocsProfile` (`internal/runprofile`) pins the docs block's new key.
