# Card 11 — llm-smoke-strand

**Edit:**
- `internal/creelcli#smokeHub.addStrand`
- `internal/creelcli#TestSmokeCreelClaudeStrand`

**Uses:**
- `internal/creelcli#smokeHub.awaitPane`
- `internal/creelcli#smokePaneWindow`
- `internal/creelcli#wireShuttle`
- `internal/creelcli#hubDelivery.Idle`
- `plan:internal/shuttleengine#Runner.AcceptStartupGate`

**Intent:** The test half of #678 (D9): the llm smoke test prints its pane, places its strand in its own window and gets past the folder-trust gate.

`smokeHub.addStrand` passes `--anchor own-window` to `lyx reed add`, the #673 workaround CLAUDE.md requires for every `reed add` with no `--parent`, so the strand is not split into the selvage shell window and drawn too short; the tmux tier's smoke test shares the helper and gets the same placement.
`TestSmokeCreelClaudeStrand` stays a hand-added `lyx reed add --cmd claude` strand.
Before its idle wait it polls the claude pane, under `smokePaneWindow`, until the pane shows its input box or a startup gate: each tick builds the prime worktree's runner through `wireShuttle` over `shuttleengines.New`, like the test's other shuttle calls, and calls `Runner.AcceptStartupGate` on the strand, so a folder-trust gate on the fresh temp hub is accepted through the engine-neutral seam and `internal/creelcli` never imports `claudeengine` (`PATTERN-shuttle-provider-seam`).
When the idle wait gives up, the failure prints the strand's last pane capture, so a third cause is readable.
Accepting the gate adds the temp hub's path to the operator's Claude trust list, for an llm run the operator starts by hand.

No gate runs the `llm` tier (`PATTERN-test-speed`): the card's check is that the tier compiles, and the operator proves the pass by hand with `lyx gate test --tags llm ./internal/creelcli`.

**ImpactSummary:** Only the two smoke tiers' shared strand helper and the llm smoke test change, both under build tags no automatic gate runs; production code is untouched.

**Verify:** go vet -tags llm ./internal/creelcli && go vet -tags tmux ./internal/creelcli
