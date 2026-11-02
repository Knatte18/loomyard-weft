```yaml
lens: fix-consistency
outcome: reported
```

The change set has two stale twins, both LOW.

1. **LOW** — `internal/gatecli/doc.go:387` still states the old order.
   - It says the runner "waits on it, marks its record terminal, rewrites `tree.json` and runs `runRequest` again".
   - F5 moved `store.markTerminal` in `fixAndRerun` to after the re-run, so the module doc now contradicts the code.
   - The same sentence has no mention of F4: a non-zero exit with empty stderr no longer reaches a fixer.

2. **LOW** — the `waitOnly` doc in `internal/gatefixer/rowgate.go` says "a failing one always does" mark the record terminal.
   - In `waitThenRun`, only `result.Passed && !markOnPass` skips the mark.
   - A Pending or Terminal result also marks the record terminal when `markOnPass` is false.
   - The comment understates this, and a Pending result from an earlier entry loses the live record, which is the premise of F2.
   - Either narrow the skip to Passed or Pending, or reword the doc.

Checked and consistent:
- The `cardGateKey` replacement has no other bare `"card"` key check in `config.go` or `tiers.go`; the `goReadGates` list is a list, not a key check.
- `gateslot/doc.go:80` still says `LoadConfig` refuses a fixer timeout below 1, which backs F8's deletion.
- All three goto sites go through `gotoGuardEndingFixers`, and `fixerendguard_test.go` still calls `fixerEndingGuard` directly.
- The other `"verify"` literals live in `shedrecipe` and `loomrecipe`, which name the gate element, not the `gateFixerBodies` key.

Calls made:
1. `grep` over the `doc.go` files of `gatecli`, `gatefixer`, `loomcli` and `gateslot` for terminal-marking and fixer-timeout text, plus `"verify"` literals in `loomcli`, `loomrecipe` and `shedrecipe`. My context held no module docs or recipe files.
