```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix-code-correctness lens over the F1–F5 test changes.

No findings.

I ran each added test case by hand against the production code in my context:

- **F1:** The path walk in `rejectUnknownKeys` builds `the proposal.parts[0].pattern_clusters[0]`, so the new row's `parts[0].pattern_clusters[0]` match holds. A mistyped tag or a broken nested walk fails either that row or the `DeepEqual` in `TestParse_PatternClusters`.
- **F2:** The loop now also reads the project record's file for the project shape. The tier 0 gate passes, so the file exists and holds the live bullet and no `PATTERN-a`. Dropping the mapping from the project record would fail it.
- **F3:** The `--from` row rebuilds `input` with `by` and the file, so `tc.pattern` is false and the no-`pattern_change` assertions still hold. Dropping `pattern_action` in `decisionEntries` would leave `WithdrawnPattern` empty and fail the row.
- **F4:**
  - The row with the PATTERN changes section ahead of Decisions takes the branch where the bullet is struck at `patternAt < at`.
  - The row whose bullet is the last line with no newline would panic or mis-slice if the `patternEnd < len(content)` guard went. Its expected text is the record plus `## PATTERN changes\n\n`.
  - The fenced `Withdraws:` line is inside the entry's span, since the span runs to the closing fence. Removing the `!fenced[i]` check would exempt the ID and fail the row.
- **F5:** `os.Remove` runs after `got` is computed and the PATTERN.md write succeeded, so it cannot fail on a missing file. A `raddleBoardContext` that treated the missing file as an error would fail the new check.

No signature or caller changed, since every edit is in a test file. I also saw no regression in either gate run: tier 0 (2252 passed, 0 failed) and tier 1 for `loomcli` (149 passed, 0 failed).

Tool calls made: none. My context already held every changed file and the gate results.
