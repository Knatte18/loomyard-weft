# Card 11 — tmux-tier-green

**Edit:**
- `internal/reedcli/smoke_dotfill_test.go`

**Intent:**
Decision D4 makes Publish gate on the `tmux` tier, so the tier has to pass on a clean tree first.
`TestSmokeDotFill`'s `CrossClientControl` subtest fails with tmux's "no space for new pane" when it splits the harness window: the harness window is too small for the split it asks for.
Give the harness the explicit window size the split needs, the way the other subtests pin theirs, so the subtest passes on any terminal.

If `go test -tags tmux ./...` shows any other failure on the clean tree, this card fixes it too; the card that sets loomyard's `publish_verify` (card 12) comes after this one, so no commit of the task sets the value while the tier fails.
This is a test-environment fix with no production change, so the subtest itself is its coverage, run by this card's `Verify`, which the card gate does not run.

**ImpactSummary:** One `tmux`-tier test file; no production code.

**Verify:** go test -tags tmux ./...
