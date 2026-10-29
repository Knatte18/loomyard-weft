# Record the refill after a logger change at tier_parallel 4 and 8

Adds a section to `docs/benchmarks/gate-test-xdr.md` that measures the case the earlier `go_parallel` sweep left open: a refill after a real change to a widely imported package, `internal/logger`, on OSL-1046 with Cortex XDR.

Two gate runs, `lym gate test --wait --upto 2`, one at `tier_parallel` 4 and one at 8, each with its own edit to `internal/logger`, so neither could reuse the other's test cache.
The verify took 497 s at 4 and 543 s at 8, with 22 and 38 held packages, and 8 was no faster than 4; `tier_parallel` stays 4.
Only the doc changes; no code, PATTERN or `doc.go` is touched.

## Design decisions

- `go_parallel` is retired, so the sweep set `tier_parallel` for tiers 0 to 2 in the machine layer.
  The task session is refused that hub-wide write, so the orch wrote it and removes it after the runs.
- The scope is the diff, as in a darn verify, so each tier ran the packages that import `internal/logger`.
- The run at 8 failed one flaky `internal/creelengine` test, `TestScheduler_HookWake/MailArrivingInAWokenTurnsBlockedChainRingsNothingUntilANewTurnEnd`; the cap allowed no repeat, so the row records it.
