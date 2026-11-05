```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test that pins it, or none.

**Per-behaviour list**

1. `docsLinkIsResearchData` skips a study's `data/` at depth 3 below `docs/research` (`topic/x/data/prompts/step.md`).
   - Pinned by `TestEnforcement_MarkdownLinks`, subtest "research study data at any depth is skipped while the report itself is scanned" (`internal/lymcwd/docslink_test.go`, diff hunk near line 565).
   - Reverting the predicate to `parts[3] == "data"` makes this row report a break, so it would fail.
2. Series depth (`topic/series/x/data/output.md`) is skipped.
   - Pinned by the same subtest, row `docs/research/topic/series/x/data/output.md`.
   - The old `parts[3]` check would not skip it, so the row fails on the old predicate.
3. A study's `report.md` stays scanned.
   - Pinned by the same subtest, row `docs/research/topic/x/report.md`, which is in the expected breaks.
4. The `len(parts) > 4` guard: `docs/research/data/x.md` is not skipped.
   - Pinned by the same subtest, row `docs/research/data/x.md`, which is in the expected breaks.
5. `data/` outside `docs/research` is not skipped.
   - Pinned by the same subtest, row `docs/other/data/x.md`.
6. A `data/` folder directly under a topic (`docs/research/topic/data/x.md`) is skipped.
   - This is real predicate behaviour: `parts[3:4]` is `["data"]`, so it returns true.
   - No row covers it. Finding 1 below.
7. The last path element is excluded from the `data` match (`parts[3:len(parts)-1]`).
   - No row covers it, for example `docs/research/topic/x/data` as a file. Finding 1 below.
8. `vocabScanAllowlist` entries for `docs/logging-inventory.md` and `docs/reference/weaving-glossary.md`.
   - Pinned by `TestEnforcement_FabricVocabulary`, which runs `fabricVocabularyFailures` over the real tree. Both files name weft/warp (`docs/logging-inventory.md:5-6`, `docs/reference/weaving-glossary.md:34-72`).
   - Dropping either entry makes the scan report a bare weft/warp token, so it would fail.
   - Comparison of the `vocabScanAllowlisted` exact-match path against these two entries is pinned by the same real-tree scan.
9. Go comment-only path edits (`internal/batcher/doc.go`, `internal/lenses/*`, `internal/shuttle*`, `internal/shuttlecli/smoke_guardrail_test.go`).
   - They add no behaviour, so no test is required.
10. The research scripts now use `git -C "$HERE" rev-parse --show-toplevel`, and `turn-savings-measure/run-all.sh` and `review-turn-audit-2026-11-01/run-all.sh` changed their staging.
    - These are measurement scripts outside the production surface, with no test harness in the repository.
    - I raise nothing: no existing test pattern covers them, and the verify gate cannot run them (they need local transcripts).

**Findings**

1. **LOW**, class `design`, `internal/lymcwd/docslink_test.go` (the changed subtest, hunk after line 565): the subtest covers depth 3 and depth 4 but has no row for a `data/` folder directly under a topic folder, or for a file named `data` at the last position. Add a row, or state the intended result, for `docs/research/topic/data/x.md`. The predicate skips it today (`parts[3:len(parts)-1]` is `["data"]`), and nothing pins that choice. Evidence: predicate in the diff, `docsLinkIsResearchData`, and the subtest's file map.

No other behaviour lacks coverage.

**Tool calls:** none. The inherited diff of `docslink_test.go` and `enforcement_test.go`, and my earlier greps, already held everything this lens needed.
