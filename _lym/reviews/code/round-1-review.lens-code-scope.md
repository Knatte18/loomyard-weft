```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. I mapped each change in the diff to the goal point it builds, and each goal point to the changes that build it.

**Decisions** (decision record alone, from the three goal points under `## Goal`; the record has no `Added after Discussion` entries):
- **G1:** remove the triage lines from failure reports, meaning `internal/gatecli/triage.go` and every caller and consumer (webster, loomshed and runplan verify reports, and their stencils and specs).
- **G2:** leave the reach note, the reach turn and the mutation step untouched.
- **G3:** docs and specs that describe triage lines lose that text.

**Per change:**
- **C1** (traces to G1): `internal/gatecli/triage.go` deleted, the `addTriage` call removed from `printReport` (`run.go`), the `triage` field and `isTest` removed from `runFailure` (`counters.go`), and `triageFailuresKept` plus the `reportLines(triageKept)` parameter removed (`failurereport.go`). These are the line producer and its only wiring.
- **C2** (traces to G1): `internal/gateslot` drops `TriageLinePrefix` (`runlog.go`) and `triage.go` (`TriageLines`, `reportTestIdentity`, `RenderTriageSection`). These are the shared reader and renderer the consumers used.
- **C3** (traces to G1, loomshed consumer): `verifyFailureFindings` no longer appends the Triage section (`internal/loomshed/verifygate.go`).
- **C4** (traces to G1, runplan consumer): `VerifyGateReport.Triage` removed, and both `RenderTriageSection` calls are gone from `RenderVerifyGateFindings` (`internal/runplan/verifyreport.go`).
- **C5** (traces to G1, webster consumer): both `Triage:` fills are gone, in `failHung` and in the rerun report (`internal/websterengine/verifygate.go`).
- **C6** (traces to G1): `failureReport` no longer fills `Triage` (`internal/weaverengine/strand.go`). Weaver is not named in the entry, but it is a consumer of the same record field, so "every caller and consumer" covers it.
- **C7** (traces to G3): the triage text is removed from the `doc.go` files of `gatecli`, `gateslot`, `loomshed`, `runplan`, `weaverengine` and `websterengine`. Doc comments are also trimmed in `failurereport.go` `renderFailureReport`, `run.go` `printReport`, `loomshed` `verifyFailureFindings`, `runplan` `RenderVerifyGateFindings` and `weaverengine` `failureReport`.
- **C8** (forced by C1 to C6): the tests that pinned triage are removed or trimmed:
  - `gatecli/triage_test.go`
  - the `failurereport_test.go` row and cap subtest
  - the `test_integration_test.go` prime `go.mod` setup, the triage assertion and the mapped-test subtest
  - the `gateslot/failedtests_test.go` row and `TestTriageLines`
  - the `loomshed/verifygate_test.go` row and the `Triage:` wantNot guard
  - `weaverengine/failurereport_test.go` and its `export_test.go` `FailureReport`
  - the `websterengine/verifygate_test.go` `triagedPackageLog` and its block
  - the `websterengine/verifygatereport_test.go` Triage checks

  None of these is unrequested scope, since each pinned only behaviour that C1 to C6 remove.

**Per decision:**
- **G1:** built by C1 to C6. The entry also names "their stencils and specs". My repo-wide grep found no stencil or spec under `contracts/` that describes coverage-map triage lines. The remaining hits are about flaky-rerun triage (`contracts/specs/webster-spec.md:71,90`, `contracts/stencils/landing/landing-template-describe.md:35`), a different feature, so there was nothing for the diff to change.
- **G2:** built by the diff leaving them alone. It touches no reach or mutation file: no `reach.go`, `reachverb.go`, `reachreport.go`, `mutation.go` or `mutate.go` hunk, and no `internal/testreach` hunk. The helpers `triage.go` shared (`redundancyModule`, `reachModuleOf`, `worktreeMapsDir`, `parentMergeBase`, `changedFuncsAt`, `hubgeom.IsPrime`, `impactset.ChangedPaths`) still have production callers in `reach.go`, `reachverb.go`, `mutate.go` and `scope.go`, per the manifest grep.
- **G3:** built by C7. The grep found no other doc outside `docs/research` describing triage lines. `docs/research` is a measurement record, which the prose rule exempts.

**Findings:** none.

Out of my lens, for the handler: `testreach.Reader.FuncsReachedBy`'s `among` parameter now gets only `nil` from production (`internal/gatecli/reachverb.go:200,221`). The removed `triage.go` was its only caller that passed a function list, so it is a candidate for leftover dead surface. Shrinking it would touch the reach code that G2 says to leave alone.

Tool calls: none.
