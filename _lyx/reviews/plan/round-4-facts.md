# Round 4 facts

| Round | Findings | BLOCKING | MEDIUM | LOW | NIT | Highest | design | scope | decision | consistency | Gating |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 19 | 6 | 4 | 8 | 1 | BLOCKING | 11 | 2 | 0 | 6 | 8 |
| 2 | 9 | 1 | 4 | 4 | 0 | BLOCKING | 4 | 2 | 0 | 3 | 4 |
| 3 | 5 | 0 | 2 | 3 | 0 | MEDIUM | 5 | 0 | 0 | 0 | 2 |
| 4 | 2 | 0 | 1 | 1 | 0 | MEDIUM | 2 | 0 | 0 | 0 | 1 |

## Recurring keys (ledger rounds each key was open in)

- `arming-refusal-reads-as-interrupted`: rounds 1, 2, 3
- `loop-lock-liveness-race`: rounds 1, 2
- `second-waiter-loses-envelope`: rounds 2, 3
- `session-end-respawn-window`: rounds 2, 3

## Keys open in an earlier round (ledger rounds each key was open in)

- `allowlist-not-in-overview-entry`: rounds 2
- `arming-refusal-dead-loop-write-order`: rounds 3
- `arming-refusal-reads-as-interrupted`: rounds 1, 2, 3
- `arming-refusal-row-no-way-forward`: rounds 2
- `card19-decide-resign-callers`: rounds 1
- `card8-bundles-three-units`: rounds 1
- `detached-loop-arming-refusal-lost-to-log`: rounds 3
- `failure-stop-invariant-early`: rounds 1
- `failure-stop-pattern-line-shape`: rounds 2
- `findings-file-wrong-scratch-dir`: rounds 1
- `interrupted-stops-no-way-forward`: rounds 1
- `kills-not-logged`: rounds 1
- `loop-detached-flag-unbounded`: rounds 1
- `loop-lock-liveness-race`: rounds 1, 2
- `pattern-overview-over-cap`: rounds 1
- `report-arm-error-test-overlap`: rounds 1
- `run-lock-wait-wrong-pid`: rounds 2
- `second-waiter-loses-envelope`: rounds 2, 3
- `session-end-respawn-window`: rounds 2, 3
- `shedverbs-trace-path-join`: rounds 1
- `sigquit-no-grace`: rounds 1
- `stale-pid-file-after-envelope`: rounds 1
- `start-go-header-stale`: rounds 1
- `tests-drive-unexported-helpers`: rounds 1
- `until-stop-inflight-record`: rounds 2
- `waiter-fswatch-misses-dead-loop`: rounds 1
- `windows-code-not-compiled`: rounds 1
- `windows-killtree-unnamed-job`: rounds 1
