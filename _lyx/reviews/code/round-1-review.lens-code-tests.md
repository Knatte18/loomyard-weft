```yaml
lens: code-tests
outcome: reported
```

**Lens code-tests: behaviour → pinning test.** I found nothing to report at BLOCKING or MEDIUM. I read test names and a few bodies, not every assertion, and I ran no tests.

| Behaviour | Pinning test |
|---|---|
| D8 guard admits the prime and a marked beam, and refuses a non-beam task pair, naming the nearest beam or the prime | `TestResolveBattenPrime` rows `PrimeReturnsItself`, `MarkedBeamReturnsThePrime`, `PairForkedFromThePrimeNamesThePrime`, `PairForkedFromTheBeamNamesTheBeam`, `PrimesRecordsSiblingIsStillRefused` |
| Guard refuses a beam held by its landing, naming the landing and its parent | `TestResolveBattenPrime/BeamHeldByItsLandingNamesTheLandingAndItsParent` |
| `status`, `arm`, `cancel`, `cleanup` run from a beam, and refusals from a non-beam pair | `TestVerbs_RunFromABeam` |
| `run`, `step`, `goto` and `cancel --remove` refused from inside the slug's own pair | `TestCancelCmd_RemoveRefusesFromInsideTheBeam`; run/step via `RunOfTheBeamFromInsideItRefusesBeforeAnyWrite`. No named test for `goto`, which is minor. |
| Worktree-Create forks from the beam, forks from the prime for a grouping project, and works with the prime dirty | `TestWorktreeCreate_ForksFromTheBeam` |
| Precondition names `lyx beam create` or `lyx adopt` by whether the branch is on origin, and names the landing when the beam is run-held | `TestBeamForkPrecondition` |
| `ErrBranchExists` kept for batten | `TestWorktreeCreate_ForksFromTheBeam/LocalBranchLeftFromARemovedPairIsRefused` |
| Start guard: dirt exempt for a beam member, still refused for a prime task, promoted-pair refusal, project/beam refusals, `reconcile` text | `startguard_test.go` rows |
| A landing passes the guard and reaches the Worktree-Create skip with the prime dirty | `TestBeamLanding_ReachesWorktreeCreateSkip` |
| `beam land` refuses on open members at depth, child pairs, live orch or watcher | `TestBeamLandRefusal` (unit), `BeamWithMembersIsRefusedNamingEachOpenEntry` (integration), `TestLand` (`beamcli`, pass-through) |
| Window opens in the parent's session | `TestWindowSite` |
| Notice walk and ready-notice routing | `TestNoticeSite`, `TestNoticesReachDriversParent`, `TestReadyRoutes`; `TestLand_ReadyNotice` and `TestLand_ReadyRouteResume` exist in `battenshed` |
| `NearestLiveOrch` walk (live, stopped, cycle, gone parent, error) | `nearestlive_test.go` rows |
| `ParentOf` reads the origin record, empty reads as the prime, no fall-through, prime renamed | `parentof_integration_test.go` and `UnspawnedBeamMemberReadsItsBeamsOwnerElsewhere` |
| `Add` with `Parent`, refusals before any mutation, dirt rules | `TestAdd_FromParentPair` |
| `AdoptLocal`: adopts, refused without the flag, branch checked out elsewhere (warp and weft), default branch recorded as a root pair | `TestAdd_AdoptLocal` |
| Weft reset executor | `TestResetRecordsToParent` |
| `beam create` steps, skips and refusals; `promote` refusals, step ordering and re-runs; `land` refusals | `TestCreate`, `TestPromote`, `TestLand`, plus `TestCreate_StepFailureNamesTheRerun` and `TestCreate_RunsFromABeamAndReachesTheVerbsOwnRefusal` |
| `EndRunForPromote` and `ClearRunForPromote` | `TestEndRunForPromote`, `TestClearRunForPromote` |
| Orch in a beam, orch refusals, tender and overview reading every orch | `TestOrchInBeam`, `TestOrchIntegration_Refusals`, `TestOrchLiveFrom`; `tenderengine/resume_test.go` and `pause_test.go` changed |
| Recipe seeding for a project entry | `TestBoardRecipe` |

**Test economy**
- **LOW, duplication risk.** `internal/battencli/beamland_integration_test.go` (`BeamWithMembersIsRefusedNamingEachOpenEntry`) repeats part of what `TestBeamLandRefusal/EveryMemberIsNamed` and `startguard_test.go` `BeamWithWorkRemainingRefusedByTheLandRefusal` cover. It earns its place only if it pins the real wiring (`hubStartGuard` to `BeamLandRefusal`) rather than the refusal text. Class `consistency`. The unit-level duplicate is `TestBeamLandRefusal`; I did not confirm what the integration assertions check.
- **No finding.** `TestEndRunForPromote` and `TestClearRunForPromote` cover new entry points over the cancel phases. The existing cancel tests cover the same phases, but the new rows pin the promote-specific rerun text and the end-only error, so I rejected this as a duplicate.

**Gaps**
- None confirmed. No test covers `refuseOwnPair` for the `goto` verb, though `arm.go` includes it. That is minor, so I record it here and do not file it.
- I did not check test coverage for D1–D3 board validation, D12 workspace files, D14 or D15. The new files `slugrule_test.go`, `project_test.go`, `parentbeam_test.go`, `hubworkspace_integration_test.go`, `markbeam_integration_test.go` and `render_test.go` exist, so a gap is unlikely.
- The umbrella grep over `internal`, `contracts`, `docs`, `PATTERN.md`, `pattern` and `plugins` matched only `internal/boardengine/rename_test.go`. That is outside my lens and I did not check whether the match is a new or pre-existing line.

**Tool calls (3 of 6)**
1. `git diff -U0` over the new test files: to list added test names and rows, which the manifest lacked.
2. A grep for `refuseOwnPair`, `AdoptLocal` and `ErrBranchExists` in tests, plus a `func Test` listing: to map behaviour to test functions the manifest lacked.
3. A grep of the `hubgeom`, `orchcli` and `tenderengine` test names, plus an `umbrella` grep: to check the remaining coverage and the focus directive's naming rule.
