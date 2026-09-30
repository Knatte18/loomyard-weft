---
format: 5
approved: false
language: go
---

# Plan: bound a batten run's status history during budget-exempt waits

`shedengine` appends one `history[]` entry per `Run-Shed` poll, and budget-exempt Stucks are not capped by `max_bounces`, so a multi-day approval wait writes tens of thousands of near-identical entries.
This plan folds consecutive identical budget-exempt Stucks into one entry carrying `repeats` and `last_at`, engine-generically, inside the step's single persist.
It also stops `Run-Shed` from asking for a re-approve on the poll right after an approved resume, by recording the child's history length in the approval-acted marker and reporting "resumed, driver has not re-stepped yet" while that length is unchanged.

## Card Index

1 — history-fold — add `repeats`/`last_at` to `HistoryEntry` and the pure append-or-fold helper
2 — step-fold-wiring — route every verdict arm through the fold, recompose `activity.last` after a fold, update `docs/overview.md`
3 — approval-acted-layout — render/parse helpers for the two-line approval-acted marker
4 — awaiting-resume-pending — tell a pending resume from a re-await in `callAwaiting`

## Shared Decisions

### Decision: fold-key

- **Decision:** An entry folds only when it and the current last entry are both budget-exempt Stucks with the same producer, the same `output`, and equal `gate_attempts`; the last entry's own `repeats`/`last_at` never take part in the match.
- **Rationale:** those entries were otherwise byte-identical, so a fold loses only per-poll timestamps, and any interleaved entry ends the fold.
- **Applies to:** cards 1, 2, 4

### Decision: marker-layout

- **Decision:** The approval-acted marker is the unchanged identity line followed by the child's history length in decimal; identity alone decides whether an approval is new, and a one-line or unparseable second line reads as the old layout.
- **Rationale:** a re-await always lengthens the child's history, so equal length means the resumed driver has not re-stepped, with no clock involved.
- **Applies to:** cards 3, 4

## verify:

go build ./...
go vet ./internal/shedengine/... ./internal/battenshed/...
go test ./internal/shedengine/... ./internal/battenshed/... ./internal/battencli/...
go test -tags integration ./internal/shedengine/... ./internal/battenshed/... ./internal/battencli/...
