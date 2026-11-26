```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I looked for a test row that fails when that behaviour breaks.

**Behaviours and the tests that pin them**
1. `SectionBullets` reads an unindented line right after a bullet as part of that bullet. Pinned by `TestSectionBullets/UnindentedContinuationLine` (`internal/discussionparser/partsplice_test.go:92`). It fails on the merge base.
2. An unindented heading line (`#`) ends the bullet. Pinned by `TestSectionBullets/HeadingEndsBullet` (`partsplice_test.go:93`). It fails if `#` is removed from the exclusion set in `continuesItem`. `bulletLines` uses the same helper, so a mutation inside the helper is caught. No `TestCheckDecisionIDs` row puts a heading right under an acceptance bullet.
3. An unindented line opening an HTML block (`&lt;`) ends the bullet. **No test.** In `TestSectionBullets/CarryOverBlockSkipped` (`partsplice_test.go:95`), a blank line separates the carry-over marker from the bullet, and the skipped lines flush the bullet on their own. That row passes even with `&lt;` removed from the exclusion set. No row in `TestCheckDecisionIDs` has a `&lt;` line either.
4. An unindented line opening a blockquote (`&gt;`) ends the bullet. **No test** in `TestSectionBullets` or `TestCheckDecisionIDs` (`validate_test.go:558-662`).
5. `bulletLines` counts a decision id written on an unindented continuation line as covered. Pinned by `TestCheckDecisionIDs/IDOnUnindentedContinuationLineIsCovered` (`validate_test.go:630-633`).
6. `bulletLines` still ignores unindented prose when no bullet is open. Pinned by `TestCheckDecisionIDs/IDInsideNonBulletProseIsNotCovered` (`validate_test.go:634-637`). This behaviour existed before the diff and still holds.
7. A blank line or whitespace-only line ends the bullet. This existed before the diff and is now the `line == ""` branch of `continuesItem`. Pinned indirectly by `TestSectionBullets/PlainBullets` (the blank line before `## Acceptance criteria` sits inside the section range) and by the blank line in `CarryOverBlockSkipped`.
8. A part whose criterion has an unindented continuation splices and reads back to the same text. The change description claims this. **No test**: `TestSplicePart` (`partsplice_test.go:176-197`) carries only an indented continuation (`"Risk one\ncontinues here."`, fixture at `partsplice_test.go:61-62`). `requireBullets` and the `renderBullets` round trip are never run on an unindented source bullet.

**Findings**

- **MEDIUM**, class `design`, at `internal/discussionparser/decisionids.go:203` (`continuesItem`, the `&lt;` exclusion). The diff adds the rule that an unindented `&lt;` line ends a bullet, and no test row fails if `&lt;` is dropped from the set. `CarryOverBlockSkipped` (`partsplice_test.go:95`) has a blank line before the marker, and the skip already flushes the bullet. This is the exclusion the change description justifies by the carry-over marker (`_lym/landing/summary.md:33`). Fix: add a `TestSectionBullets` row with an HTML comment directly under a bullet, for example `"## Open risks\n\n- a\n&lt;!-- note --&gt;\n- b\n"` wanting `{"a", "b"}`.
- **MEDIUM**, class `design`, at `internal/discussionparser/decisionids.go:203` (`continuesItem`, the `&gt;` exclusion). The diff adds the rule that an unindented blockquote line ends a bullet, and no row in `TestSectionBullets` (`partsplice_test.go:90-97`) or `TestCheckDecisionIDs` (`validate_test.go:589-646`) has a `&gt;` line after a bullet. Fix: add a row such as `"## Open risks\n\n- a\n&gt; quote\n- b\n"` wanting `{"a", "b"}`.
- **LOW**, class `design`, at `internal/discussionparser/partsplice_test.go:176` (`TestSplicePart`). The change description's claim that a spliced part record reads back the same text (`summary.md:24`) is untested for the reported case. The fixture's only continuation is indented (`partsplice_test.go:61-62`), so splicing a bullet whose source continuation is unindented, and reading it back through `SectionBullets`, is never exercised. Fix: make one fixture criterion's continuation unindented and carry it in `part.Criteria`. The existing read-back assertion at `partsplice_test.go:195` then pins the round trip.

Tool calls:
- Read `internal/discussionparser/validate_test.go:540-700`. My context held only the diff hunk of that file, not the existing `TestCheckDecisionIDs` rows and harness.
