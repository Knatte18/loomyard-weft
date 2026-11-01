```yaml
lens: fix-proposal
outcome: reported
```

fix-proposal lens: I challenged the proposal behind F1, which the fixer carried out as proposed in cards 4 and 8.

no findings

- **The guard after the exemption is dropped.** `internal/fabricengine/livestate_doc_test.go` has one `_board` hit, at line 189, and it sits in the header comment. Card 4 now rewords that hit, so card 8's guard can drop the exemption without failing on this file. No assertion reads that text, so card 4's line "No assertion changes meaning" still holds.
- **The Prosa group.** The file declares no symbol, so no member glyph can name it. `validate-plan --overlay` returned `ok: true` with the Prosa group in place.
- **Losing the verbatim quote.** The rewording means the line no longer quotes the earlier run's error text exactly. The decision record's entry "No _board literal stays in livestate_doc_test.go" chooses that on purpose, so this is not a defect.
- **Card order.** Card 4 lands before card 8, which matches card 8's note that its guard passes "only after cards 2 through 6".

Outside this lens: `internal/fabricengine/livestate_refusal_selftest_test.go:307` has a `_board` comment ("no _board entry and no weft sibling"). The guard would catch it unless card 4 lists the member that holds it. That set belongs to the fix-affected-set lens.

Tool calls:
- One grep for `_board` in `livestate_doc_test.go` and for references to that text elsewhere. My context showed only lines 1–10 and 180–195 of the file, so it could not confirm line 189 was the only hit.
