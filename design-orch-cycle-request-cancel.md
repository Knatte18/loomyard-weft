# Withdraw a pending orch distill or refresh request

`orch-cycle-request-cancel` · Note · enhancement, orch

## Seen (orch, 2026-10-21)

The orch ran `lyx orch distill` while a hub pause stood.
The watcher starts no context cycle under a hub pause, so the request sat pending until the resume.
The operator wanted it withdrawn, and no verb does that: the orch deleted the marker `.lyx/orch/cycle-request` by hand.

## Idea

A verb that withdraws a pending `refresh` or `distill` request, such as `lyx orch distill --cancel` or `lyx orch cancel-cycle`, refusing once the cycle has started.
`lyx orch status` could also show a pending request, and `distill`/`refresh` could say when a hub pause holds the request back.
