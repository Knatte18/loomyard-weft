# Add lyx report convergence: orch wakes, repairs, fresh-code fixes

## What changes

A new read-only verb, `lyx report convergence`, reports per finished run and per ISO week (UTC):

- what woke the orch, by kind;
- which stops and failure classes recurred;
- which steps an LLM took by hand, split from batten's automatic retries and the arbiter's repairs;
- parent decisions added after Discussion;
- review rounds per segment and their outcome;
- landings by kind, and the share of fixes that repair code landed within a few days.

It answers the four questions on the board entry: do orch wakes per run fall, do the same stop and failure classes recur, does the fresh-code fix share fall, and do review rounds per segment fall after a change meant to cut them.
Every count is derived from records lyx already writes; no orch, driver or agent keeps new bookkeeping.

Flags: `--since <YYYY-MM-DD>` (default twelve weeks back), `--fresh-days <N>` (default 3, at least 1), `--format json|markdown` (default json).
It runs from the hub's prime worktree only and refuses elsewhere, naming the prime.
`internal/reportcli` and `internal/reportengine` are new, `cmd/lyx/main.go` mounts the `report` group, and `docs/overview.md`, the module `doc.go` files, `tools/sandbox/SANDBOX-CORE-SUITE.md` and `cmd/lyx/testdata/test-budget.yaml` are updated.

## Design decisions

- **Git objects only.**
  All records are read from refs resolved once at the start (archive tags, the prime's records branch, the board branch, the parent branch).
  The verb writes nothing, takes no lock and makes no commit.
  `internal/gitrepo` gains tag listing, a first-parent range walk, pre-image lines, first-parent blame and subdirectory listing.
- **One run per `archive/<slug>/<sha>` tag.**
  Only full loom runs with a Finalize entry enter per-run figures; other tags count per week as `partial`.
- **Absent is not zero.**
  A run whose archive predates a record reports that source as absent, weekly aggregates state how many runs lack it, and a week with no data reports no data.
  A record that exists but does not parse is a finding naming run and file; the exit code stays 0.
- **Tolerant parsing is confined to the report.**
  Each owning package gains a tolerant parse-from-bytes function; no reader a live run or gate calls changes acceptance.
  The report reads seeds through `shedrun.ParseSeed`, the single seed parser.
- **Wakes come from durable records.**
  `internal/battenshed` gains a classifier beside the writer of the state-change text.
  It separates wakes (`review-escalation`, `pr-gate`, other blocked/failed states, Land and Worktree-Teardown failure episodes) from automatic steps and ordered pauses.
  Parent-review rounds, applied alignment approves naming superseded entries and arbiter attempts come from their own records.
  The Land ready-tasks notice leaves no git object, so that kind is always absent.
- **Stop classes from halt notes.**
  The halt-note format moves into the new leaf package `internal/haltnote`, which `internal/loomcli` writes through and the report parses through.
  `PATTERN-sole-parsers.md`, `PATTERN-leaf-packages.md` and `PATTERN-told-geometry.md` are updated in this change.
- **Landing kind.**
  A landing maps to its run by the `Warp-SHA` trailer of the run's newest weft commit (fast-forward, merge or squash), then by a time window from the Finalize checkpoint to the board's done commit.
  Kind comes from the board entry's labels as last recorded before the landing: `bug` is `fix`, `feature` or `enhancement` is `feature`, else `other`.
  A commit mapping to no run is `unmapped`; a run landed without a Finalize `done` is `hand`.
- **Fresh-code share.**
  For `fix` and `unmapped` landings, the modified or deleted pre-image lines (excluding Go tests, `docs/` and markdown) are blamed at the landing's first parent.
  A fix is `fresh` when more than half of those lines landed within `--fresh-days`.
  The `unmapped` share is reported beside the fix share and never mixed into it.
- **Departure from the board entry.**
  Drive reports are counted per run, not parsed, because their bodies are LLM prose.

## Known limits

- A halt resumed between two batten reads leaves no state-change entry and counts only through its leftover halt note.
- Board labels are assigned unevenly, so a bug fix labeled `enhancement` counts as a feature.
- Archive tag dates can skew against other timestamps and move a run across a week boundary.
- Early weeks carry few comparable runs, since older archives lack most sources.
- A clean Tier 2 run leaves no committed record, so it reads as absent stop classes and weekly stop-class figures cover only runs that left a note.
- A hand landing or direct push dated inside a run's checkpoint-to-done window that the exact join did not claim is taken by the time join, and the run's real landing then reads `unmapped`.

## To check by hand

- Run `lyx report convergence --format markdown` from a real hub's prime and compare a few rows against `lyx batten` history and `git log --first-parent` on the parent branch.
- Check the report's output on a hub with older archives: absent sources must show as absent, not 0.
- `internal/gitrepo/blobread.go` was reworked (about a hundred lines) to share blob reads with the new history reads; confirm existing callers behave unchanged.
- `internal/reportengine/runs.go`, `report.go` and `batten.go` and `cmd/lyx/gitrepoboundary_test.go` differ from the plan as first written; review them for fit with the plan's rules.
