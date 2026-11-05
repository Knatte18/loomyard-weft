```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: checked whether the three fixes left a stale twin behind.

**1. MEDIUM — F1 fix left a twin on line 5.**
- Changed text: `docs/research/twill/twill-discussion-bench/deviations.md`, line 5.
- Summary: line 5 still says `see twill-discussion-bench.md "Deviations, round 4"`. The file was renamed to `report.md`, so the reference points at nothing.
- Evidence: the fixer's `replace_all` matched only the backticked form `` `twill-discussion-bench.md` `` (line 1). Line 5 has the name unquoted, so it was not replaced. The grep `grep -rn "twill-discussion-bench\.md" docs internal` returns only that line.
- Target check: `report.md:453` holds the heading `## Deviations, round 4`, so the fix is to replace the name on line 5 with `report.md`. The review's F1 named line 5 explicitly, so this is also a finding the fix did not fully resolve.

**2. No finding — F2.** Line 13 of `COMMANDS.md` now agrees with line 101. A grep for `research/glyph-audit-2026-10-1[35]` over `docs` and `internal` finds no remaining old-path reference.

**3. No finding — F3.** The predicate and its test rows agree.
- The doc comment on `docsLinkIsResearchData` was updated to match the new bound.
- The file header (line 2) and the comments at lines 370 and 418 say "data/ directory of a study", which still holds.
- The new `docs/research/topic/data/x.md` row is in the expected breaks, in sorted order.
- No existing `data/` folder sits directly under a topic: the earlier `find` showed only study-level ones.

Further tool calls:
1. grep for `twill-discussion-bench\.md`, old glyph paths and the `data` predicate wording over `docs` and `internal`: my context did not hold repo-wide twin references.
2. grep for `Deviations, round 4` in `report.md` and `sed -n 1,2p` of `docslink_test.go`: my context lacked the target heading and the file header text.
