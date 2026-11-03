```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per behaviour, the test that pins it**
- **Gated fragment chosen when `GateFixed` is set:** `TestDarnSpec` row "a cold fixer on the verify gate…" (`internal/darnengine/spec_test.go`). It asserts `gatedNoGateStep` is present and `ungatedTestStep`, `ungatedSubagentGate` and "each leaving the tree green" are absent.
- **Ungated fragment chosen otherwise, so the prompt keeps today's text:** `TestDarnSpec` row "first spawn…". It asserts `ungatedTestStep` and `ungatedSubagentGate` are present and `gatedNoGateStep` is absent. Byte-for-byte equality with the old Step 4 is not pinned, only those two lines. The rubric exempts static prose.
- **`subagent_model` filled in both fragments:** both `TestDarnSpec` rows assert `model: "haiku"`. `TestStencils_TopLevelMarkers` pins `subagent_model` as each fragment's only marker (`contracts/stencils/markers_test.go:56-57`).
- **`make_change` marker in the writer stencil, no stale `subagent_model`:** `TestDarnSpec` renders the real seeded stencil through `seatengine.Prompts`, so an unfilled or extra marker would fail. No `markers_test` row pins `darn-template-write` itself. That is covered indirectly.
- **`makeChangeList` error paths (a fragment missing from `stencilsDir`, or a `Fill` failure), wrapped as "darn: DarnTable: Step 4 list":** none. `TestDarnSpec_Refusals` removes only `darnStencilName` (`spec_test.go:226`).
- **`wireDarn` sets `GateFixed` from `writerFixerOn(c.recipe, darnWriteEngine, darnGateEntry, c.env.Setting, pairSlug)`, including passing `pairSlug` through from `wire`:** none. `gatefixer_test.go:81` tests `writerFixerOn` itself, and `gatefixer_test.go:33` and `gatefixer_test.go:161` cover the seam's pair slug. `wiring_darn_test.go` has no hit for `GateFixed`, `gate_fixer` or `pairSlug`. It only checks that `DarnTable` is non-nil or nil.
- **`DarnInputs.GateFixed` doc and `doc.go` text:** prose, exempt.

**Findings**
1. MEDIUM, `design`, `internal/loomcli/darnwiring.go:240-243` (evidence: `wiring_darn_test.go` has no hit for `GateFixed`, `gate_fixer` or `pairSlug`).
   - The wiring from the Darn-Write row's `verify` element and the pair slug to `GateFixed` has no test.
   - A regression would pass every test: a wrong `gate` or `engine` argument, the pair slug dropped, or `GateFixed` unset. It would leave the prompt on the ungated text with the fixer on, which is the bug this task fixes.
   - The record's "chosen on the same condition the row gate uses" is only checked by reading.
   - The fix is a `wireDarn` test that builds `DarnTable` with a pair slug and `darn.gate_fixer` set and asserts the gated rendering, and with either one missing asserts the ungated one.
2. LOW, `consistency`, `internal/darnengine/spec_test.go:205-206` and `spec.go:115`.
   - `DarnTable` now reads the fragments and returns an error for a missing one. The `TestDarnSpec_Refusals` comment still says "the table builder reads none".
   - No row covers a missing or unfillable fragment.
   - The fix is one row removing `makeChangeStencil`, and one removing `makeChangeGatedStencil` with `GateFixed` set. Each should expect an error naming the stencil, plus a corrected comment.
3. LOW, `scope`, `internal/darnengine/spec_test.go`, the gated row's `wantInPrompt` and `wantNotInPrompt`.
   - Several assertions repeat static fragment prose: "tells the subagent to run no `lym gate test`", "act on that alone", and "each leaving the tree green".
   - The record asks the test to pin both renderings, so `gatedNoGateStep` and `ungatedTestStep` suffice.
   - The rubric's test economy flags assertions on a stencil's static prose. Trim the extra ones.

**Tool calls**
1. One `grep` over `wiring_darn_test.go`, `gatefixer_test.go`, `spec_test.go` and `*_test.go` for `GateFixed`, `gate_fixer`, `pairSlug`, `make_change`, test function names and `darn-template-write`. My context lacked those test files' contents.
