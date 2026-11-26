```yaml
lens: code-affected-set
outcome: reported
```

I traced everything outside the diff's lines that reads or describes what it changes. The two that need a fix: the `AmendedCards` docs in `internal/runplan/state.go`, and the Master waiting rung, which has no instruction for an answer.

## Affected-set list

**1. `runplan.RecordedOverviewFrame` (moved here from websterengine, cause constants renamed)**
- Callers: `beginbatch.go` `fingerprintMismatchWayForward`, `rebaseline.go` `Rebaseline` and loomshed `overviewFrameEditedFindings`. All covered.
- No Go or contract file still names the old `recordedOverviewFrame` or the old `cause…` constants.
- `docs/research/burler/lens-fan/lens-fan-lab-phase0.md:80` still names `recordedOverviewFrame` in `rebaseline.go`. It is a lab record of a past run, so it is exempt.

**2. `goPackageDir` exported as `GoPackageDir`**
- Every call site is renamed, and so are the docs in `planparser/doc.go` and the `RefScope` doc.
- The remaining `goPackageDirs` hits in loomshed are a different function.
- Covered.

**3. `overviewFrameEditedFindings` (new runDir parameter, unframed branch)**
- Updated: `ValidatePlan`'s doc, the `loom-plan-checks-spec.md` entry, the test that calls the function directly, and the `TestNewPlanGate` rows.
- The new unframed detail has the same shape as the existing frame-edited detail (`gates.go:345`). Covered.
- The `loom-rubric-plan-review.md` line added by the diff says "a fix edits only its Card Index". That is not true for a run with no recorded frame and no baseline copy: there the gate also refuses a Card Index edit (`unframedOverviewFindings`). Finding F3.

**4. `AmendedCards` now recorded on terminal failed, dead and stuck batches**
- Readers: `rendersNewAmendment` in `recoverbatch.go:237`, the cap refusal at `:520`, `recoveryRetryRead:448`, `beginbatch.go:226`, `recordbatch.go:177`, `runlevel.go:1191/1193`, `batchfail.go` `amendedReasons`, `failureDigestBlock` and `recoverSpawn`. All of them already treated an unrendered amendment as exempting the batch, so they are consistent.
- Docs updated: `websterengine/doc.go:105-108` and the `Rebaseline` doc.
- Master rung `:155` names the rebaseline route. Covered.
- Not updated: the field and type docs in `internal/runplan/state.go:101` and `:123`. Finding F1.

**5. Recovery turn end with a message becomes `waiting`**
- Updated: the `awaitTerminal` doc, `waiting.go` header, `websterengine/doc.go:188-189`, `recover-batch` Long help and Master rung `:138`.
- The webstercli terminal path at `recoverbatch.go:358` already exits 0 for `waiting`.
- The Master rung's resume clause (`webster-template-master.md:140-142`) is still worded only for a step taken or refused. It gives Master no way to pass the parent's answer to the resumed recovery. Finding F2.

**6. Glyph-not-found detail and the created-member pass**
- Updated: the `planglyph/doc.go` bullets, the `resolvePass` doc, the plan-checks spec lines 97-102 and the status-enforcement allowlist.
- Other `statusFindings` callers: `drift.go:271` passes nil, which is intended because it judges only repaired glyphs.
- Covered.

**7. `gatefixer.LastTurnMessage`**
- The `strand.go` file header and `strand_test.go` header are updated.
- `gatefixer/doc.go` names none of the strand readers, so it has nothing to update.
- Covered.

**8. Commit messages and trailers**
- Root-Cause trailers are on `78614b6` and `f1b1705`, and each names the defect its diff removes.
- `f219614`'s Loosened-Check names the allowlist additions and the tripwire report that caused them. It only adds allowlist entries and removes the stale `statusFindings` one, so it holds.
- `dfaec51` changes an expected value to match intended new behaviour, and its message explains it. No Loosened-Check is needed.
- No Pattern-Departure trailers. No declared PATTERN changes to check.
- PATTERN-refusal-way-forward: no new refusal message in a bound verb lacks a way forward. The new gate detail follows the existing finding's form.

## Findings

- **F1** — LOW, consistency, `internal/runplan/state.go:101,123`. The `AmendedCard` doc says "one card of an in-flight batch" and the `AmendedCards` doc says "accepted an edit to while the batch was in flight". Both are stale now that `Rebaseline` (`rebaseline.go:220-225, 371-378`) also records amendments on terminal failed, dead and stuck batches.
- **F2** — MEDIUM, consistency (stencil wording), `contracts/stencils/webster/webster-template-master.md:140-142`. The waiting rung now covers a recovery that "ended its turn on a question". But the resume step only reacts to "the step is taken" or "will not be taken". It says nothing about passing the parent's answer to the resumed `recover-batch`, and `recoverSpawn` (`recoverbatch.go:317`) gives that call no input for one.
- **F3** — LOW, consistency (stencil wording), `contracts/stencils/loom/loom-rubric-plan-review.md:31` (the added line). It tells a fixer that a Card Index edit is allowed once a batch has begun. On a run with no recorded frame and no baseline copy, the gate refuses that edit too (`internal/loomshed/gates.go` `unframedOverviewFindings`; spec `loom-plan-checks-spec.md:298`).

## Tool calls
1. Searched specs and code for the old `goPackageDir`, `recordedOverviewFrame` and cause-constant names, amendment wording, asking wording and `overview-frame-edited` readers; the manifest had no repo-wide sweep.
2. Searched specs, docs and package docs for `recovery_retry`, `AmendedCards`, `NEEDS_PARENT`, waiting and `glyph-not-found`, to find descriptions the diff did not touch.
3. Read `websterengine/doc.go`'s recovery and waiting sections and the Master stencil's rebaseline lines, which sit outside the diff hunks.
4. Read `pattern/PATTERN-refusal-way-forward.md`; the manifest did not include it.
5. Read the existing frame-edited finding detail and the gate tests' way-forward assertions, to compare the new detail's form.
6. Searched `gatefixer/doc.go` for the strand readers, to see whether the new export forces a package-doc change.
