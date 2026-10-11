---
format: 5
approved: false
language: go
---

# Plan: run-time-by-role

`tools/tokencount` reports where each finished run's wall-clock time goes: each role's working time and time share beside its tokens, an `idle` row for the time no session of the run is in a turn, and a Producers table splitting the run's span by shed history entry, per run and summed in "All runs".
Working time is turn time read from the transcripts; the span and the Producers rows come from the run's archived `seed.json` and `status.json`, read from its newest `archive/<slug>/<sha>` tag in the weft repository through `gitrepo`'s read side, with `shedrun` naming the paths and decoding the seed.

## Card Index

1 — shedrun-tree-records — add shedrun's tree-relative seed and status paths and a seed decoder over bytes
2 — turn-time-by-role — measure turn time per role and print time columns and an idle row in the role tables
3 — archive-tag-dates — read each archive tag's commit and commit date, newest first per slug
4 — shed-span-producers — set a run's span from its archived shed status and print the Producers tables
5 — run-reads-shed-status — wire the status read into run, refuse a foreign -hub without -weft, retitle the report
6 — time-docs — describe the time columns, idle row, span and Producers table in the package doc and launchers

## Shared Decisions

### Decision: one interval toolkit for role time, idle and producer idle

- **Decision:** `Interval`, `unionIntervals` and `covered` from card 2 compute every time figure: a role's time is `covered` of its union over the span, idle is the span minus `covered` of the run's union, and a producer's idle is its interval minus `covered` of the run's union over it.
- **Rationale:** the three figures differ only in the set and the window, so one helper keeps them consistent and makes elapsed and idle add up to the span by construction.
- **Applies to:** cards 2 and 4

### Decision: the in-flight check covers every counted slug

- **Decision:** a slug whose worktree `<hub>/<slug>` exists reads no archive tag, whether it was named or picked by `recentRuns`.
- **Rationale:** Decision 8 bounds named slugs because teardown removes the worktree; the same reason holds for a picked slug that reuses an older run's slug, and one check site covers both.
- **Applies to:** card 5

### Decision: a status with no history entry counts as no status

- **Decision:** an archived status whose history is empty gives no record, so the run keeps its transcript-window span and prints no Producers table.
- **Rationale:** Decision 3's span ends at the latest history entry's end, which an empty history does not have; the no-status fallback is the record's own answer for a run without that end.
- **Applies to:** card 4

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
