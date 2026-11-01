```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: checked every claim the fix added to cards 4 and 8 against the code at HEAD.

no findings

- Card 4's new bullet says `livestate_doc_test.go` declares nothing. That holds: the file has no `func`, `type`, `var`, `const` or `import` line, only a header comment above `package`.
- The same bullet says the file has one hit, at line 189. That holds: `_board` appears once as a whole word, at line 189, inside the quoted error text "no _board entry and no weft sibling".
- The bullet says that after the rewording, card 8's guard needs no exemption for the file. That holds: the reworded text "no &lt;Bolt directory&gt; entry and no weft sibling" contains no `_board` token, and line 189 is the file's only hit.
- The new `**Prosa:**` group names the file by its path. Before the forks were spawned, `lym loom validate-plan --overlay` with `{}` returned `ok: true` and only informational findings.
- Card 8's rewritten exemption sentence only drops the third exemption. Its two remaining claims, about `anchor.go` and `enforcement_test.go`, are carried over unchanged.

Tool calls:
- One grep over `internal/fabricengine/livestate_doc_test.go` for the `_board` token and for top-level declarations. My context showed only lines 1–10 and 180–195 of the file, which cannot show that it has a single hit and no declarations.
