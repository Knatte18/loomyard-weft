```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing per changed behaviour the test that pins it.

**Per-behaviour list**

| Behaviour the diff changes | Pinning test | Verdict |
|---|---|---|
| Each shipped entry's text opens with its own `##` heading, so the rendered file steps from `# LoomYard's own instructions` with no level skip. | `TestStore/ShippedEntriesArePinned` (`store_test.go:170-194`). It compares the first line of each shipped entry's text with `wantHeading`, so removing or changing a heading fails it. | Pinned. |
| The shipped layer is still exactly the two entries, in order, each aimed at `claude-file` alone. | The same subtest, through the length check and the `Slug`/`Targets` check (`store_test.go:~183-190`). | Pinned, unchanged. |
| The bullet and numbered-list text of both entries. | None. | Exempt: static prose. The old exact-text pin of `lym-batching` was dropped, and the Goal asks only that tests follow the new text. |
| The renderer joins the shipped heading and entry texts. | `TestShowAndPassagesReadTheShippedLayerFirst` (`render_test.go:166-219`) and `claudeFileWant` (`render_test.go:72-88`) pin the join. | No renderer change in the diff. These tests build their expectation from `shippedStore` or from injected layers, so they follow the new text unchanged. |
| The `doc.go` sentence about the level-two headings. | None. | Exempt: documentation. |

**Findings**

None at BLOCKING, MEDIUM or LOW.

Two notes, neither raised as a finding:
- **NIT, class `consistency`:** `store_test.go:178-179` pins the exact heading strings (`## Batch your calls`, `## Make every round of edits in this order`). That is a literal assertion on prose, so a heading reword forces a test edit. The structural contract it protects is only "first line is a `##` heading". It is not a stencil assertion in the rubric's sense, so I do not count it as a defect.
- **Coverage not added:** nothing checks that an entry contains no `#` or `###` line inside its text. The heading-only pin cannot catch a later level skip inside an entry. I do not count this as a coverage gap for the diff: the diff adds no such line, and the Goal asks only for the heading structure.

I made no tool calls, because my inherited context already held the diff, `store_test.go`, `render_test.go`, `render.go` and `shipped.go`.
