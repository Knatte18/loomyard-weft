```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1–F5 as the fixer carried them out.

No findings.

I tried to build a case where each proposal is wrong, and none held:

- **F1:** The new unknown-key row asserts the path `parts[0].pattern_clusters[0]`. The key walk builds that path from `parts`, the `[0]` index, `pattern_clusters` and a second `[0]`, so the row exercises the nested-list walk. The new `Parse` test compares `Part.PatternClusters` against the expected values, so a mistyped tag on `sketch_id`, `entry` or `text` would fail it.
- **F2:** Asserting the project record is sound. The `project` shape sets `Project.Slug`, the loop adds that slug to the records it reads, and the tapestry and tasks shapes read only `part-a`. The test passes in the gate.
- **F3:** The `--from` row replaces `input` whole, so `tc.pattern` stays false and the `pattern_change` absence checks still hold. The shared `withdraw` assertions then cover `WithdrawnPattern` and `pattern_withdrawn`, which proves the `decisionEntries` path.
- **F4:** Each new row fails with its branch broken.
  - `SectionBeforeDecisions` takes the `patternAt < at` splice.
  - The no-newline row would slice out of range if the `patternEnd < len(content)` guard were removed.
  - The fenced `Withdraws:` row would yield no finding if the fence skip in `withdrawnPatternIDs` were removed.
- **F5:** The removal and second call sit after the first context is built, and `CopyHub` gives this test its own hub, so no parallel test shares the file. The check is that no error comes back and `LivePattern` answers false for `PATTERN-live`, which pins the missing-file branch.

Further tool calls: none, because my context already held every changed file.
