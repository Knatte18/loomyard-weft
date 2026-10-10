---
round: 3
focus:
  - "Card 5 (`_lyx/plan/05-tmuxkit-hermetic-server.md`) pre-starts a tmux server in tmuxkit's `Socket`/`KillOnCleanup`, while reed's `spawnSession` in `internal/reedengine/lifecycle.go` otherwise starts it with `cmd.Dir` at the hub logs dir, the global `-v` debug args and a Claude-scrubbed environment. Settle whether the plan states what the pre-start keeps or drops of those properties, and whether every test card 6 registers before a boot (for example one asserting on `tmux-server-*.log`) still holds."
  - "Cards 6 and 10 (`_lyx/plan/06-tmuxkit-package-server.md`, `_lyx/plan/10-tmuxkit-socket-check.md`) name a net for a missed registration or leftover server. Settle which of the plan's Verify lines actually run tmux-tier and llm-tier tests rather than compile them with `-run '^$'`, and whether each net the Intent claims is among them."
  - "Card 10's `checkSockets` lists socket files after `m.Run`, and reed's `ensureServerGoneLocked` removes its own socket file on teardown. Settle whether a test that boots and tears down a reed server still fails the check in tier 2, and whether the plan states the bound where it does not."
  - "Card 13 (`_lyx/plan/13-gatecli-prebuild.md`) bounds a forged `LYX_PREBUILT_LYX` to ungated runs, and `internal/verifytree.Verify` passes `lease.Env(os.Environ())` to the plan-level verify command. Settle whether every run that executes tests inherits a stripped or freshly built value, and whether the card's nested-module sentence agrees with `buildRoot`."
  - "Card 31 (`_lyx/plan/31-tierpurity-tmux-constraint.md`) classifies files by build constraint. Settle whether a file whose constraint is true with only `integration` set, such as `integration || tmux`, can reach a tmuxkit spawn under the scan's rule."
  - "Reed boots after a test's own `down` or `kill-server` (D4: the operator's `~/.tmux.conf` is never read by a test server). Settle whether the plan handles or bounds the server reed restarts without `-f`, and whether card 7 and card 8 agree on which probe copies stay in loomcli and orchcli."
---

Entries restate the sites they depend on and pre-state no verdict.
