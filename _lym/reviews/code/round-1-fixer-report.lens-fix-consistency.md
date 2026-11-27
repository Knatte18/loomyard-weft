```yaml
lens: fix-consistency
outcome: reported
```

Checked that the fix's change set agrees with its surroundings (fix-consistency lens). Three LOW findings, no BLOCKING or MEDIUM.

1. **LOW** — `internal/gatecli/redfirstdotnet.go:293`, the `noTestRan` field comment: it does not mention the new memory pressure exception.
   - Evidence: the comment reads "neither timed out nor crashed their runner". The fix made line 334 also require `!pressured` and updated the function doc at line 308, but not this field comment.

2. **LOW** — `_lym/landing/summary.md`, `## Tests`: the section was left stale; of it, only the `TestMemoryGuard` opening line changed.
   - `TestRunTierMemoryGuard` is described only as "a slice-pressure kill marks the result for a rerun, and a kill over the cap does not". It now also covers the killed package's own failure, another package's failure and a failed test.
   - `TestRetryTier` is described without the new `memory_pressure` mark assertions.
   - `TestProcessTreeReads` is described without the root and shared-prefix cases.
   - The section omits `TestRunStopsSlicePressureOnly`, `TestUnjudgedRunsClosingLineAndStopsCall`, the `TestMergeCopies` slice-pressure rows, and the `verifytree`, `loomshed` and dotnet red-first pressure rows.

3. **LOW** — `internal/landingshed/fixerrun_test.go:300`, the row name: it still says "is the crash end", though the fix renamed that end to `fixerEndUnjudged` (value `"unjudged"`) and changed line 306 to match.

Outside this lens's scope (the original diff left them, the fix did not touch them): `internal/gatecli/cli.go` lines 36, 50, 162 and 189 still describe the rerun delay and `--stop-on-fail` without the memory pressure case.

Tool calls (3 of 6):
- Grep for crash-only wording across `internal`, `cmd`, `docs` and `README.md`, plus the diff stat: my context did not show whether other files held twins.
- Read `_lym/landing/summary.md` whole and grepped the remaining `RunnerCrash` mentions: my context held only the pre-edit summary plus the edit fragments.
- Read spans and grepped `gatecli/doc.go`, `cli.go` and the red-first files for pressure and crash wording: my context lacked `cli.go` and `doc.go` lines 18-28.
