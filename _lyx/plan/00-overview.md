---
format: 5
approved: false
language: go
---

# Plan: run-time-by-role

Make `tools/tokencount` report where each finished run's wall-clock time goes beside its tokens: each role's working time as the union of its sessions' turns, an idle row for the time no session of the run is in a turn, and a Producers table splitting the run's span by shed history entry, per run and summed over all runs.
The span and the history come from the run's archived `status.json` and `seed.json`, read at the slug's newest `archive/<slug>/<sha>` tag in the weft repository through `gitrepo`'s read side, with the paths and the seed decoder coming from `internal/shedrun`.
Cards 1 and 2 add the two library seams, cards 3 to 5 add the measurement to the tool, card 6 prints it, and card 7 wires the records read into `run`, applies the two bounds on reading the weft, retitles the report and updates the package doc and the launchers.

## Card Index

1 — gitrepo-tags-under — list tags under a prefix with their commit SHAs and committer times through go-git
2 — shedrun-archived-records — tree-relative seed and status paths without alias resolution, and an exported seed decoder
3 — tokencount-working-intervals — collect each role's working intervals from turn boundaries while counting transcripts
4 — tokencount-shed-records — read a run's archived seed and status from its newest archive tag, with the in-flight bound
5 — tokencount-run-timing — compute the span, per-role time, idle and the Producers split from the intervals and records
6 — tokencount-report-time — print time and time share columns, the idle row and the Producers tables
7 — tokencount-main-wiring — wire the records read and timing into run, bound the weft default, retitle, and document

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
