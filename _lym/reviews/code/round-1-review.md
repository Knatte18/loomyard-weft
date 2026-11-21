---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/runplan/verifyreport.go#VerifyGateReport
    summary: The change drops the triage key from the persisted verify-gate.yaml record, but the change description calls the door two-way.
    fix: "In _lym/landing/summary.md, under ## Merge danger, change the Door line to one-way and add one sentence saying that runplan.VerifyGateReport no longer writes the optional triage key of verify-gate.yaml, which lym reads back in a later process, and that tolerant decoding keeps old and new records readable both ways. Leave the code unchanged."
    origin: assist:assist
---

### [LOW] The merge door misclassifies a persisted record format change

**Location:** internal/runplan/verifyreport.go#VerifyGateReport

**Issue:** The diff removes the `Triage` field, yaml key `triage`, from `runplan.VerifyGateReport`.
`runplan.WriteVerifyGateReport` persists that record as `verify-gate.yaml` in the run's reports directory,
and `websterengine.readVerifyGateReport` (`internal/websterengine/verifygate.go:526`) reads it back in a later process, which can be a rebuilt binary on reentry.
`contracts/specs/loom-final-summary-spec.md:39` says the door is one-way for a record format another binary reads,
and the rubric's Merge danger bullet makes a `two-way` call on such a change a finding.
`_lym/landing/summary.md` calls the door `two-way`.
The change itself is compatible:
the reader uses plain `yaml.Unmarshal`, so an old record's leftover `triage:` key is ignored, and an older binary decodes a new record with the key absent.
Only the classification is wrong, which is why this is LOW.

## Rejected

- Assistant F1 (MEDIUM, pin the absence of triage annotations in `TestGateTest_Scenario`): no production code can emit a triage line any more, because `TriageLinePrefix`, `addTriage` and the `triage` field are gone and `go vet` is clean on every touched package. An assertion that a deleted feature's output is absent guards against nothing the code can do, and the renderer's exact-output rows in `TestRenderFailureReport` already fail on any extra line.
- Assistant F2's severity: kept as F1 above, lowered from MEDIUM to LOW because the decoding is tolerant both ways and the only fix is the description's wording.
- Lens code-affected-set, LOW on `testreach.Reader.FuncsReachedBy`'s `among` parameter, which now gets only nil from production: the function is still called from `internal/gatecli/reachverb.go:200,221`, is correct, and its doc at `internal/testreach/doc.go:41` describes what it does. Narrowing it would edit the coverage-map reader and the `lym gate reach` verb's callee, which the goal says to leave untouched and invest no more in. A general parameter on a live function is not dead code.

## Review evidence

The range is `4c031f8b..HEAD`, one commit `efefe8c3` with no trailers, and none is needed: this is a feature removal, not a bug fix, and no assertion of a surviving behavior was loosened.

- **Goal coverage:**
  - `internal/gatecli/triage.go` is deleted, and so are `gateslot.TriageLinePrefix`, `TriageLines` and `RenderTriageSection`.
  - Every consumer is gone: the gatecli report budget pass, the loomshed findings section, `runplan.VerifyGateReport.Triage` and its rendering, and the weaver and webster fills.
  - The triage text is removed from all six package docs and from the touched functions' doc comments.
- **Leftover concept:** a repo-wide grep outside `_lym` finds coverage-map triage wording only in `docs/research`, which holds dated measurement records.
  Every other `triage` hit is the flaky-rerun feature (`AppendIntegrationTriage`, `triageWarnings`, `## Integration suite triage`, `webster-verify-triage`), the board's triage of Sketches, or unrelated prose.
- **Kept untouched:** the diff touches no file of the reach note, the reach turn, `lym gate reach` or the mutation step, and no file under `internal/testreach`.
  The helpers that triage shared all keep production callers: `redundancyModule`, `reachModuleOf`, `worktreeMapsDir`, `parentMergeBase`, `changedFuncsAt`, `workingTree`, `impactset.ChangedPaths`, `hubgeom.IsPrime` and `testreach.Open`.
- **Compatibility:** `verify-gate.yaml` is read only by `readVerifyGateReport`, which decodes without `KnownFields`.
  The strict decoders at `internal/runplan/report.go:73`, `internal/weaverengine/inflight.go:45` and `internal/websterengine/outcome.go:49` decode other records.
- **Tests:** each removed test or row pinned triage behavior alone.
  `TestRenderFailureReport`, `TestVerifyFailureFindings`, `TestRenderVerifyGateFindings` and `TestVerifyGate_FailedEvaluationWritesReport` keep their other assertions unchanged.
- **Build:** `go vet` passes on `internal/gatecli`, `internal/gateslot`, `internal/loomshed`, `internal/runplan`, `internal/weaverengine` and `internal/websterengine`.
- **PATTERN:** no PATTERN changes are declared for this round, and no PATTERN entry or `pattern/` background file describes triage lines.
