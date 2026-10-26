```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. Per behaviour, the test that pins it:

1. **Blocked stop at a marked row goes to the arbiter, and a hand-on asks the parent once.**
   - `TestDecideDrive`, new row "blocked stop at a row the arbiter judges first goes to the arbiter" (`driverules_test.go:78`).
   - `TestGoDriver_Settle`, new row "a blocked stop at a row the arbiter judges first goes to the arbiter with no tackler call…" (`drive_test.go:336`).
   - It asserts `wantCalls` of `step` and `arbiter:` only, so the missing tackler call is pinned.
2. **An unmarked row, a nil `JudgesFirst` or a nil `Arbiter` leaves the tackler path unchanged.**
   - `TestGoDriver_Settle`, new row "…does not judge first goes to the tackler" (`drive_test.go:344`).
   - The existing blocked-stop rows, which set no `judgesFirst`, pin the nil-seam case.
   - The existing `noArbiter` rows pin the nil-`Arbiter` case.
3. **Row order after hand-over.** `TestDecideDrive`, "…on a changed build hands over" (`driverules_test.go:79`).
4. **Landing, budget and precondition halts still hand back at a marked row.** `TestDecideDrive`, three rows (`driverules_test.go:80-82`).
5. **Pause and ownership rows precede the new row.** None. The new row sits after both, but no row sets `ArbiterFirst` with a paused stop. This is a minor gap, and the focus names pause.
6. **The wiring marks only the Webster row.**
   - None for `internal/loomcli/drivewiring.go:155`.
   - The driver tests use a fake `JudgesFirst`.
   - Nothing fails if the wiring marked a different row, none, or darn or batten as well.
7. **`AddedDecisionTexts`, with entry extents, order, a fenced heading, no entry, a missing record and an empty path.**
   - `TestAddedDecisionTexts` (`append_test.go:358`) covers all of these in four rows.
   - The `second` entry holds a fenced heading, which pins the skip.
   - A read error other than not-exist has no test.
8. **`addedDecisionSpans` refactor.** The existing `IdentityText` tests that already covered `dropAddedDecisionEntries` pin it. No new test duplicates them.
9. **Render of `parent_decisions`, with entries in order and joined by a blank line, and `none` for no entries.**
   - `TestRenderPrompts`, rows "master renders each parent decision in order…" and "master renders none…" (`template_test.go:830, 839`).
   - An empty record path reaches `RenderMasterPrompt` as a nil slice, the same input as no entries, so the "none" row covers it.
10. **`parent_decisions` is a required marker.** `TestTemplates_FillRequiresEveryRequiredMarker`, the added `required` entry (`template_test.go:480`), and the marker value in `masterTemplateMarkerValues`.
11. **`Run` reads `Geometry.DecisionRecordPath` and returns a transient-refusal error on a read failure.** None. Neither the pass-through nor the `runlevel.go:740-743` error path is tested, and the rendering tests call `RenderMasterPrompt` directly.
12. **Stencil: every stuck rung ends in `outcome: stuck` alone, and the rule is stated once.** No test, which is exempt as static prose.

Findings:

- **LOW, consistency (test economy), `internal/websterengine/template_test.go:839-843`.**
  - Summary: the row "master renders none for no parent decision" asserts the stencil sentence `never raise a question one of them answers.\n\nnone\n\n## A plan-drift`. That is an assertion on the stencil's static prose.
  - It also couples to the two neighbouring paragraphs and the next heading.
  - The first new row's `inOrder` already pins the heading text, and `TestTemplates_FillRequiresEveryRequiredMarker` pins that the marker is filled.
  - Fix: assert only the `none` render, for example with a narrower `contains`. Or drop the prose and reuse the order row's markers around `none`.
- **LOW, scope (uncovered behaviour), `internal/loomcli/drivewiring.go:155`.**
  - Summary: the wiring that marks only `loomshed.NameWebster` has no test. It is the behaviour that makes the change reach Webster, and the loomcli tests do not assert the seam.
  - Fix: add a row to an existing `loomcli` driver-deps test, if one builds `goDriverDepsFor`, asserting `JudgesFirst("Webster")` is true and another row name is false.
- **LOW, scope (uncovered behaviour), `internal/websterengine/runlevel.go:740-743`.**
  - Summary: `Run`'s read of the decision record and its transient-refusal error path have no test. A `Run` test with a record holding an added entry would pin both.
  - Fix: add a row to an existing `Run` test that sets `Geometry.DecisionRecordPath` and checks the written master prompt. Add one row with an unreadable record, such as a directory path, that checks the error text.

No existing test is duplicated by a new one.

Tool calls: one (1/6), `git diff` of the three test files, because the manifest held none of the test diffs.
