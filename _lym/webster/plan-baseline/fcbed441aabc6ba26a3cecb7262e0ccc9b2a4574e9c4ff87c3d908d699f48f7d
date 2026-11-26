# Card 1 — config-key-name

**Create:**
- `internal/configengine#KeyName`
- `internal/configengine_test#TestKeyName`

**Extends:**
- `internal/configengine/doc.go`
- `PATTERN.md`

**Intent:** Add the one helper D3 names for rendering a config key in full.
`configengine.KeyName(module, path)` returns `<module>.<dotted path>`, such as `loom.code.max_bounces` from `"loom"` and `"code.max_bounces"`.
It takes the module's name constant and the key's path constant; it never reads a template, so it answers for any module.
Every later card that renders a key name in a message calls it.

`TestKeyName` is a tier0 table test with rows for a top-level key (`drift`, `judge_ask_parent` → `drift.judge_ask_parent`) and a nested path (`loom`, `discussion.review_ask_parent` → `loom.discussion.review_ask_parent`).

`doc.go` gains one sentence naming `KeyName` as the renderer of a key's full name.

`PATTERN.md` gains the declared `adds` entry under `## CLI and config`, directly after `PATTERN-config-layers`, as this whole line:

```markdown
- `PATTERN-config-key-names` — Naming a config key in a message, comment, doc or stencil: in full, `<module>.<dotted path>`, rendered in Go by `configengine`'s one key-name helper from named constants.
```

**ImpactSummary:** One new exported helper in configengine with no caller yet, a one-sentence doc addition and one new PATTERN.md entry line.
