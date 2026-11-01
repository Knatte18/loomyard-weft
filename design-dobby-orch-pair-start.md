# Dobby and orch start in pairs: a batten run brought dobby up alone

`dobby-orch-pair-start` · Task · bug, dobby, batten, orch · running · Describe

## Seen (operator and orch, 2026-11-01)

After the hub's tmux server was killed, `lym batten run bolt-dir-rename`, run by hand to finish a teardown, brought the prime's dobby up alone: `ensureParentDobby` in `internal/battencli/cli.go` starts the parent worktree's dobby when it is down, and nothing starts its orch.
`lym a` then found no orch in reed while dobby ran; only `lym s` (`lym beam start`) brought the orch up.
Meanwhile dobby's mail to the orch had no reader.

## Design (operator, 2026-11-01)

Dobby and orch are started in pairs, by beam: every path that brings a worktree's dobby up brings its orch up with it, through `lym beam start`'s way, never dobby alone.
- `ensureParentDobby` and any other lone dobby spawn go through that one start, or stop and name `lym s` instead of spawning.

## Out of scope

Whether dobby, finding its orch gone while it runs, brings it up again belongs to note `dormant-orch`, where the orch lies dormant and is woken when needed (operator, 2026-11-01).
