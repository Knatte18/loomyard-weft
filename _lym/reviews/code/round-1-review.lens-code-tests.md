```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

## Per-behaviour list

| Behaviour | Test that pins it |
|---|---|
| `cmd/lym` `run`: builds the root, sets args, returns the exit code | `TestRunDispatchesToUninitializedRepoModules`, `TestExitCode_*`, `TestJSONHelp_Schema` |
| `run` sends stdout and stderr to separate writers, and the JSON error envelope to stdout (`cmd/lym/main.go` `run`, `SetErr(stderr)`) | **none** — every caller passes one buffer as both writers |
| `main` calls `run(os.Args[1:], os.Stdout, os.Stderr)` | none (main is never under test; this was untested before too) |
| `configcli.resolveReal`'s real sync through `clihelp.ExecuteIn(fabriccli.Command(), cwd, …)` | none found in configcli; unit tests inject a fake sync (`configcli.go:806-818`) — no change from before the diff |
| `reedcli` `executeReedIn`: empty cwd runs `clihelp.Execute` | smoke tests through `socketAndSession` (tier 2) |
| `paneIDForStrandIn` now calls `ExecuteIn` directly | its callers, which all pass a non-empty cwd (`naming_integration_test.go:258`, `smoke_lifecycle_test.go:377/428`, `smoke_selvage_keepalive_test.go:145`, `smoke_staterecovery_test.go:526`, `smoke_teardown_test.go:266/271`) |
| Every `RunCLI`/`RunCLIIn` call replaced by `clihelp.Execute`/`ExecuteIn` | the rewritten tests themselves; same arguments, same single writer |
| Guard: detects unreachable production functions, admits allowlist entries, fails a stale entry | `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain`; its non-empty allowlist plus `RequireNoStale` doubles as a self-check (detection that marked everything reachable would trip the stale check) |
| Guard: roots at test-support packages, marker-method exemption, generated-file skip | only implicitly, through today's tree; no seeded row |
| `ownedBoardWorktree` refusing a foreign board, live through the Bolt replay (`replay.go:755`, `destroy.go:559`) | **none** — its only test was deleted |
| `tmuxProcessName` (moved to `proctree_windows.go`) | `TestTmuxProcessName` in `proctree_windows_test.go`, Windows hosts only |
| Hibernate ended phases equal the engine's terminal phases | `TestHibernateEndedPhases_AreTheEnginesTerminalPhases`, weakened (F4) |
| `PushAnchored` (now in `export_test.go`) | `TestPushAnchored_*` — tests whose subject is test code (F2) |
| `SnapshotWarpSHAForTest` read loop (now in `export_test.go`) | `TestSnapshotWarpSHA_*` — mostly pin test code; it serves as an oracle only at `commit_integration_test.go:263` |
| Anchor recovery over a standing move record (lost `TestEnsureAnchor`) | `anchorchange_integration_test.go:466-477`, covered |
| `seatengine` reserved markers sorted and complete | `TestTable_NamesAccessorsAndReservedMarkers`, now reading the var directly |
| `termwindow` resolve | `TestResolve_ChoosesLauncherPerPlatform`, now reading `launcher.name` |
| `AppendDecisions` single-entry path through the test-file wrapper | `TestAppendDecision_*` |
| `discussionparser` batch path | `TestAppendDecisions_WritesBatchWholeOrNothing` |
| Every other deleted test (`decodeLegacy`, `PushWarpAt`, `mergeableTips`, `planUpLaunches`, `Report.Has`, `RemoveLinksIn`, `slugReservedNames` rows, span, runstart, `shedbuild.Load`, messagetail, `LoomReworkCoveragePath` row, `WeftLymDir`/`CodeLymLink` rows, `BareSiblingPath` rows) | tested only the deleted function, so no live behaviour lost coverage |

## Findings

**F1 — MEDIUM — design — `internal/fabricengine/destroy.go:291-297` and `:559`, live caller `internal/fabricengine/replay.go:755`**
- Summary: the board-ownership predicate refuses a board-named directory that no weft repo of the hub registers. Nothing tests that refusal any more.
- The diff deleted `TestBoardDropRequest_RefusesForeignBoard` (formerly `boltpullfirst_integration_test.go:~675-693`) together with `dropSeedCommits`.
- `ownedBoardWorktree` still guards the live Bolt replay's `reset --keep`. A grep of `*_test.go` for `ownedBoardWorktree`, `pathOwnershipBoardWorktree` or `BoardWorktree` refusals finds nothing.
- Fix: re-target the deleted test at the live path, e.g. `checkPathRequest(replayRequest(foreign, ownedBoardWorktree(), nil))` in an internal test, asserting `CheckOwnership` and an empty mutation record.

**F2 — MEDIUM — design (test economy) — `internal/fabricengine/pushanchored_integration_test.go:46`, `:93`, `:436`; `internal/fabricengine/pushclasses_integration_test.go:626` (row `AParkedPairPush`), `:681`, `:709`, `:732`, `:753`**
- Summary: these tests drive `PushAnchored`, whose whole body now lives in `export_test.go:801-833`. They assert the behaviour of test code, not of a live entry point.
- `TestPushAnchored_PushesAndRecordsBranchPush` and `TestPushAnchored_DivergedWeftRemote_ReturnsErrPushRejected` are named for it. The file header (`pushanchored_integration_test.go:1-2`) still says it covers `PushAnchored`, and `TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks` describes "the push lock both entries take".
- The live records-side push is `PushPairAnchored` at the prime, which the cross-package callers were switched to.
- Fix: re-target these rows at `PushPairAnchored`, or delete the rows that `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` already covers.

**F3 — LOW — design — `cmd/lym/main.go` `run` (the `root.SetErr(stderr)` line and the doc line "A JSON error envelope goes to stdout")**
- Summary: the stdout/stderr split that `run` now carries is pinned by no row.
- Every `cmd/lym` caller passes the same buffer as both writers; a grep for `run(` with two different writers returns nothing.
- Fix: add a row with separate buffers asserting that a failing command's envelope lands on stdout.

**F4 — LOW — design — `internal/fabriccli/hibernatedaemon_test.go:24-33`**
- Summary: the engine's terminal phases now come from a hand-written list of eight phases filtered by `Request.Terminal()`. A terminal phase the engine adds later but this list omits escapes the "engine terminal ⊆ ended" check.
- Before the diff, the list came from the engine (`TerminalPhases()`), so this check is weaker than before.

**F5 — LOW — design — `internal/reedengine/proctree_windows_test.go`**
- Summary: `TestTmuxProcessName` now compiles only on Windows, so the Linux gate no longer runs it. The `.exe` and backslash derivation is pinned only on a Windows host.
- This follows the pattern's Windows-file bound. Recorded so the coverage drop is a known one.

**F6 — LOW — consistency (test economy) — contained tests the redundancy report lists that the diff touched**
Each pins something its covering test does not assert, so each should carry `//testtiming:keep <reason>` above its `func Test…` line:
- `cmd/lym/exitcode_test.go` `TestExitCode_HelpPaths`: exit 0 and no error envelope on help paths. The covering test asserts only exit 1 (`main_test.go:45-51`).
- `TestExitCode_UnknownModule`: the "unknown command" error text (`exitcode_test.go:~103`).
- `TestExitCode_HandlerFailure`: a board handler's failure envelope decodes whole (`envelope.Decode`). The covering test checks only an `"ok":false` substring.
- `cmd/lym/jsonhelp_test.go` `TestJSONHelp_Schema`: the JSON help schema.
- `internal/battencli/cli_test.go` `TestRunCLI_GroupGuard_NoGitRepoNeeded`: a bare group exits 0 outside a repo. `TestCancelCmd_RefusesBeforeAnything` asserts exit-1 refusals only.
- `internal/burlercli/cli_test.go` `TestRunCLI_GroupGuard_OutsideGitRepo`: a bare group exits 0 outside a repo. I did not read the covering `TestValidateReviewVerb_AgreesWithTheReviewGate`; it drives a verb, not the bare group.
- `internal/dobbyengine/landnotice_creel_test.go` `TestLandNotice_ReachesAnOrchStrandListedAfterTheLanding`: delivery through a real creel store to an orch listed after the landing. `TestLandNotices` uses a fake `Notice` and asserts lines, clears and flags only.
- `internal/fabriccli/cli_test.go` `TestRunCLI_CloneOfAnUnmigratedWeftLeavesTheMarkerToTheWalk`: the marker is tracked, the walk commit's subject, and the stripped `fabric.yaml`.
- `TestRunCLI_ReconcileBacksFillsWarpBinding`: `warp_binding` is recorded and the binding file is tracked again.

**F7 — LOW — design (test economy) — `internal/fabricengine/snapshot_integration_test.go:47`, `:73`, `:169`, `:328`**
- Summary: the `TestSnapshotWarpSHA_*` tests mostly assert the tag-match loop of `SnapshotWarpSHAForTest` (`export_test.go:445`), which is now test code.
- Only `commit_integration_test.go:263` uses it as an oracle over live writes. The live part these tests reach is `scanWarpSHATrailers`'s ordering.

**F8 — NIT — consistency — test names across packages**
- Summary: test names keep the removed seam's name.
- Examples: `TestRunCLIIn_FromNonGitDirectory` (`configcli_test.go`), `TestRunCLI_GroupGuard_NoGitRepoNeeded` (battencli), `TestRunCLI_GroupGuard_OutsideGitRepo` (burlercli), `TestRunCLI_*` (fabriccli, idecli, reedcli, quarrycli and others). Each names a function that no longer exists.

**F9 — NIT — consistency — `internal/preflightshed/preflight_test.go:28-49`**
- Summary: `TestNewPreflight_CarriesToldName` names and documents the test-file helper `NewPreflight` as its subject. What it pins is `NewPreflightWithChecks`'s field assignment, and its doc and keep reason should say so.

## Tool calls
1. Grepped `cmd/lym` for `run(` called with distinct writers, `configcli` for `resolveReal`/`realSync` uses, and read `hibernatedaemon_test.go` lines 15-60, `main_test.go` lines 20-70 and `exitcode_test.go` lines 75-130. The context lacked the split-writer callers, the sync coverage, the rewritten hibernate test body and the contained cmd/lym tests' assertions.
2. Printed the contained tests in battencli, burlercli, dobbyengine and fabriccli, plus `TestCancelCmd_RefusesBeforeAnything`, to compare assertions as the rubric requires before filing.
3. A `git grep -A40` for `TestLandNotices`. It failed on a bad argument and returned nothing.
4. The `TestLandNotices` body via awk, to compare its assertions with the contained creel test.
