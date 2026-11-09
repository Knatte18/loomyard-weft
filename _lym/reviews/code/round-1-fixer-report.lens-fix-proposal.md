```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the four proposals (F1–F4) the fixer carried out as proposed.

Findings: none that meet the rubric.

Checks, from the context I already hold:
- **F1 (CLI rows):** The rollback row's `wantFile: seeded` and `wantErr: run profile "light"` hold. The gate passed 707 of 707, so both rows pass against the real load check. The rows use `fakeHubCommit` and `fakeSyncTracker`, which the existing rows already use, so there is no new isolation or parallelism issue. The first dotted row pins the exact output, `plan_max_bounces: 2`. Block style was kept, and the row passes.
- **F2 (yamlengine rows):** One proposed expectation was wrong. The `on: true` output is unquoted, and the fixer corrected the row to match. The comment-keeping row (`writer: opus # the writer`) and the null-parent row pass, so the branches are pinned. The malformed-value row asserts only the key and `double-quote`, which is a loose match but names the intended refusal. The `{a: 1` case could in principle fail at a different error path, but the gate shows it reaches `openMapEntryNode`'s refusal text.
- **F3 (JSONEntries comment):** The reworded two-line comment is accurate. It keeps the "no template key check" claim, which holds for both forms. The line lengths stay under 120.
- **F4 (dot note):** The sentence was added to the `SetEntries` doc comment and to `doc.go`. It sits after the "keep their order and comments" sentence and before the error sentence, so it breaks no sibling text. The claim matches the code: `strings.Contains(pair.Key, ".")` always routes to the path form.
- **Proposed change breaking elsewhere:** The `JSONEntries` comment is the only text on this field, and no other doc, such as `configengine/doc.go` or `yamlengine/doc.go`, was shown to contradict the new sentence. I did not grep them for the old phrasing "whole top-level entry". `configengine/set.go` is named in the gate's change list, and the review said its prose was already updated.

No further tool calls made.
