# Round 4 facts

| Round | Findings | BLOCKING | MEDIUM | LOW | NIT | Highest | design | scope | decision | consistency | Gating |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 8 | 0 | 4 | 4 | 0 | MEDIUM | 7 | 0 | 0 | 1 | 3 |
| 2 | 5 | 2 | 3 | 0 | 0 | BLOCKING | 4 | 0 | 1 | 0 | 4 |
| 3 | 4 | 1 | 2 | 1 | 0 | BLOCKING | 3 | 0 | 0 | 1 | 2 |
| 4 | 3 | 1 | 1 | 1 | 0 | BLOCKING | 1 | 0 | 1 | 1 | 1 |

## Recurring keys (ledger rounds each key was open in)

- `pattern-entries-rewritten-undeclared`: rounds 1, 2, 3
- `replay-tags-after-reset`: rounds 1, 2
- `resetkeep-failure-reads-dirty`: rounds 1, 2
- `watchdog-down-up-idle-window`: rounds 2, 3

## Keys open in an earlier round (ledger rounds each key was open in)

- `final-tip-read-leaves-branch-moved`: rounds 3
- `pattern-entries-rewritten-undeclared`: rounds 1, 2, 3
- `reconcile-verdict-no-slug-filter`: rounds 1
- `refused-restore-untagged`: rounds 3
- `replay-tags-after-reset`: rounds 1, 2
- `resetkeep-failure-reads-dirty`: rounds 1, 2
- `retry-failure-hides-cause`: rounds 1
- `set-aside-paths-in-dirtiness-check`: rounds 1
- `settle-unmoved-weft-swallows-read-error`: rounds 2
- `unclassified-push-failure-unrecorded`: rounds 1
- `watchdog-down-up-idle-window`: rounds 2, 3
- `weft-error-stalls-warp-arm`: rounds 1
