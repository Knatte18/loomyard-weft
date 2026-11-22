```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change the diff makes, find what else calls, reads, asserts or documents it, and flag any of those the diff leaves wrong.

**Set per change**

- **Each report step's `run` split into `judgeReport` plus `writeReport`** (`reach.go`, `redundancy.go`, `mutation.go`). The methods are unexported and called only inside `internal/gatecli`. Covered:
  - production caller `run.go`, through `judgedReports` in `reports.go`;
  - tests `reach_integration_test.go`, `redundancy_integration_test.go` and `mutate_integration_test.go`, every call site updated.
- **Report steps now judge beside the last tier's first run and write after the lints** (`run.go`, `reports.go`). Rule documents and docs that describe when the steps run:
  - `internal/gatecli/doc.go` "Report steps" section: covered.
  - `internal/gatecli/reach.go` and `redundancy.go` file headers: covered.
  - `internal/gateslot/tiers.go` `Reports` field doc: covered.
  - `internal/gateslot/template.yaml` comment: covered.
  - `internal/gateslot/doc.go:79`: **not covered**.
  - `internal/gateslot/doc.go:45`: **not covered**.
  - `pattern/PATTERN-test-speed.md:225`, `:329`, `:344`: **not covered**.
  - `docs/overview.md:959`: still roughly true, since "runs last" fits the write. Not a finding.
- **Report steps take their gate slot beside the last tier's slots.** Documents:
  - `internal/gatecli/doc.go`: covered.
  - `internal/gateslot/doc.go:45`: not covered, already counted above.
  - `pattern/PATTERN-test-speed.md:225`: not covered, already counted above.
  - `pattern/PATTERN-gate-slot.md`: says only "the map run's slot ask" and nothing about timing, so it is still correct.
- **Memory guard `foreign` field and `withoutForeign`.** Covered:
  - `memoryguard.go` `run` doc;
  - `internal/gatecli/doc.go` memory guard section;
  - `runGuardScripted`'s only callers, in `memoryguard_test.go`.
- **`testRequest.commands`, `runningCommands` and `runInvocations` registering tier commands.** All readers and writers are inside `gatecli`, and a nil set is safe for every other `testRequest` builder. Covered.
- **Scenario subtest name changed** in `test_integration_test.go:1537`. No other site references the name. Covered.
- **Stencils.** The grep over `contracts` found no stencil wording about report-step timing. Nothing to cover.
- **Commit messages and departures.** There is no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer, and none is needed:
  - no test limit was widened;
  - the change is not a bug fix;
  - no declared PATTERN changes exist.

**Findings**

1. **MEDIUM, consistency** at `pattern/PATTERN-test-speed.md:225`, `:329` and `:344`. The background file still says the report steps run after the tiers released their slots, after the lints, and after the tiers and lints. The diff now judges them beside the last tier's first run and only writes them after the lints. That is a repository rule document the diff leaves stale.
   - Lines 225, 329 and 344 come from the grep. The same text sat at `PATTERN-test-speed` "Tests run through the gate" and "The coverage map and the gate's redundancy report" in my context.
   - Contradicts `internal/gatecli/run.go` `startReports` at the `reportsStart` loop, and `internal/gatecli/doc.go:366`.
2. **MEDIUM, consistency** at `internal/gateslot/doc.go:45`. It says the redundancy report step "takes one slot after the call's tiers released theirs". Under the diff the step asks for its slot while the last tier holds its own. The diff's own `internal/gatecli/doc.go` now says "beside the slots the last tier holds".
   - Evidence: `internal/gateslot/doc.go:45` against diff 001 lines 39–41.
3. **LOW, consistency** at `internal/gateslot/doc.go:79`. It says "A GateKey may hold `lints` and `reports`, the steps its call runs after its tiers". The reports now judge beside the last tier, so the line contradicts the `Reports` field doc in `tiers.go:118` that the diff updated.
   - Evidence: `internal/gateslot/doc.go:79` against `internal/gateslot/tiers.go:118`.

**Tool calls**

1. Grep of the repository for report-step ordering wording, because my context held none of the docs outside the diff except the pattern backgrounds.
2. Read of `internal/gateslot/doc.go:40–48` and `:76–82`, plus a grep of `contracts`, `docs` and `_bolt`. The first grep found only the matching lines, so I read the doc spans around them and searched the other doc trees.
3. Read of `docs/overview.md:955–962` and grep of `internal/gatecli/doc.go:376–440`, to see the context of the overview hit and of the doc's other report paragraphs, which my context lacked.
