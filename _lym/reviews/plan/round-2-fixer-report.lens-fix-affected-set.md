```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran F1's search to check that the fix covers every site.

Unchecked findings: none.

no findings

F1's set has two parts, and the fix covers both:
- **Whole-token `_board` literals in `internal/fabricengine/livestate_doc_test.go`:** there is only one, at line 189. Card 4's new bullet rewords it, and the new Prosa group lists the file.
- **Places in the plan that name the file's exemption:** card 8's exemption sentence was the only one. It now names only `internal/lymcwd/anchor.go` and `internal/lymcwd/enforcement_test.go`. None of the plan's card files mentions a `livestate_doc_test.go` exemption anymore.

Tool calls:
- One `grep` over `internal/fabricengine/livestate_doc_test.go` and the plan card files `_lym/plan/0*.md`, `1*.md` and `2*.md`. My context showed only lines 1–10 and 180–195 of the test file, so I could not otherwise confirm line 189 is the only hit. My context's earlier plan grep came before the edits, so it could not show the cards in their changed form.
