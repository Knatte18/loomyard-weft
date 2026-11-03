```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set, i.e. what the two commits force outside the lines they touch.

**Per change, the set I found:**

1. **`CheckEditTestFile` signature, new `glyphless` argument.**
   - Callers: only `planglyph/index.go` `ValidateFormat` (line 53) and `ValidateRework` (line 62), both passing `glyphlessFile`. `edittestfile_test.go` is the only other caller and passes a non-nil func. Covered.
   - Docs: the `edittestfile.go` comment, `planparser/doc.go:145-146`, `planglyph/doc.go:29` and the `index.go` comments all agree. Covered.
   - Specs: `loom-plan-checks-spec.md` and the `loom-block-plan-cards.md` stencil are updated. Covered. No other spec, stencil or doc restates the edit-test-file row: the grep hits were `loom-plan-checks-spec.md` lines 13 and 19, which only list the check, and `docs/research`, which holds historical bench output.
   - Other checks: the glyph resolve pass, `glyph-not-found`, `redundant-file-target` (`plangate.go:19`, which fires only beside a member glyph) and `directory-target` were not run against a file self glyph that has no symbols. Dispatch and format validation were not re-run for an admitted glyphless test file.

2. **Renumber result and report semantics: `prose_mentions` narrowed and `rewritten_refs` widened.**
   - Producers and consumers: `planparser/renumber.go`, `loomcli/renumber.go` (envelope plus both `--help` texts) and `loomcli/doc.go` all agree. `loom-plan-spec.md` is updated. Covered.
   - Other readers: no other Go, stencil or spec file reads `prose_mentions` or `rewritten_refs`, apart from the tests. Covered.
   - Removed symbols (`proseMentions`, `planAfterRenumber`, `movedOldNumbers`, `namesOldNumber`, `createdSlug`): no remaining references. Covered.
   - Doc comments in `renumber.go` (`ProseMention`, `RewrittenRef`, `RenumberResult`) are updated. Covered.
   - Stencils: no stencil describes insert-card or remove-card reporting. Covered.

3. **PATTERN.**
   - No declared PATTERN change in this task. The changes touch no `PATTERN-sole-parsers` rule: `planparser` stays the sole plan writer, and `glyphless` is injected, so it reads no file.
   - The `PATTERN-test-speed` tier0 spawn ban applies to `index_test.go`. `TestIndex_AppendsPlanChecks` is tier0 and now calls `writeFixtureRepo`, which goes through `plankit.Repo`. That call is not shown spawning git here, and other tier0 tests in the package use it the same way. Covered.

4. **Commit messages.**
   - Neither commit carries a `Pattern-Departure:` trailer, and none is needed.
   - Commit 17fe6785 says remove-card rewrites bare references too. That is consistent with the diff.

**Findings:**

- **LOW, consistency.** `contracts/specs/loom-plan-spec.md` (renumber paragraph, around line 226) reads "they leave it as is and report its line, and a card id never needs that, so a card refers to another card by its card id." The subject "they" now has no antecedent after the rewrite, and "never needs that" no longer follows from the sentence before it. The spec now describes the verbs' behavior ambiguously. Evidence: the diff hunk at `loom-plan-spec.md` line 222.

- **LOW, consistency.** The `--help` text of `insert-card` (`loomcli/renumber.go` line 233) says only "a bare range the new card falls inside" is left as is. It omits that a range ending at the insert position is also kept when the insert position falls inside it, but `mapsFaithfully` (`renumber.go:729`) uses `previous < position <= number`, so a range ending exactly at the position is kept. A range starting at the position is not kept, so the text is imprecise at this boundary. Also, `remove-card`'s help says "a range card N falls inside", but the code also keeps a range whose lower bound is below N and whose upper bound equals N. The help is imprecise at that boundary, not wrong in substance.

**Tool calls (2 of 6):**
1. `grep` for `prose_mentions`, `rewritten_refs`, `edit-test-file` and "never rewritten" across md, go and yaml files outside `_lym`, to find any other doc, stencil or spec describing the changed behavior. My context had only the diffed files.
2. `grep` for `redundant-file-target` and `file-target` in `planglyph` and `planparser` production code, to see whether another check acts on a file self glyph with no symbols. My context lacked `plangate.go`.
