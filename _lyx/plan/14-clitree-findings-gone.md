# Card 14 — the CLI tree walk has no recorded exceptions

**Delete:**
- `cmd/lyx#cliTreeFindings`

**Edit:**
- `cmd/lyx#checkInvocation`
- `cmd/lyx#TestCLITree_EveryCommand`

**Intent:**
With cards 12 and 13 landed, no group's bare or bogus invocation differs from the invariant (D11).
Delete `cliTreeFindings`, the allowlist built from it in `TestCLITree_EveryCommand`, its stale check, and the known-finding path in `checkInvocation`, so every command must meet the invariant outright.
Drop the `scankit` import if nothing else in the file uses it.

Bound: this removes the escape hatch rather than adding one; a future regression fails the walk instead of being recorded.

**ImpactSummary:** `cmd/lyx/clitree_test.go` only; the walk now fails on any bare or bogus deviation.

**Commit:** 14: the CLI tree walk has no recorded exceptions
