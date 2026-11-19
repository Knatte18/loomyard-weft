```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change, I looked for what it forces outside the lines it touches.

**Per change: the set found, covered or not**

- **`ReportReach` and the `reports` vocabulary:** covered.
  - The config check (`gateslot/config.go:538`), `tiers.go`, `gateslot/doc.go`, `template.yaml` and its comment all name both reports.
  - `gatecli/run.go:238` dispatches both.
  - `gateslot/config_test.go` rows pin the template defaults and the refusal wording.
  - Nothing else in non-test Go code switches on report names.
  - `gatecli/doc.go:564` ("runs the redundancy report step last") is still true.
- **New config keys `reach_budget_min`, `mutation_max_mutants` and `mutation_budget_min`:** covered.
  - The `Config` and `configFile` structs, the cross-pair `ConfigSpec` categories, the below-1 bound checks, the template defaults and `gateslot/doc.go` all carry them.
  - `redundancy_budget_min` appears nowhere else outside these files and `PATTERN-test-speed.md`, so no other key list needed updating.
- **`reach_report` rubric marker:** covered.
  - It reaches the stencil, `shedadapters` (`rubric.go`, `bouncer.go`, `doc.go`), `shedrecipe` (`Env`, both entries, `doc.go`), `loomcli` wiring and `doc.go`, and `envkit`'s `nilLegal`.
  - `contracts/stencils/rubric_test.go` has both the allowlist and the code-review-only assertion.
  - Every file that mentions `redundancy_report` was updated for `reach_report`, except `bouncerprompt_test.go` and `rubric_test.go` in `shedadapters`. Those only assert the existing marker, so no new assertion is forced.
- **`GateResult.WriterOnly`, `RerunMark` and `GateEntry.Rerun`:** covered.
  - `shuttleengine` and `gatefixer` have doc and code changes, and `WrapRowGate` carries the mark onto the wrapped entry.
  - `shedrecipe` wires it for the `verify` entry, and the weaver batch entry builds its own turn.
  - `loomcli` sets both `VerifyReach` and `weaverGeom.ReachPending`.
  - `envkit` allows nil for both seams.
- **Failure report's triage lines:** covered.
  - The `renderFailureReport` signature change is internal to `gatecli`.
  - `gateslot.FailedTests` is tested unchanged against triage lines, and `TriageLines` is tested.
  - `loomshed`, `runplan.VerifyGateReport.Triage` (yaml, `omitempty`), `weaverengine` and `websterengine` all carry the lines into findings, each with a doc line and a test.
  - `PATTERN-sole-parsers.md` has no entry for the failure report, so `TriageLines` breaks no sole-parser rule.
- **Verbs `lym gate reach` and `lym gate mutate`:** covered.
  - Both are added in `gatecli/cli.go` under the `gate` subtree, as `PATTERN-cli-cobra` asks.
  - `gatecli/doc.go` has sections for both.
  - Refusals name a way forward, and the tests reach them.
  - `mutationCheck` gained a `site` field, so its slot-holder label and its log prefix are right for the new verb.
- **Declared PATTERN changes:** both land.
  - `PATTERN.md` carries `PATTERN-coverage-map-reader` with the declared text.
  - `PATTERN-test-speed` carries the declared text, character for character.
  - `pattern/PATTERN-coverage-map-reader.md` is new, and `PATTERN-gate-slot.md` and `PATTERN-test-speed.md` gain the refresh and mutate paragraphs.
  - `cmd/lym/coveragemapreader_test.go` enforces the rule.
  - No commit in the range has a `Pattern-Departure:` or `Loosened-Check:` trailer.
- **Stencil wording:** covered.
  - The `Reach report` section in `loom-rubric-code-review.md` matches D5: evidence for coverage and redundancy findings, an unreached function is a candidate, and no finding about its absence.
  - A one-line addition under test economy points at it.
  - No existing rule is dropped.
- **`testmap.go` index write and `mapPackages` deadline:** covered.
  - `gatecli/doc.go` and the integration tests cover the index write, and `testmap_integration_test.go` covers a call whose HEAD moved and a failed write.
  - The `lym gate test --map` doc sentence, the `PATTERN-test-speed.md` sentence and the `gateslot` slot paragraph were all updated.

**Findings:** none. I found no stale caller, doc, fixture or rule document left wrong by the diff's changes.

I did not check `docs/overview.md`, which has 26 changed lines. Its content was not in the pieces I read.

**Tool calls**
1. Grep for `redundancy_report`, `redundancy_budget_min` and `ReportRedundancy` across the tree: my context did not show which other files mention them.
2. Grep for report-name vocabulary in prose and docs, plus the docs that name `lym gate redundant`: to find stale lists of report names.
3. Grep of `PATTERN-sole-parsers.md` for the failure report, plus the stat of `docs`, `contracts/specs` and `pattern`: to check whether the new `TriageLines` parser conflicts with a sole-parser entry.
