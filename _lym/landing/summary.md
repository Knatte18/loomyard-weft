# Remove the coverage-map triage lines from failure reports

The measurement in `docs/research/speed/testmap-use-measure-2026-11-21/report.md` found the triage line "reaches none of this run's changes" wrong in every case it showed.
Each failed test had failed because of the run's own change, reached through a tree text scan, an embedded file or a nil seam that the coverage maps cannot see.
No fixer used a triage line.
This change removes the triage lines and everything that reads or renders them.

## What changes

- `lym gate test`'s failure report no longer prints `triage: ` lines under a failed test.
  `internal/gatecli/triage.go` is gone, along with the `triage` field and `isTest` method of `runFailure` and the budget pass in `renderFailureReport` that kept triage lines whole or dropped them.
- `internal/gateslot` drops `TriageLinePrefix`, `TriageLines` and `RenderTriageSection`.
- `runplan.VerifyGateReport` drops its `Triage` field, and `RenderVerifyGateFindings` no longer renders a `Triage:` section.
  Webster's verify gate and weaver's batch and plan-level verify no longer fill the field.
- loomshed's verify findings no longer add a `Triage:` section.
- The package docs of `gatecli`, `gateslot`, `runplan`, `weaverengine`, `websterengine` and `loomshed` lose their triage-line text.

The tests that pinned triage lines are removed with them: `gatecli`'s triage unit tests, the report-rendering row and cap subtest, the two integration subtests' triage assertions (one subtest was only about triage, so it is removed whole), `gateslot`'s `TestTriageLines` and its triage row in `TestFailedTests`, `weaverengine`'s `TestFailureReport` and its export, and the Triage checks in webster's and loomshed's verify-gate tests.

## What stays

The reach note, the reach turn, `lym gate reach` and the mutation step are untouched.
The helpers that triage shared with them, such as `redundancyModule`, `changedFuncsAt`, `parentMergeBase` and `testreach.Reader.FuncsReachedBy`, have other callers and stay.
Webster's and weaver's `## Integration suite triage` summary section and the `webster-verify-triage` friction note are about flaky reruns, not coverage-map triage lines, and stay.

## Design decisions

- An existing `verify-gate.yaml` that still holds a `triage:` key reads fine: yaml decoding ignores the unknown key, as the tolerant-records pattern requires.
- No stencil, spec or `PATTERN.md` entry described triage lines, so none changes.

## Merge danger

**Door:** two-way
**Blast radius:** gate
