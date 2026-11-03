```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the search behind each finding whose premise is a set, to check the fix reaches every member.

Unchecked findings: none. F1 and F2 are the set-premise findings, and I reran both searches. F4 names a single site. F3 and F5 are decision-class findings, so they are not part of this fix's set.

no findings

- **F1:** Of the files outside `_lym`, only `internal/boardcli/cli.go` spells `on_land?` in a `Use` token, and that token belongs to the `upsert` command, which card 1 now targets through `storeVerbs`. The `upsert-batch` and `merge` tokens nest `&lt;task&gt;`, which the test maps to `upsertPayload` (`cli.go:458-471`), so neither token needs an edit. Card 5 lists `Command` for the `rename` help and `storeVerbs` for `getCmd`, which matches where each command is built.
- **F2:** The cards no longer mention a `pastApproval` or `approvalRow` error branch, an `approvalErr` field, or an "with no error" fake answer. What remains: the signatures in card 8:33 and card 9:48, the error-drop comments in card 8:38 and card 9:49, and plain bool uses.

Tool calls:
1. Grepped `upsertPayload` in boardcli and the boardcli targets in the cards, to rerun F1's Use-token and symbol search. My context did not include `upsertPayload`'s definition.
2. Grepped `on_land?` across the repository and the cards' `pastApproval`, `approvalRow`, `approvalErr` and "no error" mentions, to rerun F1's token-spelling search and F2's error-mention search over every card. My context only showed the spans I had edited.
