```yaml
lens: fix-affected-set
outcome: reported
```

Findings: none.

- **F3** (set: every site of the old steer wording). `grep -rIl "otherwise parse JSON with jq"` over the repo returns nothing, so every site now carries the new wording: `rows.go` and all eight `denyPython=true` settings in `settings-golden.json`. The notices never carried the clause. No other engine's golden or doc (including `codexengine`) held the old text.
- **F1** (set: python phrase assertions in tests). The removed loop was the only one; the remaining sites of the wording are `internal/shuttleengine/template.yaml:14`, `internal/shellguard/doc.go:22` and `rows.go:79`. Of those, `doc.go` and `rows.go` are untouched by the fix. I did not read `template.yaml:14` past its first 140 characters, so whether it repeats the "otherwise" binding is unchecked.
- **F2** (set: none; a single `wants` entry). Left unchecked.

Calls made:
1. `grep -rIl "otherwise parse JSON with jq" .`, to rerun F3's search for any site of the old wording.
2. `grep -rIn "never work round the guard\|differs from machine to machine" ...`, to find any other test or doc pinning the python phrases, which F1's removed loop asserted.
