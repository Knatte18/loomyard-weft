# Credit Claude Code, not Claude, in the landing credit block

The landing credit block now lists `Claude Code <noreply@anthropic.com>` for a task in which a Claude session ran, since the engine is Claude Code.
Only the shipped default of the `claude` item of `credit_engines` in `internal/landingshed/template.yaml` changes.
`internal/landingshed/config_test.go` pins the new text in the default-config check and in the well-formed config its invalid-key cases start from.

## Design decisions

- No `hubreconcile` reinterpretation is shipped: the key's meaning is unchanged, only the text of a default.
- No hub layer keeps the old text: the hub's `landing.yaml` base file holds no `credit` or `credit_engines` key, so every hub reads the new default.
- No trailer of any kind is added; the credit block stays the one place the engine is credited.
