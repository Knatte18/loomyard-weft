---
format: 5
approved: false
language: go
---

# Plan: `lyx orch start --continue`

After a reboot or a dead orchestrator strand, `lyx orch start --continue` brings the same orch conversation back.
Shuttle exposes the provider session id of a run's reading and a started run, orch's state records it at every fresh launch and the watcher keeps it current across `/clear`, `--continue` resumes it through the existing adopt branch, a fresh launch replacing a recorded id names it in its hint, and `lyx orch status` shows it.

## Card Index

1 — shuttle-session-id — the session reading and a started run expose the provider session id
2 — watcher-records-session-id — orch state gains `session_id` and the watcher records the id it reads each tick
3 — start-continue — `start --continue` resumes the recorded id, every fresh launch records its id and names a replaced one in the hint
4 — status-session-id — `status` reports the recorded id, and `stop` is pinned to keep it

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
