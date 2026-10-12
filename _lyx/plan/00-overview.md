---
format: 5
approved: true
first_card: 16
language: go
---

# Plan: lyx report convergence — rework round 1

Cards 1 to 15 built a read-only `lyx report convergence` verb that reports, per finished run and per ISO week, what woke the orch, what stopped and repaired runs, which steps an LLM took by hand, how many review rounds each segment took, and landings by kind from the parent branch's first-parent history.
The pull request was rejected on one finding: `shedrun.ParseSeedTolerant` survived the Merge-In of run-robustness, whose tolerant `ParseSeed` the parent decision makes the report's one seed parser.
This generation removes the second parser and points the report at `ParseSeed`.

## Card Index

16 — single-seed-parser — delete ParseSeedTolerant, read the report's seeds through ParseSeed, and drop the tests that pin the second parser

## Shared Decisions

### Decision: one-seed-parser

- **Decision:** the report decodes a `seed.json` read from a git tree with `shedrun.ParseSeed`, the parser `ReadSeed` uses, and `shedrun` exports no other seed parser.
- **Rationale:** the decision record's `Added after Discussion (parent, 2026-10-12): Seed parsing follows run-robustness D4` entry overturns D5's bar on calling `ParseSeed` for the seed alone, since run-robustness made it tolerant of unknown keys and `PATTERN-tolerant-records` allows one parser per record.
- **Applies to:** card 16

### Decision: no-side-tokens-in-report

- **Decision:** `internal/reportengine` spells no fabric side token and no `_lyx` or `.lyx` literal, in code, comments or docs.
- **Rationale:** `PATTERN-fabric-vocabulary` bars a side-named spelling from every non-owner file, and `PATTERN-lyxdirs-single-declarer` keeps the directory literals in `lyxdirs`.
- **Applies to:** card 16

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
