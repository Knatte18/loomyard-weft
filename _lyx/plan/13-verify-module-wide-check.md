# Card 13 — verify-module-wide-check

**Create:**
- `plan:internal/planparser#checkVerifyModuleWide` -> `func checkVerifyModuleWide(plan *Plan) []ValidationError`

**Edit:**
- `internal/planparser#validate`
- `internal/planparser/testdata/goodplan/01-json-row-type.md`
- `internal/planparser/parse_test.go`
- `internal/planparser/validate_test.go`
- `internal/loomcli/parity_test.go`
- `contracts/specs/loom-plan-spec.md`
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/claims_test.go`

**Uses:**
- `internal/planparser#goCommandDirAndPackages`
- `internal/planparser#checkVerifyNestedModule`

**Intent:**
Add the blocking plan check `verify-module-wide` (D9): a card's `**Verify:**` command that D7's deny would refuse can never run under a fork, so a plan holding one fails its batch by construction, and the plan gate is where that is cheapest to catch.
`checkVerifyModuleWide` splits each card's Verify chain the way `crossModuleCommands` does and refuses a `go test`, `go build` or `go vet` (after any `go -C <dir>`) with a package argument holding `...` or equal to `all`, and a `go test` whose `-tags` value names `tmux` or `llm`; one finding per command, attributed to the card, naming the command and the way forward, `lyx gate test [-C <module>] [--tags <tags>] <packages>`, run as a background Bash call.
It reads the overview's `## verify:` section never, since Go runs that command itself.
It runs in `validate`'s shared list after `verify-nested-module`, so both plan gates and `lyx loom validate-plan` call the same function (`PATTERN-gate-self-check-parity`); the file comment's check list gains it.
Bound: it matches D7's static patterns and inherits their bound: a command built from a variable or behind `bash -c` passes.
The golden fixture's card 1 Verify becomes `go test ./internal/boardcli`, and `parse_test.go`'s two assertions of that value follow it.
`validate_test.go` gains table rows refusing `go test ./...`, `go -C sub build ./...`, `go vet all` and `go test -tags tmux ./internal/x`, and passing `go test ./internal/x`; `TestGateParity_PlanGate` gains a row whose card Verify is `go test ./...`, reported by both the gate and the verb.
`loom-plan-spec.md` gains the check's row in the validation table, and its worked example's card 1 Verify matches the fixture.
`loom-template-plan.md`'s Verify guidance states that a card's `**Verify:**` is package-scoped and uses `lyx gate test` for a module-wide or `tmux` run (D8), with `claims_test.go` pinning that sentence.
Any other test fixture that validates a plan whose card Verify is module-wide (such as the `go test ./...` Verify in `internal/websterengine/recordbatch_test.go`, if its path validates) is changed to a package-scoped one.

**ImpactSummary:** One new check in planparser's shared validation, reported by both plan gates, `lyx loom validate-plan` and dispatch; existing fixtures with module-wide card Verify lines are rewritten.
