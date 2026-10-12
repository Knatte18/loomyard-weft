---
format: 5
approved: true
language: go
---

# Plan: lyx report convergence

A read-only `lyx report convergence` verb, run from the hub's prime, reports per finished run and per ISO week what woke the orch, what stopped and repaired runs, which steps an LLM took by hand, how many review rounds each segment took, and, from the parent branch's first-parent history, landings by kind and the share of fixes that repair freshly landed code.
Every count comes from records lyx already writes, read from git objects at refs resolved once, each record through a tolerant parse in the package that owns its format; a record a run lacks is reported absent, never zero.
The cards first give each owning package its reader (cards 1 to 7) and `gitrepo` its new reads (cards 8 and 9), then build `internal/reportengine` source by source (cards 10 to 14), and last mount `internal/reportcli` (card 15).

## Card Index

1 — halt-note-leaf — move the halt-note format into a stdlib-only leaf that renders and tolerantly parses it
2 — shed-record-parsers — tolerant seed and status parses and the tree-relative run record paths
3 — loom-record-readers — friction tree paths, the tolerant arbiter attempt parse and the landing-checkpoint subject
4 — alignment-record-readers — tolerant alignment attempt and parent-review round parses
5 — review-round-and-decision-readers — review round counts and outcomes, and the Added after Discussion reader
6 — board-and-archive-readers — the tolerant board parse, the archive tag prefix and slug, and the task-commit trailer accessor
7 — batten-change-classifier — parse and classify batten's recorded child state changes under batten's notice rules
8 — gitrepo-history-reads — tag listing, the tip walk with first-parent, since, path and limit, and subdirectory listing
9 — gitrepo-landing-reads — pre-image lines of a commit and first-parent blame
10 — report-run-selection — the report engine's told geometry, ref snapshot, run unit, batten span and findings
11 — report-wakes-and-stops — per-run wakes by source and kind, automatic steps, alignment outcomes and stop classes
12 — report-repairs-and-rounds — arbiter repairs, LLM steps, drive reports, review rounds and parent decisions
13 — report-landings — landings by kind through the exact and time joins, and the fresh-code fix share
14 — report-weeks-and-markdown — weekly aggregates with absent counts, and the markdown rendering
15 — report-cli — the lyx report convergence verb, its refusals, wiring test, docs and sandbox scenario

## Shared Decisions

### Decision: tolerant-readers-beside-their-writers

- **Decision:** every record the report reads is decoded by a new function in the package that writes or owns its format, taking bytes or a directory's files keyed by name, tolerating unknown keys and running none of the live reader's validation; no existing strict reader changes.
- **Rationale:** D5 and D9 confine tolerance to the report's read path and keep one owner per format, and taking bytes lets the engine read every record out of a git tree.
- **Applies to:** cards 1 to 7

### Decision: no-side-tokens-in-report

- **Decision:** `internal/reportengine` and `internal/reportcli` spell no fabric side token and no `_lyx` or `.lyx` literal, in code, comments or docs; they reach fabric through `fabricengine`'s neutral exports (`RecordsRepoRoot`, `RecordsBranchName`, `ArchiveTagPrefix`, `ArchiveTagSlug`, `TaskCommitOf`) and record paths through `shedrun`, `loomengine` and the record packages.
- **Rationale:** `PATTERN-fabric-vocabulary` bars a side-named spelling from every non-owner file, owner exports included, and `PATTERN-lyxdirs-single-declarer` keeps the directory literals in `lyxdirs`.
- **Applies to:** cards 6, 10 to 15

### Decision: report-json-shape

- **Decision:** every report type carries snake_case JSON tags from the card that creates it, and a figure for a source a run or week lacks is a `nil` pointer or map that encodes as `null`.
- **Rationale:** D5 forbids reporting an absent source as 0, and D12 makes the JSON envelope the surface an agent consumes.
- **Applies to:** cards 10 to 14

### Decision: reroute-entries-as-llm-steps

- **Decision:** a `reroute` entry counts as an LLM step, as the record's D8 and its acceptance criterion say, except the one an automatic step accounts for: the `PR-Gate` reroute that follows batten's resume for a moved parent.
- **Rationale:** the record's text binds the plan; the exception is D8's own "that no automatic step accounts for", and it keeps one moved-parent resume from counting both as an automatic step and as an LLM step.
- **Applies to:** cards 7, 12

### Decision: wake-mean-without-ready-tasks

- **Decision:** a week's wake mean counts the runs holding every wake source but `land-ready-tasks`, which D6 makes absent for every run; that source is still reported absent per run.
- **Rationale:** D5 admits a run to the mean only when it holds every wake source, and an always-absent source would leave every week without a mean, which no board question could use.
- **Applies to:** card 14

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
