# Webster round 2 — fixer report

## Changes

- **F1** (BLOCKING, design) — fixed as proposed in `321e93b13`.
  The premise holds: `TestRenderIndex_FormsAndFiltering` ran its audience subtests with `t.Parallel()` over one shared `root`.
  `visibleLocalFlags` calls `LocalNonPersistentFlags`, which merges parent persistent flags into each command's `pflag.FlagSet` on first read.
  `.lyx/verify/verify-4.log` shows the resulting `fatal error: concurrent map writes` from `TestRenderIndex_FormsAndFiltering.func1`.
  The subtests no longer call `t.Parallel()`, and a comment above the loop names the shared state, as `PATTERN-test-speed` requires for a test over a shared fixture.
  The top-level test keeps `t.Parallel()`, since it builds its own tree.
  Verified with `lyx gate test ./internal/clihelp -- -race -count=50 -run TestRenderIndex_FormsAndFiltering` and `lyx gate test ./internal/clihelp -- -race -count=1`, both green.

## Disputed

None.

## Deferred

None.
