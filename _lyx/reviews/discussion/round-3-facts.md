# Round 3 facts

| Round | Findings | BLOCKING | MEDIUM | LOW | NIT | Highest | design | scope | decision | consistency | Gating |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 14 | 3 | 6 | 5 | 0 | BLOCKING | 5 | 1 | 2 | 6 | 5 |
| 2 | 19 | 1 | 9 | 9 | 0 | BLOCKING | 11 | 2 | 3 | 3 | 9 |
| 3 | 18 | 1 | 9 | 6 | 2 | BLOCKING | 4 | 1 | 5 | 8 | 4 |

## Recurring keys (ledger rounds each key was open in)

- `d1-d2-refusals-uncovered`: rounds 1, 2
- `d12-recoveries-count-not-carried`: rounds 1, 2
- `d7-allowlist-refuses-nested-orch`: rounds 1, 2
- `d8-d5-splitter-and-pinned-tests`: rounds 1, 2
- `d8-reader-write-flags-unbounded`: rounds 1, 2
- `d9-gated-leftover-files`: rounds 1, 2
- `scope-bullets-without-criteria`: rounds 1, 2

## Keys open in an earlier round (ledger rounds each key was open in)

- `d1-d2-refusals-uncovered`: rounds 1, 2
- `d1-warning-no-sink-restamp-save`: rounds 1
- `d11-candidates-perishable-criterion-not-red`: rounds 1
- `d12-card-amended-cap-no-master-rung`: rounds 2
- `d12-commit-test-inherited-startsha`: rounds 1
- `d12-new-refusal-unmarked`: rounds 1
- `d12-recoveries-count-not-carried`: rounds 1, 2
- `d7-allowlist-refuses-nested-orch`: rounds 1, 2
- `d7-guard-refuses-detached-push`: rounds 2
- `d7-refusal-spec-ways-forward`: rounds 2
- `d8-classifier-seam-unstated`: rounds 2
- `d8-d5-splitter-and-pinned-tests`: rounds 1, 2
- `d8-reader-write-flags-unbounded`: rounds 1, 2
- `d9-gated-leftover-files`: rounds 1, 2
- `d9-test-claims-disagree`: rounds 2
- `friction-constraint-no-plumbing`: rounds 1
- `goal-stale-tally-interview-contradiction`: rounds 1
- `mergeresolve-import-question-in-support-log`: rounds 1
- `scope-bullets-without-criteria`: rounds 1, 2

## Lenses (the latest round's findings per origin)

Fan: `standard`

Lenses run: generic, generic, correctness, error-handling, test-gaps

Lenses already excluded for the next round: none

| Origin | Findings | BLOCKING | MEDIUM | LOW | NIT | design | scope | decision | consistency |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `handler` | 4 | 1 | 3 | 0 | 0 | 3 | 0 | 0 | 1 |
| `lens:correctness` | 3 | 0 | 2 | 1 | 0 | 0 | 0 | 3 | 0 |
| `lens:error-handling` | 1 | 0 | 1 | 0 | 0 | 1 | 0 | 0 | 0 |
| `lens:generic` | 6 | 0 | 1 | 4 | 1 | 0 | 0 | 1 | 5 |
| `lens:test-gaps` | 4 | 0 | 2 | 1 | 1 | 0 | 1 | 1 | 2 |
