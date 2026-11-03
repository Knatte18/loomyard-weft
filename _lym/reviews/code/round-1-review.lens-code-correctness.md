```yaml
lens: code-correctness
outcome: reported
```

**Lens list (code-correctness)**

- `store_test.go` `ShippedEntriesArePinned`, `strings.Cut(got.Text, "\n")` on `lym-batching`: the first line is `## Batch your calls`, which equals `wantHeading`. Correct. `strings` is already imported at line 8, so it compiles.
- Same subtest on `lym-plan-edits-before-gate`: the first line is `## Make every round of edits in this order`. Correct.
- Same subtest, boundary case of empty text: `Cut` returns an empty heading, which mismatches and reports an error. `ValidateEntry` presumably bars empty text anyway. Correct.
- Same subtest, `len(ordered) != len(tests)` guard and slug/target check: unchanged logic. Correct.
- `shipped.json` JSON validity: both texts are escaped strings with `\n` only and no stray quotes or control characters. `decodeShipped` would still decode them. Correct.
- `fileText` render path, unchanged (`render.go:235-249`), run on the new text: `strings.Trim(entry.Text, "\n")` leaves the leading `## ` heading intact. The joined file is `# LoomYard's own instructions`, a blank line, `## Batch your calls`, then the bullets. There is no level skip. Correct.
- Markdown shape of `lym-plan-edits-before-gate`: the numbered list's 3-space-indented sub-bullets nest under their step. Step 4's colon sub-bullets nest too. The blank line before the closing `- Never…` bullets starts a separate bullet list after the numbered list. Correct.
- `render_test.go` `claudeFileWant` and `TestShowAndPassagesReadTheShippedLayerFirst`: both build their expectation from `shippedStore` or a substituted layer, so they follow the new text unchanged. Correct.
- Meaning of the `lym-batching` bullets against the old paragraph: every instruction survives. "Follow the rule above over the Read tool's own advice" now covers only the one preceding bullet by position; the never-grep and never-slice rules follow it as separate bullets. The override still applies, so this is not a defect.
- `lym-plan-edits-before-gate` step 4: "fix everything its report shows the same way" followed by the read, decide, edit and rerun sub-bullets keeps the old meaning. Correct.
- `doc.go` comment: it is prose only and describes the render accurately. Correct.

**Findings:** none.

**Tool calls**
1. `grep -n '"strings"' internal/heddleengine/store_test.go`: the diff did not show the import block, and `strings.Cut` needs `strings` imported.
