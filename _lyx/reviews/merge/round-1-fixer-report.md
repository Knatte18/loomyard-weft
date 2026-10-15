## Fixer-report — merge round 1

### Changes

- **F1** (fixed as proposed): added a subtest to `internal/shuttleengine/enginedispatch_test.go` asserting that `Runner.Send` and `Runner.Interrupt` on a live strand with no run record, over an `EngineSet` whose default is unregistered, return `ErrEngineNotRegistered` and reach the pane with no key, text or paste.
  Added an assertion to `assertBadClaudeConfigRefusesOnlyARunStart` in `internal/shuttleengines/shuttleengines_test.go` that a recordless `Send` and `Interrupt` over an invalid `claude.yaml` raise no engine config error.
  `lyx gate test` over both packages is green.

- **Gate finding** (PATTERN.md overview 4 bytes past the headroom line): shortened the `PATTERN-spawn-color` entry from "its spawning module names" to "its spawner names", cutting 7 bytes; its background file keeps the full wording.
  `lyx gate test ./internal/pattern` is green.

### Disputed

None.

### Deferred

None.
