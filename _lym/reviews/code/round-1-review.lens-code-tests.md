```yaml
lens: code-tests
outcome: reported
```

The code-tests lens: for each behaviour the diff adds or changes, the test row that pins it, or none.

**Per behaviour, the test that pins it**

- **loom `code_review_ask_parent` / `code_review_parent_timeout_min`** (defaults, override, refusal of 0): `TestLoadConfig_Loads` (`templateDefaults` plus the "Code-Review parent-review keys overridden" row) and `TestLoadConfig_Refuses` ("zero code_review_parent_timeout_min"). Pinned.
- **drift `judge_ask_parent` / `judge_parent_timeout_min`** (load, default 30, refusal below 1): `TestLoadConfig`, rows at `internal/driftengine/config_test.go:333-342`. Pinned.
- **The drift keys open to the local layer:** `TestModules_EveryTemplateKeyClassified` (`internal/configreg/configreg_test.go:152`). Pinned.
- **`CapParentReviewWait` per perch** (own key, cap, no-room reason naming its key): the two `TestCapParentReviewWait` code-perch rows. Pinned.
  The panic on a perch that asks no parent is a wiring fault and needs no row.
- **`parentAskDue` always asks on an `AskEachRound` seat:** `TestBurlerReviewProducer_Call_ParentSeatAskingEachRound`, round 2 after a satisfied round 1. Pinned.
- **`askSubject` from round 2 names the round before's review and fixer report:** same test (`internal/shedadapters/parentseat_test.go:402-406`). Pinned.
- **`askSubject` in round 1 on an `AskEachRound` seat gives the subject alone:** none.
- **`ParentAskLine` takes tag and subject:** `TestParentAskLine` (exact text) and `TestBurlerReviewProducer_Call_ParentAskingRule`. Pinned.
- **`codeParentSeat` sets Tag, Subject, `AskEachRound`; `discussionParentSeat` sets Tag, Subject, `RecordWrittenAt`; each seat reads its own key through `baseParentSeat`:** none. The mutation survivors at `parentseatwiring.go:62-64` and `76-78` confirm it.
- **`askParentSetting` reads the key it is given:** `TestAskParentSetting_ReadsTheBooleanSetting`, with `codeAskParentKey` only. Pinned for the helper.
- **Seat placement on Discussion-Review and Code-Review only:** `TestLoomSegmentSettings_ParentSeatsLandOnTheirSegments` and `TestWire_ReviewKeysReachTheEnv` (non-nil only). Pinned.
- **`awaitCap` resolved per perch in `wire()`:** none. It is a one-line closure; no finding.
- **Parent agreement gate:** `TestParentAgreementGate` covers no ask, open before deadline, at deadline, satisfied, findings before and after the rewrite, no orch strand, and failed post. Pinned.
  - **Re-post of a held pending-post ask** (`agreementGate`, `case ask.State == ParentAskPendingPost`): none.
  - **`Resolve` error leading to no-parent** (`sendJudgeAsk`): none. Only the `!recorded` row exists.
  - **Timeout race where `EndParentAsk` returns done** (`judgeparent.go:162`): none. It is reachable only through a stale in-memory ask, so a test cannot reasonably reach it; no finding.
  - **`Attempts` budget of the entry** (`judgeparent.go:71`): none.
  - **Unreadable judgement, record or report** as a gate error: none. Same pattern as the rest of the engine; no finding.
- **`parentAgreement`** (key on; key off with a held ask; key off with no ask; no seat): `TestDriftVerify_ParentAgreementCarriedWhileAsking`. Pinned.
  A `Config()` error is a plain return; no finding.
- **`tableWithParent` appends the entry after the judgement entry in the table `runAgent` hands the seat runner:** none. `TestDriftVerify` reaches it only with the key off; the test above calls `parentAgreement` directly.
- **`JudgeAskLine`:** `TestJudgeAskLine` (exact text). Pinned.
- **`judgementSummary`:** `TestJudgementSummary`. Pinned.
- **`awaitParentFindings`:** substring rows in `TestParentAgreementGate`. Pinned enough.
- **The `driftDeps` `JudgeParent` seat** (files under the anchor, slug, `ParentStrand` routing, post as `loomProgram` tagged with the run): none.
- **Help text, `orch-role.md`, `doc.go`, template comments:** static prose, exempt.

**Findings**

1. **MEDIUM, design.** `internal/loomcli/parentseatwiring.go:76-78` (`codeParentSeat`) and `62-64` (`discussionParentSeat`).
   No test pins the code seat's `Tag`, `Subject` and `AskEachRound`, nor that each seat reads its own key.
   A test calling `parentSeats(in)` with a recording setting lookup could assert each seat's Tag, `AskEachRound` and the key `AskEnabled` asks for.
   `TestAskParentSetting` (`parentseatwiring_test.go:46`) now passes only `codeAskParentKey`, so `discussion_review_ask_parent` is no longer asserted anywhere.
   Evidence: the mutation report's survivors at `parentseatwiring.go:62-64` and `76-78`, and `parentseatwiring_test.go:46,278`.
2. **LOW, design.** `internal/driftengine/verify.go` `tableWithParent`.
   No test asserts that the judge's table carries the parent-agreement entry after the judgement entry.
   `TestDriftVerify_ParentAgreementCarriedWhileAsking` (`judgeparent_test.go:639`) calls `parentAgreement` directly, and `verify_test.go` has no reference to the entry.
   Evidence: grep of `internal/driftengine/*_test.go`.
3. **LOW, design.** `internal/driftengine/judgeparent.go` `agreementGate`, the `case ask.State == burlerengine.ParentAskPendingPost` branch.
   The crash-recovery re-post of a held pending-post ask has no row: the prior asks in `TestParentAgreementGate` are only open or done.
   Evidence: `judgeparent_test.go:495-525`.
4. **LOW, design.** `internal/driftengine/judgeparent.go` `sendJudgeAsk`, the `case err != nil` branch.
   A `Resolve` error recording no-parent has no row; the fixture's `Resolve` never errors.
   Evidence: `judgeparent_test.go:114`.
5. **LOW, design.** `internal/shedadapters/parentseat.go` `askSubject`.
   Nothing checks that round 1 of an `AskEachRound` seat names the subject alone, with no round-0 review or fixer paths.
   `TestBurlerReviewProducer_Call_ParentSeatAskingEachRound` covers only round 2.
   Evidence: `parentseat_test.go:384-407`.
6. **LOW, design.** `internal/loomcli/driftwiring.go:164-172`.
   No test pins the judge's parent seat wiring: ask files under the anchor, slug, routing, and post signed as `loomProgram` with the run tag.
   Evidence: grep of `internal/loomcli/*_test.go` finds no `JudgeParent`.
7. **NIT, design.** `internal/driftengine/judgeparent.go:71`.
   The entry's re-prompt budget, `Attempts: parentAgreementAttempts`, is asserted by no test; it is a mutation survivor.
   Evidence: the mutation report, and `judgeparent_test.go:554-556` read no `Attempts`.

Tool calls:
1. Grep of the driftengine, loomcli and shedadapters tests for references to the parent-agreement entry, the seat wiring, `AskEachRound`, and a pending-post or `Resolve`-error row. The manifest named the tests but not whether `verify_test.go` or the loomcli tests assert the new wiring.
