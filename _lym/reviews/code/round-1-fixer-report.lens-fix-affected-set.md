```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Unchecked findings: none. F2, F3 and F4 are coverage gaps in `yamlengine`, not sets under one key.

**F1** (the only finding whose premise is a set, the open maps with no declared shape): I reran its search, `OpenMaps:` and `OpenMapShapes` across non-test Go under `internal`. Only `gate` and `drift` now declare shapes. Every other module's open map decodes into something other than a struct-valued entry, so none has the same gap:
- `darn`'s `tree_scope` is a `map[string]string`.
- `reed`'s `buttons.*` decode into a `yaml.Node`.
- `board`'s `types` and `labels` are lists of names.
- `landing`'s `credit_engines`, the `loom` and `tapestry` model and fan keys, and `claude`'s `*_roles` were not checked beyond their names. They hold scalars or lists, and the paths the fix covers do not touch them.

Tool calls:
1. `grep OpenMaps:|OpenMapShapes` over `internal` (non-test). My context did not hold the full list of modules declaring open maps.
2. `grep -A6 func ConfigOpenMaps` for darn, landing, claude, reed, loom and board. My context did not show their open-map keys.
3. `grep` on `tree_scope`, `creditEnginesKey`, `Buttons` and `types|labels`. My context did not show these keys' decode types.
