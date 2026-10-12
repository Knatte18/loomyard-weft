---
format: 5
approved: false
language: go
---

# Plan: shuttle-startup-ready

Shuttle starts stop failing on a ready Claude pane whose input line shows a stray `%`, a not-started producer failure names its cause and the pane's last lines, every shuttle start passes its startup probe within a probe or two of the provider becoming ready while startup-gate dismissals stay one gap apart, and a seats chair's held-turn notice is deferred no longer than 10 minutes.
The `%` knowledge lives only in `claudeengine`; the cadence, tail and start-error changes live in `shuttleengine` and the single-LLM producer's message.

## Card Index

1 — claude-lone-eol-mark-reads-empty — Claude's box reading treats a lone `%` as empty, and the pane-text-injection background records it
2 — result-carries-start-error — Result gains StartError, set on RunGated's not-started branch and appended to the single-LLM producer's message
3 — never-ready-error-ends-with-pane-tail — a never-ready teardown's error ends with the last startup capture's last lines
4 — startup-probe-backoff-and-dismissal-gap — awaitStartup probes every poll interval with backoff, and a startup gate is dismissed at most once per dismissal gap
5 — hold-deferral-cap-ten-minutes — holdNoticeMaxDeferral drops from 20 to 10 minutes

## Shared Decisions

### Decision: started-flag-tells-never-ready

- **Decision:** `RunState.Started`, persisted by the ready probe, is how `abandonStartup` tells a never-ready exit from the skill-load death that also reaches it with a nil cause.
- **Rationale:** both exits call the same teardown with no cause; the persisted flag is the one fact that differs between them and needs no new parameter.
- **Applies to:** card 3

### Decision: gap-throttle-before-faster-probes

- **Decision:** the dismissal-gap throttle and the per-poll backoff land in one card.
- **Rationale:** today both `awaitStartup` and `Wait` probe exactly one gap apart, so the throttle alone is unobservable, and the faster probing alone replays a dismissal on every probe, the hazard the gap exists to prevent.
- **Applies to:** card 4

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
