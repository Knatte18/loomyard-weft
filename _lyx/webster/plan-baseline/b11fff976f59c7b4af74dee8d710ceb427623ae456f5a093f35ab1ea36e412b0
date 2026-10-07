# Card 11 — tmux-tier-green

**Edit:**
- `internal/reedcli/smoke_dotfill_test.go`
- `internal/reedcli/smoke_test.go`
- `internal/reedcli/smoke_lifecycle_test.go`

**Intent:**
Decision D4 makes Publish gate on the `tmux` tier, so the tier has to pass on a clean tree first.
`docs/benchmarks/test-suite-timing.md` records a failure of the tier at revision `04d9b57c0`, before the prune folded the four dot-fill tests into the `TestSmokeDotFill` scenario: a top-level `TestSmokeDotFillCrossClientControl`, which tmux refused with "no space for new pane".
The document names no call site, only the message.
The test is now the `CrossClientControl` step of `TestSmokeDotFill`.

The harness window is not the cause: `newDotFillHarnessOn` creates the window with `-x`/`-y`, every caller passes 140 by 42, and it sets `window-size manual`.
The test's own `split-window` reports only `exit status 1`, so the message cannot have come from it; the text surfaces through reed's own splits, which the shared helpers `restartSession` (`smoke_lifecycle_test.go`) and `addStrandIn` (`smoke_test.go`) drive and whose output they print.

The implementer first runs `go test -tags tmux ./...` on the clean tree, and the card ends in one of three ways:

- The tier passes: the card changes nothing and its report says so, with the run's output, and card 12 follows.
- A test of the tier fails: the implementer captures the failing output (reed's output or tmux's stderr) before choosing a site, and fixes the cause in the test code, so the test passes on any terminal.
  The three listed files are the candidates, and a file is edited only where the captured output shows the cause there.
- The cause is outside those files, reed production code included: the card is not widened into it.
  It ends unfinished with a report of the file and the output, so the run halts at this card and card 12 is not started until the tier passes.

Card 12, which sets loomyard's `publish_verify`, comes after this one, so no commit of the task sets the value while the tier fails.
The card changes no production code, so the run of the tier is its coverage, and it is this card's `Verify`, which the card gate does not run.

**ImpactSummary:** At most the three `internal/reedcli` `tmux`-tier test files change, and none when the tier already passes; no production code.

**Verify:** go test -tags tmux ./...
