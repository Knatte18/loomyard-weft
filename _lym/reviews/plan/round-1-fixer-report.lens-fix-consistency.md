```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency, checking that the changed text agrees with the sibling cards, decisions and cross-references it touches.

Findings:

1. **LOW**
   - **Changed text:** card 12, **ImpactSummary**: "four blocks are new and registered, and five pointers to the format name them".
   - **Summary:** the count no longer matches card 12's own **Pointers to the format** paragraph.
   - **Evidence:** that paragraph rewords six pointers: the status line, the "produced by" line and the reference-list entry of `loom-plan-spec.md`, the opening pointer and the reference-list entry of `webster-spec.md`, and the webster entry of `docs/overview.md`. The rubric pointer in `loom-rubric-plan-review.md` makes seven. Under the prose rule against a pinned tally, the fix is to drop the number.

2. **LOW**
   - **Changed text:** card 16, **Gate vocabulary**: "`TestResolveGateSpec_ValidOneEntryPerVocabularyName` gains a `parts` row".
   - **Summary:** the `parts` gate now needs a non-nil `Setting` that answers `tapestry.parts_max`, and the card does not say that this row's env supplies one.
   - **Evidence:** the test builds its env with `reworkTestEnv(t)` (`internal/shedrecipe/entries_gate_test.go:61`). No `Setting` appears in that builder's opening lines (`entries_gate_test.go:17` onward). Before the fix the gate read only Env paths and `PartsMax`, so the row needed nothing more. Two other envs are already fine: `envkit.FullEnv` sets `Setting: TemplateSetting`, and card 1 adds `parts_max: 6` to the template. The fix is to say the row's env sets `Setting` to `envkit.TemplateSetting`, which is what the drift tests do (`internal/loomrecipe/drift_test.go:36-40`).

No other disagreements found:
- Cards 6, 7, 14 and 19 now agree on the parts-file and canonical-export rules (`ExportMet`, `part-export-missing`, chair never edits the parts file).
- The overview Shared Decision and card 12 agree on the four-block split.
- Card 18's validate-parts still reads `parts_max` from `LoadTapestryConfig`, which is consistent.
- Card 17's sequence fixture can fall back to `TemplateSetting`, as the drift tests do.

Tool calls:
- Grepped the plan cards for twins of the changed terms (respell, `PartsMax`, `part-export-missing`, `ExportMet`, `parts_max`, golden). My context did not hold the untouched cards 1, 3, 4, 13 and 15.
- Grepped card 17, card 4, card 13 and envkit for `Setting` and spelling rules, to see whether the `Setting`-based `parts` gate reaches the test envs. My context lacked those files.
- Read the `FullEnv` and vocabulary-test env lines, to learn whether `Setting` is filled. My context lacked them.
- Grepped the two env builders' bodies for `Setting`; the first grep matched nothing.
- Located `reworkTestEnv` and `buildSequenceFixture`, and the drift test's `Setting` override pattern.
- Read the opening lines of `reworkTestEnv`, to check for a `Setting` field.
