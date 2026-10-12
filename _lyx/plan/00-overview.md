---
format: 5
approved: false
language: go
---

# Plan: shuttle-startup-ready

Shuttle starts stop failing on a ready Claude pane whose input line shows a stray `%`, a not-started producer failure names its cause and the pane's last lines, every shuttle start passes its startup probe within about a second of the provider becoming ready with startup-gate dismissals spaced by the dismissal gap, and a seats chair's held-turn notice is deferred no longer than 10 minutes.
Each card is one decision group of the record, lands with its own tests in the existing tables where one covers the surface, and keeps `shuttleengine` free of Claude specifics.

## Card Index

1 — claude-lone-percent-reads-empty — Claude reads a box holding only `%` as empty, and the pane-text-injection background says so
2 — startup-probe-cadence-and-dismissal-gap — `awaitStartup` probes from `poll_interval_ms` with backoff, and a startup-gate dismissal replays at most once per dismissal gap
3 — never-ready-error-pane-tail — a never-ready teardown's error ends with the last startup capture's last lines
4 — result-start-error — `Result.StartError` carries a not-started run's start error into the single-LLM producer's message
5 — hold-notice-max-deferral — `holdNoticeMaxDeferral` is 10 minutes

## Shared Decisions

### Decision: backoff-over-exemption

- **Decision:** `awaitStartup` follows `PATTERN-daemon-wakeups`'s backoff rule (base `poll_interval_ms`, doubling while the capture does not change, ceiling the dismissal gap, reset on a change), and `pattern/PATTERN-daemon-wakeups.md` is not amended.
- **Rationale:** the record's "Added after Discussion" entry on `awaitStartup` supersedes D5's exemption reading and the Constraints bullet that still says "it gets no backoff"; the later entry is the parent's recorded design call, so the plan follows it and names the conflict here rather than picking silently.
- **Applies to:** card 2

### Decision: never-ready-read-from-started

- **Decision:** `abandonStartup` reads "the provider never reached `StartupReady`" as `run.state.Started` being false, the fact `checkLivenessTick` sets in memory on the ready probe, instead of taking a new parameter.
- **Rationale:** every `awaitStartup` exit runs before the ready probe sets it and every later teardown runs after, so the existing fact answers the record's trigger exactly, and the method's callers keep their shape.
- **Applies to:** card 3

### Decision: counted-play-beside-playInputs

- **Decision:** the gate play's "did the first key go out" answer comes from a new `playInputsCounted`, which `playInputs` calls and whose count it drops.
- **Rationale:** every other caller of `playInputs` needs no count; a re-sign would touch each of them for one caller's benefit.
- **Applies to:** card 2

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
