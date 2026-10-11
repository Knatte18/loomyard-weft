# Card 9 — The StrayInputReader capability and Claude's stray-character judgment

**Create:**
- `internal/shuttleengine#StrayInputReader`
- `internal/shuttleengine/claudeengine#Claude.StrayInput`

**Edit:**
- `internal/shuttleengine/claudeengine/session_test.go`
- `cmd/lyx/testdata/test-budget.yaml`
- `internal/shuttleengine/claudeengine/doc.go`

**Intent:**
The first clause of D8: `claudeengine` alone decides whether a box's text is a stray-character candidate (`PATTERN-shuttle-provider-seam`).
`StrayInputReader` is an optional capability beside `Engine` with one method, `StrayInput(capture string) (text string, ok bool)`: ok is true when capture shows the provider idle but for an input box holding exactly one stray-character candidate, and text is that character.
`Claude.StrayInput` implements it: a candidate is exactly one character that is neither a letter, a digit nor whitespace, in a box located as `IdleSession` locates it, with no running turn on screen.
The lone `%` keeps its own rule and reads empty, so it is never a candidate.
Nothing calls `StrayInput` yet, so no send changes.
`claudeengine/doc.go`'s box-reading paragraph states the judgment beside the lone `%` rule.

Tests:
`StrayInput`'s candidate table is a new top-level test in `claudeengine/session_test.go` (`+`, `-`, `>`, a letter, a digit, `%`, two characters, a running turn), which raises `internal/shuttleengine/claudeengine`'s row in `cmd/lyx/testdata/test-budget.yaml` by one.

**ImpactSummary:** a new method and interface with no caller until card 10; the edits are a test, its budget row and a doc paragraph, so no behavior changes.
