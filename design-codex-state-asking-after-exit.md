# lym shuttle state keeps a Codex run asking after Codex exits

`codex-state-asking-after-exit` · Note · bug, shuttle, codex

An interactive Codex run held at its composer reads `asking/awaiting-input` in `lym shuttle state`.
After `/exit` in its pane the run ended `died`, and the shadow state model logged `shuttle: session state disagrees ... loop=died state=asking cause=awaiting-input` in the same second.
The state of a Codex session follows hook signals alone and Codex exit fires none, so nothing in the run files reads as dead.

Found by the pane/load trial, docs/research/codex-engine-trial-pane-load-2026-11-03.md case 3.