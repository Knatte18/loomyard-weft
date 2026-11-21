```yaml
lens: code-affected-set
outcome: reported
```

I reviewed the code-affected-set lens: everything outside the changed lines that calls, reads, asserts or describes what the diff removed or changed.

**Per change, the set found, and whether it is covered**

1. **`gateslot.TriageLinePrefix`, `TriageLines`, `RenderTriageSection` and `reportTestIdentity` removed.** Covered.
   - Callers were `gatecli` (`failurereport.go`, `triage.go`, tests), `loomshed/verifygate.go`, `runplan/verifyreport.go`, `weaverengine/strand.go` and `websterengine/verifygate.go`. All are updated.
   - `go vet` is clean on all six packages.
   - A repo-wide grep finds no remaining reference outside `docs/research`.
   - The `gateslot` package doc lost its text.
   - `pattern/PATTERN-gate-slot.md:115` mentions the failure report but not triage lines, so it is still accurate.
2. **`runplan.VerifyGateReport.Triage` field (yaml `triage`) removed.** Covered.
   - The persisted `verify-gate.yaml` is read only by `websterengine.readVerifyGateReport` (`internal/websterengine/verifygate.go:526`), which uses plain `yaml.Unmarshal`.
   - A record from before this change that still holds a `triage:` key therefore decodes fine, as PATTERN-tolerant-records requires.
   - No `KnownFields` decoder reads this record: the strict decoders at `runplan/report.go:73`, `weaverengine/inflight.go:45` and `websterengine/outcome.go:49` decode other records.
   - The only writer is `runplan.WriteVerifyGateReport`.
   - `webster-spec.md:94-96` names `verify-gate.yaml` but not its triage contents, so no spec edit is needed.
3. **`RenderVerifyGateFindings` / loomshed `verifyFailureFindings` no longer print a `Triage:` section.** Covered.
   - Grepping `contracts/`, `plugins/` and `docs/` (research excluded) finds no stencil, spec or skill that tells a fixer to read a `Triage:` section.
   - The remaining `triage` hits are the flaky-rerun feature: `AppendIntegrationTriage`, `## Integration suite triage`, `webster-verify-triage` and `landing-template-describe.md:35`. That is a different concept and correctly stays.
4. **`gatecli` failure report: triage lines, `runFailure.triage`/`isTest`, the `reportLines(triageKept)` signature, `triageFailuresKept`, `execution.addTriage`, `gatecli/triage.go`.** Covered.
   - The only caller of `reportLines` is `renderFailureReport`, which is updated.
   - The `gatecli` doc's failure-report paragraph and the `renderFailureReport`/`printReport` doc comments lost their triage text.
   - The helpers triage shared (`redundancyModule`, `reachModuleOf`, `worktreeMapsDir`, `parentMergeBase`, `changedFuncsAt`, `workingTree`, `impactset.ChangedPaths`, `hubgeom.IsPrime`, `testreach.Open`, `goModule.relativePackage`) all keep production callers in `reach.go`, `reachverb.go`, `mutate.go`, `mutation.go`, `scope.go`, `redundancy.go` and `redfirst.go`.
5. **`testreach.Reader.FuncsReachedBy`, a callee of the removed `triageLookup.lines`.** Partly covered; this is the one finding.
   - The function stays reachable through `gatecli/reachverb.go:200,221`.
   - Both of those calls pass `among = nil` and discard the returned slice.
6. **PATTERN.md / `pattern/`.**
   - No entry or background file describes triage lines.
   - No PATTERN changes are declared for this round, so nothing needs to land.
   - PATTERN-coverage-map-reader is unaffected: the diff removes a reader and adds none.
7. **Commit message and `summary.md` claims.**
   - The commit carries no `Pattern-Departure`, `Loosened-Check` or `Root-Cause` trailer, and none is needed: this is a feature removal, not a bug fix, and the removed assertions belong to tests of the removed feature.
   - The summary says no stencil, spec or PATTERN entry described triage lines; the greps above confirm it.
   - The summary says the shared helpers have other callers; item 4 confirms it.

**Findings**

- **LOW, design, `internal/testreach/reader.go:186` (`FuncsReachedBy`).** The diff removed the only production caller that passed a non-nil `among` list, which was `triageLookup.lines` in the deleted `internal/gatecli/triage.go`.
  - The function's main contract, returning which of `among` the test reaches, is now exercised only by `internal/testreach/reader_test.go:232-263`.
  - Its remaining production callers (`internal/gatecli/reachverb.go:200` and `:221`) pass `nil` and use only `held` and `source`.
  - `internal/testreach/doc.go:41` still describes it as answering "which of a list of functions one named test reaches".
  - Fix: narrow it to the lookup reachverb needs (whether a map holds the test, and that map's source), drop the `among` parameter and the reached-subset return, and update `doc.go:41` and the `reader_test.go` row to match.

**Tool calls**

1. Grepped PATTERN.md, `pattern/`, the `testreach`/`gatecli` docs and `contracts`/`docs` for triage, `FuncsReachedBy` and `verify-gate.yaml`, to check the rule documents and specs the manifest had not covered.
2. Read `webster-spec.md:84-100`, the `pattern/PATTERN-coverage-map-reader.md` report lines, and stencil/skill hits for `Triage:` or "triage lines", to check the spec's description of the record and whether any fixer stencil depends on the removed section.
3. Grepped the `runplan` and `weaverengine` functions that handle `VerifyGateReport`, to confirm no strict decoder reads the record whose `triage` key was dropped.
