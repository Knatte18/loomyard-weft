# Card 12 — begin-batch renders each card's gate command

**Create:**
- `internal/planparser/targetdirs.go`
- `internal/planparser/targetdirs_test.go`
- `internal/websterengine/cardgate.go`
- `internal/websterengine/cardgate_test.go`

**Edit:**
- `internal/websterengine/beginbatch.go`
- `internal/websterengine/recoverbatch.go`
- `internal/websterengine/render.go`
- `internal/websterengine/beginbatch_test.go`
- `internal/websterengine/template_test.go`
- `contracts/stencils/webster/webster-body-implementer.md`
- `contracts/stencils/claims_test.go`

**Intent:**
D1: Go derives each card's gate command, replacing the implementer body's "`go build ./...` and this card's package's unit tests".

`planparser.CardTargetDirs(plan *Plan, card Card) []TargetDir` (new file, so the Planparser Sole-Parser and Glyph Conversion Chokepoint invariants hold) returns one entry per worktree-relative directory the card's targets live in:
a path target's directory (the path itself when it names a directory), a glyph's or `plan:` handle's unit mapped through `Glyph.UnitPath` (its directory when the unit is a file), and a Rename or Move pair's New side.
Each `TargetDir` carries `DeleteOnly` (every target in it sits in a `Delete` group) and `NamesGo` (some non-Delete target in it is a `.go` path, a glyph or a handle).
`Uses:` is excluded, since it is read rather than written.

`cardgate.go`'s `cardGateCommand(plan, card, worktree string) string` keeps a directory when it is not `DeleteOnly` and either `NamesGo` or it holds a `.go` file on disk, so a Delete-only directory and a non-Go directory (a stencil or doc folder) drop out, and renders
`go build ./... && go test ./... && go test -tags integration ./<dir> ...`,
omitting the last step when no directory survives.
`BeginBatch` and the recovery spawn build one line per card of the batch (`` - `<card pointer>`: `<command>` ``) and pass it to `RenderForkPrompt` and `RenderRecoveryPrompt` as a new required `card_gates` marker.
Neither render function takes the plan, per the fork-context-hygiene rule.

The implementer body's step 3 becomes: run the card's gate command from `{{.card_gates}}` from the worktree root;
a failure is the card's own gate, fixed before moving on.
The card's own `**Verify:**` still runs after the gate when declared.
The self-fix and report wording that says "build+unit gate" says "gate".

Bound: the fork runs the gate and Go never does, as with today's stencil gate;
a fork that skips it is caught by the webster verify gate (card 15), whose card hint (card 14) localizes the failure.

Tests:
`targetdirs_test.go`: path, member glyph, handle and Rename targets map to their directories, and Delete-only and `NamesGo` flags are set.
`cardgate_test.go` (temp dir): a card's command carries `go test ./...` and `-tags integration` over its package directories, leaves out a Delete-only directory and a directory with no Go files, and drops the integration step when nothing survives.
`beginbatch_test.go`: begin-batch's written fork prompt carries the card's gate command.
`template_test.go` and `claims_test.go`: the body requires `card_gates` and no longer says "this card's package's unit tests".

**ImpactSummary:** every fork and recovery prompt gains a Go-rendered per-card gate; planparser gains one exported read-only helper.
