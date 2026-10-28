```yaml
lens: affected-set
outcome: reported
```

Task: affected-set lens. For each change a card makes, find everything that reads or asserts it, and say which cards cover that set.

**Per change: the affected set, and the cards that cover it**

- **New signatures for `ValidateLoomDiscussion` and `NewDiscussionGate` (card 4).**
  - Set: `shedrecipe.resolvePlainGateClosure` (`internal/shedrecipe/entries_gate.go:155`), `loomCLI.validateDiscussionCmd` (`internal/loomcli/validate.go:72`), `TestNewDiscussionGate` (`internal/loomshed/gates_test.go:112-211`), `TestDiscussionGate_FailureSurfacesItsFindings` (`internal/loomshed/gatefindings_test.go:53`), `TestGateParity_DiscussionGate` (`internal/loomcli/parity_test.go:140`) and the smoke test (`internal/loomcli/smoke_gate_test.go:168`).
  - Covered in full by card 4.
  - Shedrecipe tests that drive a loom "discussion" gate on `newTestEnv` (whose `placeholderIndex` panics if called) reach no citation code: their record is never written, and card 4 adds nothing for a record that is missing.
  - `TestValidateDiscussionCmd`'s records hold no backticks, so its nil `PlanIndex` is never reached.
- **The loom "discussion" gate now requires `WorktreeRoot` and `PlanIndex` (card 4).**
  - Set: `newTestEnv` and `reworkTestEnv` fill both, `envkit.FullEnv` fills `PlanIndex`, and `wire` and `wireLightweight` fill both.
  - Covered by card 4, which adds rows to `TestResolveGateSpec_Rejections` and `TestResolveGateSpec_ValidOneEntryPerVocabularyName`.
- **New findings from D1 and D2 in both discussion checks (card 1).**
  - Set: records driven through the gates in loomshed, loomcli and shedrecipe tests. None has a non-D decision H3, so their verdicts do not change.
  - The fixtures that do carry a non-D decision H3 (`internal/loomcli/decision_test.go:29`, `internal/discussionparser/append_test.go`) go through `AppendDecision`, which still runs `Validate` alone.
  - Covered by card 1.
- **`ResolveGlyphs` added to the `planindex.Index` interface (card 3).**
  - Set: the only full implementer is `planglyph`'s `index`, which `NewSlottedIndex` also returns.
  - Fakes that embed the interface: `envkit.noFindingsIndex` (method added by card 4), `shedrecipe.noFindingsPlanIndex` (card 8), `placeholderIndex`, `indexkit.ValidateStub` (wraps a real index), `shedfake`'s index seam and websterengine's `gitfake`. Webster never calls `planContextFindings`.
  - Covered, apart from one stale doc enumeration (finding 1).
- **New signature for `planContextFindings` (card 8).**
  - Set: `ValidatePlan` and `ValidateReworkPlan` (`internal/loomshed/gates.go:269`, `:330`).
  - Covered by card 8.
- **New `PlanContext` fields and `Env.TierTags` (cards 7 and 8).**
  - Set: the builders `planContextOf` and `validatePlanCmd`, and the `Env` literal in `wire`.
  - Only the loom recipe carries "plan" gates (`contracts/recipes/loom-recipe.yaml:227`, `:353`), so darn's and batten's `Env` builders are unaffected.
  - The reflective walks `NilSeams` and `TestFullEnv_PathFieldsAbsoluteUnderOneRoot` ignore a non-func slice.
  - Covered by cards 7 and 8.
- **The new `doc-go-unlisted`, `seam-test-unlisted` and `test-tier-unstated` findings in existing go-plan tests (cards 7 to 9).**
  - Set: the go-plan fixtures in loomcli parity, validate and overlay tests and in loomshed `gates_test`. None writes a `doc.go` or a seam marker, and the one `_test.go` target is an Edit, not a Create (`internal/loomcli/parity_test.go:491-499`), so existing verdicts hold.
  - Covered.
- **The guard scan now admits `//lym:seam` lines (card 5).**
  - Set: `cmd/testtiming` already admits `//lym:` lines between a keep line and its func line (`cmd/testtiming/main.go:55-57`, `keep.go:14-21`).
  - `gatecli`'s switch on the scan error kind (`internal/gatecli/cli.go:212-219`) only ever receives guard-scan errors, and `FindGuardTests` never emits `SeamMalformed`.
  - Covered by card 5.
- **Markers on today's reflective tests (card 6).**
  - Set, from every `NumField`, `VisibleFields` and `NilSeams` call site in a test file:
    - websterengine `TestConfigTemplate`
    - loomengine `TestConfigTemplate_ContainsEveryConfigYAMLTag`
    - fabricengine `TestMergeVocabulary_ResultAndOptionsFieldsAreSideFree` and `TestMergeState_Record` (each through its helper)
    - loomcli `TestLandingDeps_EveryFieldPopulated`
    - envkit's four tests
    - loomcli's three `TestWire_*` tests and battencli's `TestWire_EverySeamFilled`
  - Covered in full by card 6.
- **Stencil text (cards 4 and 9).**
  - Set: `contracts/stencils/govalues_test.go:103` bans Go tier commands such as `go test` in `loom-template-plan`. Card 9's sentence names only check IDs, so it passes.
  - The marker tests (`markers_test.go:109-112`) pin headings, which the cards leave unchanged.
  - Covered.
- **Help text (cards 1, 4, 7, 8 and 9).**
  - No test pins the `validate-plan` or `validate-discussion` `Long` text.
  - `docs/overview.md:535` describes `validate-discussion` generically and stays true.
  - Covered.

**Findings**

1. **LOW, consistency: card 3 leaves `resolveTargets`' doc comment stale.**
   - Location: card 3 (`03-index-resolve-glyphs.md`), at `internal/planglyph#resolveTargets`.
   - The doc comment names every consumer of a batched Resolve answer: "statusFindings, createFindings, DetectDrift's post-repair revalidation, DoneChecks".
   - Card 3 adds `ResolveGlyphs` as a new positional consumer through `Resolve`, but does not list `resolveTargets` or rewrite the doc. The comment's claim about every consumer goes stale.
   - Fix: add `internal/planglyph#resolveTargets` under Edit and replace the list with the group, for example "every consumer of a batched Resolve answer".
   - Evidence: `internal/planglyph/repo.go`, the doc comment directly above `func resolveTargets` (about lines 50-53).

2. **NIT, consistency: card 4 does not touch `wireLightweight`'s doc comment, which lists the fields the validate verbs read.**
   - Location: card 4 (`04-citation-check.md`), at `internal/loomcli#loomCLI.wireLightweight`.
   - The doc says `validateDiscussionCmd`/`validatePlanCmd` "read only DecisionRecordPath/SupportLogPath/AnchorPath/WorktreeRoot".
   - After card 4, `validate-discussion` also reads `PlanIndex`. The comment already left out `validate-plan`'s `PlanIndex` read, so the gap partly predates this task.
   - Evidence: `internal/loomcli/wiring.go:255-258`.

Tool calls (all six used):
1. Grepped the callers of the discussion gate, any full `Index` implementers, readers of `//lym:guard` and `testtiming:keep` outside `impactset`, and files listing check IDs. The pre-read had no repo-wide caller and reader sweep.
2. Read `cmd/testtiming` and `gatecli` for how they treat directive lines, to check card 5's claim about `testtiming`.
3. Read `gatecli`'s switch on the scan error kind, which recipes carry plan or discussion gates, and the `Env` literal builders, to bound the `PlanContext` and `Env` builder set.
4. Read battencli's `Env` builder, to see whether it wires plan gates.
5. Grepped tests and docs that pin help or stencil text, which the pre-read did not cover.
6. Read `TestHubGenericStencils_NameNoGoTiers` and the `docs/overview.md` mentions, to judge the stencil and doc constraints.
