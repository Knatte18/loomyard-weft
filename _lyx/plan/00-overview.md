---
format: 5
approved: true
language: go
---

# Plan: burler-multi-llm — a Burler round as one seat table

Burler's review round moves onto the seats engine `MultiLLMProducer` runs on: the fixer becomes the `chair`, the reviewer `advisor-1`, and the ad hoc two-agent wiring (`roundHalves`, the parallel `join`, `lyx burler await-review`) goes.
The round's outcome, review files, fixer-report, failure rules and Bouncer contract stay as they are, and `BurlerProducer` keeps its row and its seam.
The seats engine first gains the generic capabilities the round needs (a per-advisor gate, per-seat fork authorization, the round number, a required advisor with an acceptance hook, the advisor's terminal result on its seat result), and shuttle gains a told hold-notice deferral the chair's deferral rides, so a seats chair's planned waits stop reaching the parent as stall notices.
`fix_start` is retired before the switch, and the fixer waits with `lyx shuttle await` inside its turn.

## Card Index

1 — shuttle-hold-deferral — a told predicate on Spec defers a run's held-turn notice by a settle interval, bounded by a maximum deferral
2 — shuttle-run-session-state — a run handle reads its own session state
3 — seat-gate-fork-round — seats gain a per-advisor gate, per-seat fork authorization and the table's round number
4 — seat-required-advisor — a required advisor with an acceptance hook ends the step on any unaccepted end and reports its terminal result
5 — seat-chair-hold-deferral — the chair's held-turn notice waits while an advisor is mid-turn or its up line is pending
6 — burler-geometry-names — burler's geometry is told the shortname and slug its seat strands are named from
7 — retire-fix-start — remove loom.yaml's fix_start and the after-review start order
8 — burler-seat-table — run a round as one seat table, the fixer as chair and the reviewer as required advisor-1
9 — burler-seat-stencils — the orchestrators become seat stencils with Burler's own channel rules and the fixer waits with lyx shuttle await
10 — remove-await-review — delete lyx burler await-review and its docs
11 — delete-fix-start-type — delete the FixStart type and its two constants once nothing references them
12 — delete-burler-shuttle — delete burlerengine's own Shuttle seam and RunnerShuttle once card 8 moved every caller to seatengine's

## Shared Decisions

### Decision: burler-builds-its-seat-engine

- **Decision:** `burlerengine.New` takes a `seatengine.Shuttle` and builds its own `seatengine.Engine` from `Geometry` and the stencils directory; burlerengine's unit tests drive that real seat engine over burlerengine's package-local scripted fake shuttle.
- **Rationale:** the seat engine is the in-process collaborator, so its rules run for real under the round's tests, and the told `Shortname`/`Slug` (D2) have a reader; no fake is shared across packages, so no new testkit kit is needed (`PATTERN-testkit`).
- **Applies to:** cards 6 and 8

### Decision: ended-step-flag

- **Decision:** `SeatResult.EndedStep` records that a required advisor's end ended the step before the chair's; `decideRound` reads it to choose between the reviewer's rules and the fixer's.
- **Rationale:** once the engine stops every seat, both a stopped reviewer and a stopped fixer read as died, so which seat ended the step must be the engine's recorded fact, never inferred from outcomes.
- **Applies to:** cards 4 and 8

### Decision: half-not-stopped-wraps-seat-not-stopped

- **Decision:** `burlerengine.ErrHalfNotStopped` stays and wraps `seatengine.ErrSeatNotStopped`; `BurlerProducer` and its tests are unchanged.
- **Rationale:** D4 allows either; wrapping keeps the producer's refusal and its way forward without touching every review segment's caller.
- **Applies to:** card 8

### Decision: liveround-keeps-its-shape

- **Decision:** `LiveRound`, `LiveHalf`, `HalfState` and `Handle` keep their exported shape; `LiveRound` gains an unexported pointer to the probed `seatengine.LiveTable`, which `Resume` hands back to the seat engine.
- **Rationale:** `BurlerProducer`, `shedfake.BurlerRunner` and their tests build and compare `LiveRound` values; a pointer keeps the struct comparable.
- **Applies to:** card 8

### Decision: reviewer-start-failure-is-not-started

- **Decision:** any reviewer start failure is `OutcomeDied` with `NotStarted` and `Review.StartError`, the chair stopped; a chair start failure keeps today's split between `ErrNotStarted` (not started) and any other start error (returned).
- **Rationale:** D5 states the reviewer case without the split, and the seat result carries an advisor's start error as text.
- **Applies to:** cards 4 and 8

### Decision: hold-deferral-constants

- **Decision:** the settle interval is 45 seconds and the maximum deferral 20 minutes, one constant each in `internal/shuttleengine/hold.go`.
- **Rationale:** D10's bounds: a real stall reaches the parent within a minute, a SendMessage has time to reach an idle advisor, and a wedged advisor hides a chair stall well inside the template's `review_timeout_min` and `discussion_timeout_min`.
- **Applies to:** cards 1 and 5

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
