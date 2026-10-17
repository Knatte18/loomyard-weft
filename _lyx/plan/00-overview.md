---
format: 5
approved: false
language: go
---

# Plan: creel CLI output fixes

Fix three output bugs in `lyx creel` from the creel stage 1 re-test (#699, #700, #701):
`log --id <id>` leaves out the `reminded` events that list the id,
a refusal's stderr fallback prints `creel: ` twice,
and a success envelope that cannot be written is lost with exit 0.
The engine's id filter is one card; the creelcli fixes are two, the prefix first, since the lost-answer card re-signs `writeRefusal` on top of it.

## Card Index

1 — log-id-reminded — `Store.Log`'s id filter also keeps an event whose unread ids hold the id
2 — refusal-prefix-once — `writeRefusal`'s stderr fallback prepends `creel: ` only when the message lacks it
3 — lost-answer-names-done — a success envelope that cannot be written exits 1 with one stderr line naming what the verb already did

## Shared Decisions

- The lost-answer line goes straight to the command's stderr, never through `writeRefusal`, so no error envelope follows a failed success write.
- `internal/output` and the hidden `daemon` verb keep `output.Ok`; creelcli's own `writeOk` carries the checked write.

## verify:

```bash
go build ./cmd/lyx
lyx gate test ./internal/creelengine ./internal/creelcli
lyx gate test --tags integration ./internal/creelengine ./internal/creelcli
```
