```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff changes, the test that pins it, and a test-economy check.

**Per behaviour: the test that pins it**
- **`EntryRuleText` cut (`internal/pattern/check.go`).** `TestEntryRuleText` in `check_test.go` pins it with six rows: bare text, `(test)`, link, both, a `(test)` inside the text kept, and a line with no backticked name. Each branch of the function has a row that fails if it breaks.
- **`Check`'s `long-line` finding measures the rule text.** `TestCheck` pins it. The row "rule text over the cap" asserts the finding and its new message through `wantMessage`. The row "rule text at the cap behind a long name, marker and link" expects no finding on a line past the old 360 cap. Either row fails if the measure reverts to the whole line.
- **Plan gate `pattern-entry-line-cap` (`internal/planparser/patternentry.go`).** `TestValidate_PatternEntryLineCap` in `planparser/validate_test.go` pins it:
  - Cap+1 fails and exactly the cap passes.
  - Text at the cap behind a long name, marker and link passes.
  - Text over the cap behind a marker and link fails.
  - The length parse now reads the new "rule text at" wording, so the message change is asserted too.
- **Discussion `pattern-change-over-cap` measures bullet text with the tail excluded.** `TestValidate_PatternChangeOverCap` in `discussionparser/validate_test.go` pins it.
  - Over-cap rows: adds, changes, appended and multi-byte, plus `OverCapBeforeTail`.
  - At-cap rows: plain, multi-byte, appended and `AtCapWithLongNameAndTail`.
  - The `RemovesNeverMeasured` row is kept.
  - The detail text is asserted through `over` and the `%d cap` string.
  - The `AppendedAtCap` row passes at the cap with the appended suffix, which shows the suffix is stripped before measuring.
- **`lyx loom decision add --pattern-text` refuses an over-cap text.** The `PatternTextOverCap` row of `TestAppendDecision_FindingRestoresRecord` in `append_test.go` pins it. Its text is now `patternEntryTextCap+1` and it expects `checkPatternChangeOverCap` with the record restored.
- **discussionparser and `internal/pattern` agree on cap and extraction.** `TestPatternEntryTextCap_EqualsThePatternCap` pins it. It compares the constants, then compares `patternRuleText` with `pattern.EntryRuleText` on five shared rows. Those rows cover bare text, `(test)`, link, both, and a `(test)` inside the text.
- **The plan gate and the `lyx loom validate-plan` verb report the same finding.** `TestGateParity_PlanGate` in `parity_test.go` pins it. Its over-cap fixture is now `MaxEntryTextChars+1`.
- **The repository's own `PATTERN.md` passes Check.** The existing `go test` check of the real file pins it. I found no new test for it and none is needed.
- **Static prose.** The doc, spec and stencil wording changes need no tests.

**Test economy**
- No new test duplicates an existing one. The additions are rows on the existing tests, plus `TestEntryRuleText`, which is the one test of the new exported function.
- The `Check` rows and `TestEntryRuleText` look alike but are not redundant. One covers `Check`'s use of the cut, the other the cut itself.
- The planparser and discussionparser tail rows test different functions, and the discussionparser copy is deliberate under the decision record.
- I found no per-helper redundancy, no review-fix test added without a coverage gap, and no assertion on stencil prose.
- The new rows in `patternRuleText`'s own tests cover only a tail with both `(test)` and a link. The link-only and `(test)`-only cuts are pinned by the equality test against `pattern.EntryRuleText`, so I count that as covered.

**Findings:** none.

**Tool calls:** none. The diff and files in my context were enough.
