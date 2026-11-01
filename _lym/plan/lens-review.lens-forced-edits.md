```yaml
lens: forced-edits
outcome: reported
```

Forced-edits lens: what each card's change forces outside the members it names.

1. BLOCKING — Card 16 (registry rows). `TestNames` pins the sorted list of registry engine names (`internal/shedrecipe/registry_test.go:95-125`, around `"PlanWrite"` at :109). Adding `PartsWrite` and `PartsPlanWrite` fails it, and no card lists `internal/shedrecipe#TestNames`.

2. BLOCKING — Card 16 (registry rows). The conformance test ranges over `Names()` and fails any registry entry with no `minimalEntries` row (`internal/shedrecipe/conformance_test.go:107-111`; map at :62, `func minimalEntries`). No card lists `internal/shedrecipe#minimalEntries` for the two new entries.

3. BLOCKING — Card 17 (recipe rows). `gatedRowFanViolations` reports any row that carries a `gates` list with an engine outside DiscussionWrite/PlanWrite/DarnWrite/DriftVerify/Describe/PRRework/BurlerFix (`internal/loomrecipe/gatequiescence_test.go:55-60`). So the gated `Parts-Write` and `Parts-Plan-Write` rows fail `TestNoGatedRowAuthorizesForkSubagents`. The switch, its failure message and the test's `//testtiming:keep` reason all name the writer engines. No card lists `internal/loomrecipe#gatedRowFanViolations`.

4. MEDIUM — Card 17 (timing phase). The row-to-phase pins live in the test-file var `loomRowPhases` (`internal/reportengine/timing_test.go:273-285`). Its comment says "lists every loom row". The card targets `internal/reportengine#TestComputeTiming` instead of `internal/reportengine#loomRowPhases`, which is the member that must gain the two rows.

5. LOW — Card 16 (gate vocabulary). `gateCapableEntries` (`internal/shedrecipe/entries_gate_test.go:41`) lists every gated entry for `TestGateCapableEntries_RejectLeftoverOldKeys`. Neither new gated entry is added, so their leftover-key rejection goes untested.

6. LOW — Card 9 (seat `Timeout`). The doc comment on `internal/seatengine#Table`'s `Timeout` field ("Timeout bounds every seat's run") becomes false once a seat's own `Timeout` overrides it. `Result`'s `Advisors` doc ("one entry per advisor seat") no longer covers writers. Card 9 does not list `internal/seatengine#Table` or `internal/seatengine#Result`.

7. LOW — Card 11 (writer mapping). The doc comment on `internal/shedadapters#MultiLLMProducer` says "The advisors' results are logged and never judged". After the card, a writer's result decides the step, and the card does not list the type.

8. LOW — Card 17 (recipe rows). `internal/loomrecipe/doc.go:15` says "Drift-Baseline sits between the Discussion-Review ring and Plan-Write". It is now followed by `Parts-Write`, and no card lists that doc.

9. LOW — Card 2 (test file). The Intent puts `TestDecisionIDs` in `decisionids_test.go`, which does not exist yet (the package's tests are in `validate_test.go`). The new test file is not declared as a `**Create:**` path, so `test-tier-unstated` never sees its stated tier. Either name the test as a row of `internal/discussionparser#TestValidate_CleanRecords`'s file, or add the file as a Create target.

Further tool calls:
- grep of `planGate`, `segmentWriter`/`settledSegmentGuard`, `refuseVerbOffRecipe` callers, seam markers and `NamePlanWrite` test pins: to find callers and pins of the members the cards change.
- `timing_test.go:270-285`, plus readers of the tapestry keys, stencil registry tests and recipe-row pins: to find the phase table and config/stencil pins.
- `checklist_test.go:478-500`, `stencilstore_test.go:175-215` and config key-move readers: to check whether those row and stencil lists are exhaustive.
- grep of `"PlanWrite"` engine-name pins plus `interruptpolicy_meta_test.go:51-75`: to find registry and recipe engine lists.
- enclosing function of each pin, plus `gatequiescence_test.go:40-75` and `recipekeys_test.go:20-50`: to name the pinning members and check whether they are exhaustive.
- stencilcli/envkit stencil references, the `TestNames` list body and the package docs naming Plan-Write: to check stencil lists and docs the cards leave stale.
