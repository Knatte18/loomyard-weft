```yaml
lens: code-tests
outcome: reported
```

**Behaviour → pinning test**

| Behaviour | Pinning test |
|---|---|
| `FindWatchedTests` parse: patterns, stacking, misplaced, malformed, parse error | `TestFindWatchedTests` (`watches_test.go`) |
| `UnwatchedTests` matching: base name, slash pattern, own file, sort, nil | `TestUnwatchedTests` |
| `targetsFor` fills `Unwatched` and drops unwatched guards | `TestTargetsFor`, 5 new rows |
| Guard stacked with a watch line on either side | `guards_test.go` new row |
| Computed scope emits `-skip` and lifts it when a `_linux.go` file changes | `TestGateTest_Scopes` new subtest |
| Whole pass (`--all`) runs the watched test | `scopes_integration_test.go` line ~631 |
| `planTierRuns` adds the skip, with gates for whole share, filter flag and no `{args}` | `TestPlanTierRuns`, 4 new rows |
| `isWatchSkip` in `argsMatch` | `green_test.go` rows |
| `holdsTierTest` discounts skipped tests | `TestEmptyTierFailure` new row |
| Malformed watch marker refusal text | `TestGateTest_Scenario` new row |
| `lymbin` fails before the build | `TestBuildInto_UnwritableDirReturnsError` |
| `attachLineWaiting` polls, production line unchanged | `TestAttachLine_EndsOrKeepsTheShellByAttachStatus` and `termwindow_test.go:68` |
| `verifyrun` cancel and expiry on the child's own signal | `TestRun_Cancellation`, `TestRun_ExpiryKillsBackgroundedDescendant` |
| gatecli poll and lease-read seams, release-file stand-ins | the reworked `TestGateTest_DetachedRun` subtests |
| `armDeadline` seam, deadline value armed, none for a local command | `TestRun_RemoteBounds` rows |
| `--card` scope reads the diff from the merge base and skips unwatched tests (`cardChanges`, `skipNames`, `sc.skip`) | none |
| `scanGuards` drops watched guard tests when `changesRead` is set, via `moduleRelative` | none |
| `--card` fallback (no parent branch or unreadable diff) skips nothing | none |
| `--base` scope skip | covered only by the shared `DeriveTargets` path; no `--base` row |
| Production `armDeadline` default (real `context.WithTimeout`) kills a hung remote | none |
| `frontPoll()` and `leaseRead()` zero-value fallbacks | none; `defaultDetachSeams` sets both explicitly |

**Findings**

1. **MEDIUM, design.** The `--card` scope's watch skip has no test. It covers `computeScope`'s `scopeCard` branch (`internal/gatecli/scope.go`, around lines 192–215), `cardChanges` and `skipNames`. The only `scopeCard` hits in tests are `tiers_test.go:99,296,362`, and those rows pass hand-built scopes with no `skip`. The only scope test of the skip is the computed one. Breaking card-scope narrowing would fail nothing. The fix is a `TestGateTest_Scopes` subtest that runs a `--card` call against a watched package and checks `-skip` appears, then disappears once a matching file changes.

2. **MEDIUM, design.** `scanGuards` dropping watched guard tests under `sc.changesRead`, with `moduleRelative`'s prefix trimming, has no test (`scope.go`, around lines 524–546). `targetsFor` covers only the computed path. The dotnet fixture test is the real case, a guard that is also watched. The fix is a card-scope or `moduleRelative` test with a non-root module dir and a watched guard.

3. **LOW, design.** The production `armDeadline` default is no longer exercised by any test. Every `TestRun_RemoteBounds` deadline row now ends on a ready signal, and `SetRemoteDeadlineForTest` is used only in that file. Nothing pins that the real deadline kills a hung remote. The seam is what the board entry asked for, but this coverage was lost. A tiny row with a short deadline against an already-started hang would keep it without a long wait, as long as the bound does not cover a process start.

4. **LOW, design.** The `detachSeams.frontPoll` and `leaseRead` zero-value fallbacks are untested. This matters little because `defaultDetachSeams` sets both.

5. **NIT, design.** `isWatchLine` rejects `//lym:watchx` and accepts a tab separator. No `TestFindWatchedTests` row covers either.

Tool calls:
1. `grep` over test files for `cardChanges`, `moduleRelative`, `changesRead`, `skipNames`, `scopeCard` and `SetRemoteDeadlineForTest`. My context did not show which tests touch the card-scope code or the deadline seam.
