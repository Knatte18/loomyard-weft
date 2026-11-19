```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: for each change, the readers, docs and rule documents it forces, and whether the diff covers them.

**Per change: the affected set I found, and whether it is covered**

- **`mutate*` types exported as `Mutation*`** (`mutatecheck.go`): `mutate.go`, `mutate_integration_test.go`, `mutation.go`, `mutationreport.go` and `loomcli/mutationnote.go`. Covered. No old name is left (`grep mutateCounts|mutateSurvivor|mutateNotJudged|.callee` comes back empty).
- **`Mutant.callee` replaced by `noise`** (`testcover`): `mutatesBehavior`, `PlanMutants`, `fileSites`, the `testcover` package doc and `linemutant.go`'s doc. Covered.
- **The `mutationCall` extraction and `fillMutateAnswer`'s new signature**: `runMutate` and `mutationStep.answer`. Covered.
- **`changedFunctionsMatch` split out of `ReachReportMatches`**: `ReachReportMatches` and `MutationReportMatches`. Covered.
- **`ReportMutation` accepted in `reports`**:
  - Covered: `decodeGates` and its refusal message, `TestLoadConfig` rows, `GateKey.Reports` doc, the `gateslot` doc's reports paragraph and refusal list, and `template.yaml` keys and comments.
  - Covered: `config_test` `templateDefaults`, `landingdeps_verify_test`'s template replacer, `scopes_integration_test`, the gatecli doc and `run.go`.
- **Template diff keys now list `mutation`**: other fixtures that tune those keys without `reports` inherit the step through the template fill. These are `loomshed/verifygate_integration_test.go:148-150`, `websterengine/verifygate_integration_test.go:50-52` and `configcli_integration_test.go:660`. Whether they still pass is the verify gate's to report, so no finding.
- **Config doc for `reach_budget_min`, `mutation_max_mutants` and `mutation_budget_min`**: the `config.go` field comments and the `template.yaml` comments are updated. The `gateslot` package doc's bound-keys paragraph and its list of non-tier slot holders are not (finding 1).
- **The mutation step as a new slot holder** (`mutationSite`): the gatecli doc (covered), the `gateslot` doc's non-tier holder list (not covered, finding 1), and `PATTERN-gate-slot`, whose rule points at that list (not covered through finding 1).
- **The memory guard now also covers the mutation step**: the gatecli doc's guard paragraph and the `template.yaml` `memory_quit_mb` comment. Covered.
- **The `mutation_report` rubric marker**:
  - Covered: the Code-Review rubric stencil, `rubric_test` allowlist, `shedadapters.RubricNotes`, `ReadRubric`, `BouncerConfig`, `Bouncer.rubricNotes`, `shedrecipe.Env`, `bouncerEntry` and `reviewRubricNotes`.
  - Covered: the `envkit` nil-legal list, `loomcli` wiring and its test, and the docs of `shedadapters`, `shedrecipe` and `loomcli`.
  - Stencil wording: the new section adds a rule and drops or weakens none.
- **The review-note rule**: `PATTERN-coverage-map-reader` "The review note" now names the mutation report. Covered.
- **The `PATTERN-test-speed` report-step bullets**: the new mutation bullet and the edited `lym gate mutate` line are present. The `PATTERN.md` summary line needs no change.
- **`docs/overview.md`**: the gatecli row and the dobby line are covered.
- **"A hand verb for the mutation trial"**: rewritten in the gatecli doc, and no other occurrence remains outside `docs/research`.
- **Commit messages and trailers**:
  - No `Pattern-Departure:` trailer.
  - The `Loosened-Check:` trailer on `TestLoadConfig` names a refusal row the intended behaviour change replaced; no tolerance was widened, so it holds.
  - 28216d3f says log-only and GOOS sites were "the bulk of the survivors no test could reasonably kill". The trial report (`docs/research/speed/mutation-trial-2026-11-19/report.md:131`) says "most of the 12 survivors no test could reasonably kill", so the claim matches.
- **Declared PATTERN changes**: the directive declares none, so there is nothing to check.

**Findings**

1. **MEDIUM**, class `consistency`, location `internal/gateslot/doc.go:62-71` and `internal/gateslot/doc.go:187-204`.
   - **Summary:** the `gateslot` package doc was not updated for the new mutation report step.
     - Its list of non-tier slot holders (doc.go:65-71) names the redundancy step, the reach step's refresh, `lym gate redundant` and `lym gate mutate`. The mutation report step also takes a slot, under its own `mutationSite` label (`internal/gatecli/mutation.go:290-291` and `364-371`, through `mutationCall.run`'s `mutationCheck` holder), and the list leaves it out.
     - The bound paragraphs (doc.go:187-188, 190, 201-204) still name only `lym gate redundant` and `lym gate mutate`.
     - doc.go:203 still says `mutation_max_mutants` caps what "`lym gate mutate` runs … spread over the changed functions in turn", with neither the step nor the new even spread. The diff updated the matching field comments in `config.go` (diff 006 lines 95-105) but not this doc.
   - **Why it matters:** `PATTERN-gate-slot`'s rule allows a gate build "in one of the non-tier holders `internal/gateslot`'s doc lists" (`pattern/PATTERN-gate-slot.md`). The new holder sits outside that documented list.
   - **Fix:** add the mutation report step to the holder list and to the bound-key sentences, and state the even spread for `mutation_max_mutants`.

**Tool calls**

1. Grep of the `gateslot` doc for holders, of `PATTERN.md` for report entries, of `reach_report` users, and of template `reports` lists: the manifest did not hold the `gateslot` doc's holder or bound sections.
2. Read of `gateslot/doc.go` lines 60-75 and 183-206, gatecli holder-label lines, and the specs naming the markers: needed to confirm finding 1's text.
3. Grep for stale "mutation trial" and report-list mentions, and the trial report's survivor breakdown: needed to check commit 28216d3f's claim.
4. Grep of tests that tune the template's diff keys: needed to find fixtures that inherit the new template `reports`.
