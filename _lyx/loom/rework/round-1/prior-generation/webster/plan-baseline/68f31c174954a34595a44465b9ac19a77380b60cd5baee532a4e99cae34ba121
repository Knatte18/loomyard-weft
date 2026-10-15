# Card 15 — creelcli-smoke

**Create:**
- `internal/creelcli/smoke_tmux_test.go`
- `internal/creelcli/smoke_llm_test.go`
- `internal/creelcli#TestSmokeCreelFakeProviderPane`
- `internal/creelcli#TestSmokeCreelClaudeStrand`

**Edit:**
- `//cmd/lyx/testdata/test-budget.yaml`

**Intent:** Prove delivery on real panes, the two acceptance criteria no fake reaches.
`smoke_tmux_test.go` (`//go:build tmux`): `TestSmokeCreelFakeProviderPane` builds a hub with `hubforge.CopyHub`, adds a strand through reed whose command is a shell script standing in for a provider (it draws a ready marker and an input box line, echoes each line it reads, and redraws idle), and runs the real daemon in process on the real socket with the hub delivery of card 13 over a `shuttlefake.Engine`-based engine that reads that script's frames; a `post` is acknowledged and its reminder line appears in the pane, a privileged `inject` is typed into the pane and `inject --wait` reports injected, and `log --id` shows the events.
`smoke_llm_test.go` (`//go:build llm`): `TestSmokeCreelClaudeStrand` skips without `llmkit.Claude`, adds a strand with `lyx reed add --cmd claude` in a hub, and asserts that the adapter's idle check reads the booted session idle, that a posted message's reminder and an inject are typed at idle, and that the agent's pane shows both.
The plan's verify compiles both tiers and runs neither; Publish runs the tmux tier.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/creelcli` by two.

**ImpactSummary:** Two tagged smoke tests and one budget raise; no production code changes.
