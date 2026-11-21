# Remove the coverage-map triage lines from failure reports

Failure reports no longer carry `triage:` lines under a failed test.
Those lines said whether the test reaches a function the run changed, read from the coverage maps.
The measurement in `docs/research/speed/testmap-use-measure-2026-11-21/report.md` found the "reaches none of this run's changes" line wrong in all 6 cases it showed.
Each failed test failed because of the run's own change, reached through a tree text scan, an embedded file or a nil seam the maps cannot see.
No fixer used a triage line.

## What changes

- `internal/gatecli`: `triage.go` and its tests are deleted.
  `renderFailureReport` no longer budgets triage lines, `runFailure` loses its `triage` field and `isTest`, and `printReport` no longer calls `addTriage`.
- `internal/gateslot`: `TriageLinePrefix` and `TriageLines` are deleted.
- `internal/websterengine`, `internal/weaverengine` and `internal/runplan`: verify-gate reports no longer set or render a `Triage` field.
- `internal/loomshed`: the verify gate no longer passes triage lines on.
- Package docs that described triage lines lose that text.

## Design decisions

- The reach note, the `lym gate reach` command and the mutation step are untouched.
  The operator chose to keep them and to invest no more in the maps.
- The report keeps its line budget and cut rules; only the triage lines that counted against it are gone.

## Check by hand

Nothing was left unrun.
A reviewer can grep the tree for `TriageLine` to confirm no consumer remains.
Other `triage` matches, such as in `internal/websterengine/triage.go`, belong to the integration-suite triage and are unchanged.

## Merge danger

**Door:** two-way
**Blast radius:** gatecli
