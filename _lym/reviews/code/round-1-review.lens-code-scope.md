```yaml
lens: code-scope
outcome: reported
```

**code-scope list**

Task: trace each change in the diff to the goal points of the decision record. The record has no `Added after Discussion` entries.

Decisions (the goal points under the record's `## Goal`):
- G0: build the `mutation` gate report step and list it in the template's `diff` keys.
- G1: spread the mutants across the changed functions, one per function first, under `mutation_max_mutants`.
- G2: skip as a mutation site a statement that only logs, and a branch that tests `runtime.GOOS` for another platform.
- G3: write the full list to `mutation.json` and print one summary line: mutants run, killed, survived, unjudged, uncovered changed blocks, packages not judged.
- G4: the step feeds a `mutation_report` marker of the Code-Review rubric with the survivors.

Changes and what they trace to:
- C1, the log-only and other-platform noise sites: `internal/testcover/mutant.go` (`logsOnly`, `blockLogsOnly`, `comparesOtherPlatform`, the `noise` field), `linemutant.go` (`mutatesBehavior`), `testcover/doc.go`, and the rows of `linemutant_test.go`. Traces to G2.
- C2, the even spread: `spreadEvenly` and `selectWithinCap` in `internal/gatecli/mutatecheck.go`, `mutatecheck_test.go`, and the cap line in `gatecli/doc.go`. Traces to G1.
- C3, the new step: `internal/gatecli/mutation.go`, the `runsMutation` wiring in `run.go`, `mutationreport.go` (`MutationReport`, `SummaryLine`, `ReadMutationReport`), `mutationreport_test.go`, the new `TestGateMutate` row, and the Mutation report section of `gatecli/doc.go`. Traces to G0 and G3.
- C4, the shared `mutationCall` that `mutate.go` now builds for both callers. Forced by C3, since the step runs the verb's refresh and check.
- C5, the exported result types (`mutateCounts` becomes `MutationCounts`, and the rest likewise) and their uses in `mutate_integration_test.go`. Forced by C3 and C6, which carry those types in the report and the note.
- C6, the extraction of `changedFunctionsMatch` from `ReachReportMatches` in `reachreport.go`. Forced by `MutationReportMatches`, which G4 needs so it reads only a report that matches HEAD's tree.
- C7, gateslot config: `ReportMutation` in `tiers.go`, the `decodeGates` acceptance in `config.go`, the four `diff` keys and the comments in `template.yaml`, `gateslot/doc.go`, and `config_test.go`. Traces to G0. Its other edits are forced:
  - the comments of the budget keys;
  - the replacer string in `landingdeps_verify_test.go`;
  - the empty `reports` list on the `pr-review` key in `scopes_integration_test.go`.
- C8, the marker: the `## Mutation report` section of `loom-rubric-code-review.md`, `loomcli/mutationnote.go` and its test, `wiring.go` and `wiring_test.go`, `shedrecipe` (`Env.MutationReport`, `entries_bouncer.go`, `entries_burler.go`, `doc.go` and their tests), `shedadapters` (`rubric.go`, `bouncer.go`, `doc.go`), `envkit` `nilLegal`, and the allowlist in `rubric_test.go`. Traces to G4.
- C9, documentation: `docs/overview.md`, `PATTERN-test-speed.md`, `PATTERN-coverage-map-reader.md`, and the `lym gate mutate` help text and file header in `mutate.go`. Forced by C1–C3 and C8, which make these texts stale.
- C10, the `mutateGate` tier0 command in `mutate_integration_test.go`, changed from `x` to `'true'`. Forced by C3's new row, which needs a green call before any report step runs.
- C11, the `, N errors` suffix that `MutationReport.counts` adds to the summary line (`internal/gatecli/mutationreport.go:532-538`). G3 does not ask for it; see F1.

Decisions and the changes that build them:
- G0: C3 and C7.
- G1: C2.
- G2: C1. It drops only the `runtime.GOOS` comparison itself, not the other-platform branch. The branch body never runs on this platform, so it is an uncovered block and holds no mutation site in any case (`internal/testcover/mutant.go`, `comparesOtherPlatform`, and the noise span check in `fileSites`).
- G3: C3, through `mutation.json` and `SummaryLine`.
- G4: C8, with C6.
- No `Added after Discussion` entries exist.

Every goal point is built, and only C11 traces to no goal point.

**Findings**

- F1: NIT, class `scope`.
  - Location: `internal/gatecli/mutationreport.go:532-538`, `MutationReport.counts`.
  - Summary: the summary line appends `, N errors` whenever the report holds error lines. G3 lists exactly six counts for the line, and the record gives no licence for a seventh. The change description defends it ("so a step whose builds failed does not print all zeros silently") under its own design decisions, which are the writer's unreviewed claim. Either keep it as an extension the count list forces, or drop it to match G3 literally.
  - Evidence: `_lym/discussion/decision-record.md`, `## Goal` item 3; `mutationreport.go:535-537` (`if len(r.Errors) > 0 { text += ", %d errors" }`).

Tool calls: none. My inherited context already held the record, the diff pieces and the change description.
