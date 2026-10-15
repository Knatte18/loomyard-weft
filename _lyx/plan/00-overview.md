---
format: 5
approved: false
language: go
---

# Plan: engine-usage-readers

`tools/tokencount` stops parsing Claude Code's transcripts itself and reads token use, time and cost through the shuttle engine that recorded each session.
`shuttleengine` declares a neutral locator and reader, a per-message usage record and a pricing capability; `claudeengine` implements them, sharing one transcript parse with its `SessionUsage`; `shuttleengines` hands them out offline behind adapters; tokencount and its calibration then read only through them, so tokencount holds no transcript layout, line format, price table or instruction-file name.

## Card Index

1 — neutral-transcript-types — declare the neutral transcript, usage, pricing and read-set types in shuttleengine
2 — claude-transcript-reader — claudeengine locates and reads Claude transcripts through one parse shared with SessionUsage
3 — claude-pricing — claudeengine prices a message from its model and neutral usage
4 — offline-record-readers — shuttleengines hands out each engine's read capabilities behind adapters
5 — tokencount-reads-through-engines — tokencount counts sessions, forks and orch through the read set and drops its Claude parser
6 — calibration-engine-instruction-files — calibration reads the Webster session engine's instruction files

## Shared Decisions

- Provider rules stay in the engine and lyx-format rules in tokencount (D2): a card that moves a rule names which side it lands on.
- Absence contributes nothing, and a locator or read failure fails the count naming its subject (D7).

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
