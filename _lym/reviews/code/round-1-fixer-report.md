---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The proposed records-path filter inside landedDiffEmpty needs a records-path source the landing package has no seam for. Not made: a filter in landingshed over a faked ChangedPaths. Made instead: Fabric.ChangedCodePaths drops every path under RecordsPathsAt(to), wired as a new Deps.ChangedCodePaths seam, with the records row tested in internal/fabricengine/parenttips_integration_test.go instead of in TestFinalize_RecordsWhetherTheLandedDiffIsEmpty. The base was not moved."
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "The proposed skip of the flag computation in recordLanding needs Finalize to know the first pass's record. Not made. Made instead: shedrun.RecordLandedCommit keeps the recorded EmptyDiff when the same commit is recorded again, and the loomcli RecordLanding seam calls it. The re-step case is pinned in internal/shedrun/landed_test.go, since the landingshed test only sees a fake RecordLanding."
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
---

# Round 1 fixer-report

## Changes

Commits, oldest first:

- `496cd82b` F3: `shedrun.RecordLandedCommit`, `TestRecordLandedCommit`, shedrun package doc.
- `58a43111` F1 and F2: `Fabric.ChangedCodePaths` and its test, `Deps.ChangedCodePaths`, `landedDiffEmpty`, the `landingshed` package doc, the loomcli wiring, and the two seam-list tests.
  F2 shares the commit because `finalize_test.go` holds both changes.
  The loomcli `RecordLanding` hunk of F3 is in this commit too, as it shares `landingdeps.go` with the F1 wiring.
- `6e1f7650` F4: a `TestLandNotices` row for a beam member whose run landed empty.
- `4dd37e72` F5: `LoadConfig` refuses a `report_dir` under a records directory, with a `TestLoadConfig` row.
- `6536f09a` F6 and F7: `TestDarnSpec` asserts the filled report values after the Step 4 heading, and `TestDarnSpec_Refusals` gets the report-fragment row.
  Both share `spec_test.go`.
- `bb0c0ae7` applies the self-check findings below.
- `08ff4c8c` repairs the gate's line-length findings: it splits the `LoadConfig` doc line in `internal/darnengine/config.go` and the `WriteLandedCommit` line in `internal/shedrun/doc.go` into sentences of their own.

F1 details: `ChangedCodePaths` judges emptiness over code paths only.
Step 5b is untouched, so a pull-request landing's base stays before the GitHub merge.
The doc comments and package docs now say "no code path".

- The final commit repairs the verify gate's `envkit` failure: `TestFullEnv_FillsEverySeam` found the new `Deps.ChangedCodePaths` seam unfilled, so `internal/testkit/envkit/envkit.go` now fills it beside `ChangedPaths`.
- A later verify run failed `TestGateMutate/APackageWithAStaleWorktreeMapAndCommittedFilesIsRefreshedThenJudged` in `internal/gatecli`.
  Its refusal reads "not refreshed ... run the gate again", and it ran beside many other test binaries while `/tmp/.git` appeared.
  The test is outside the coverage maps and touches none of this round's code.
  `lym gate test --tier 1 ./internal/gatecli` alone passed, so no change was made for it.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW: `Deps.RecordLanding` comment still said "no path". Fixed, with its siblings in `shedrun`, `dobbyengine` and the Warn log.
- fix-consistency, LOW: sibling descriptions `LandedCommit.EmptyDiff`, `EmptyLanding`, `landNoticeLine` and the shedrun package doc said "no path". Fixed. The dobby line's wording is user-facing and matches its test, so it is left.
- fix-proposal, MEDIUM: F5's check missed a subpath anchor's `<anchor>/_lym`. Fixed differently from the review's proposal: any path segment equal to a records directory name is refused, with a `backend/_lym/research` row.
- fix-proposal, LOW: `wantAfterStep4` held a value that is a substring of the other. Fixed: the `docs/research/` entry is dropped; the path asserts the same position.
- fix-proposal, LOW: no `landingshed` row feeds a records-only landing. Not changed: the filtering sits behind the seam and is pinned by `TestChangedCodePaths_LeavesOutTheRecordsPaths`; the existing rows fail if `landedDiffEmpty` reads `ChangedPaths`.
- fix-affected-set, LOW: `internal/darnengine/doc.go` still described the old refusal. Fixed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
