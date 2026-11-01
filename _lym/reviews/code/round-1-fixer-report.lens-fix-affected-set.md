```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- F1: the old rules file is named in two places, the `rules` long help and the role's "Orch rules" section, and both now carry the carry-over sentence. The other mentions don't describe the old file as live. They are `orchcli/doc.go`, `orchengine/doc.go`, `PATTERN-heddle-instructions.md`, the role's lines 189, 237 and 239, and the dated research docs. The role's line 239 ("a draft starts as a copy of `lym heddle get orch-rules`") is the premise F1 flagged, and the new sentence beside it covers the case where that entry doesn't exist yet.
- F2: `ChangeStore` has one production caller, `writeOrchRules` (`internal/orchcli/rules.go:85`). Its error, warning and out-of-hub paths now have tests.

Calls made:
1. `grep -rn "orch/rules\|orch rules"` over the Go, markdown and yaml files outside `_lym`, `.lym` and tests. I needed every site that names the old file or the new verb, and my context only held the files the fixer read.
2. `grep -rn "ChangeStore\|changeStore"` over the Go files. I needed to confirm that the callers of the changed function are all covered.
