```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Behaviour → pinning test**

| Behaviour | Pinning test |
|---|---|
| Row wrap, pass, pending or terminal result spawns none | `TestWrapRowGate/InnerOutcomesThatSpawnNothing` (`rowgate_test.go:170`) |
| Row wrap, failure spawns one fixer, re-runs and returns the re-run | `FailureSpawnsOneFixerWaitsAndJudgesAgain` (`rowgate_test.go:203`) |
| Re-run failure carries the new findings plus the reply or the dead note | `ReRunFailureCarriesNewFindingsAndTheFixersAnswer` (`rowgate_test.go:226`) |
| Forked evaluation skipped when it fails only on the lens review | `ForkedFailureSpawnsUnlessSkipForkedHoldsIt` (`rowgate_test.go:262`) and `TestLensReviewOnly` (`loomshed/gates_test.go:1109`) |
| Spawn error leaves the inner failure standing | `SpawnErrorLeavesTheInnerFailureStandingWithNoRecord` (`rowgate_test.go:297`) |
| Cap of attempts plus one fixers per visit | `ASetAtAttemptsPlusOneSpawnsNone` (`rowgate_test.go:316`) |
| Live record waited on instead of spawning | `ALiveRecordIsWaitedOnAndTheEntryJudgesOnce` (`rowgate_test.go:329`) |
| Wait error stops the strand and marks it dead, or keeps it live on a failed stop | `AWaitErrorStopsTheStrandAndMarksItDead` (`rowgate_test.go:363`) |
| `Final` never spawns and only waits | `FinalNeverSpawnsAndWaitsOnlyForALiveRecord` (`rowgate_test.go:403`) |
| Other entries wait on a live fixer | `OtherEntriesWaitForALiveFixerBeforeJudging` (`rowgate_test.go:447`) |
| `Expired` probe | `ExpiredIsTrueOnlyForALiveRecordAFreshWrapDidNotSpawn` (`rowgate_test.go:492`) |
| Entries keep their other fields, and refusals | `rowgate_test.go:527`, `rowgate_test.go:540` |
| Row store (round trip, tolerant of unknown keys, `Live`, `markTerminal`) | `TestRowStore` (`rowstore_test.go`) |
| `SpawnOrAttach` (attach, spawn and register, nil register, role refusal) | `TestSpawnOrAttach` (`spawn_test.go`); the spawn-failure and abandon paths I did not locate by name |
| `Await` classification, stop on every terminal result, bad record | `TestAwait` (`await_test.go:31`, `:133`, `:156`) |
| `Registrar` | `TestRegistrar` (`registrar_test.go:47`) |
| `IsFixerRole`, including numbered roles | `TestIsFixerRole` (`roles_test.go:13`) |
| `RenderPrompt` | `TestRenderPrompt` (`prompt_test.go:37`) |
| `StrandLive` and `TurnEndedAfter` | `strand_test.go:19`, `:76` |
| D8 visit key (stable, new on append, new on fold into the newest entry, missing status file) | `TestRowVisitDir` (`loomcli/gatefixer_test.go:92`) |
| `gateFixerSeam` (empty slug, wrapped fixer entry, unknown alias, missing timeout) | `TestGateFixerSeam` (`loomcli/gatefixer_test.go:137`) |
| Fixer launch per body, and a body-less entry refused | `TestFixerSpec` (`loomcli/gatefixer_test.go:201`, `:256`) |
| `writerFixerOn` | `TestWriterFixerOn` (`loomcli/gatefixer_test.go:50`) |
| Goto ends each fixer role, and the guard runs only after `inner` admits | `TestEndGateFixers`, `TestFixerEndingGuard` (`fixerendguard_test.go:24`, `:85`, `:125`) |
| `fixer` element: parse, empty value turns it off, refusals | `TestGateFixerElement` (`shedrecipe/entries_gatefixer_test.go:22`) |
| Opt-in of the three rows | Added tests in `entries_darnwrite_test.go`, `entries_planwrite_test.go`, `entries_discussionwrite_test.go` (diff stat only; I did not open them) |
| gate.yaml `fixer` on the card key only, unparsable value, `fixer_timeout_min` of at least 1 | `TestLoadConfig` rows (`gateslot/config_test.go:180`, `:187`, `:190`, `:191`) |
| Module keys default to `sonnet[medium]` and empty turns off | `darnengine/config_test.go:36`, `:101`, `:106`; `configreg_test.go:79` |
| Card gate hand-off rule (role guard including `gate-fixer-2`, no slug, a key without `fixer`, wrong verb) | `TestHandsOff` (`cardfixer_test.go:26`) |
| Card gate green, red-fix-rerun, dead note, spawn error, turned-down request, live-record attach before the gate, stop on failed attach, failed stop, lapsed lease stops the fixer | `TestRunWithFixer` (`cardfixer_test.go:235`–`:430`) |
| Tree-identity rewrite at re-run start | `TestRunWithFixer/RedCallIsFixedAndRerun` (`cardfixer_test.go:289`) |
| Fixer-phase join over changed dirty paths | `TestJoinable/FixingRunUnderChangedDirtyPaths` (`detachfront_test.go:84`) |
| Fixer strand's call keyed apart, and role parsing | `TestCallKeyDir` (`detachfront_test.go:404`) |
| Webster fork and recovery card-fixer text | `template_test.go:332`, `:366`; `TestCardFixerOn` (`webstercli/cardfixer_test.go:12`) |
| Webster `fix-verify` on the core and its pair slug | `TestFixVerifySpawnOrAttach`, `TestFixVerifyAwait` (`verifyfixer_test.go:58`, `:225`); `TestWebsterCLI_PairSlug`, `TestFixVerifyCmd` (`webstercli/fixverify_test.go`) |

**Findings**

1. **Severity LOW, class design.** The front's write of the role into the run request has no direct test I could find.
   - Location: `internal/gatecli/detachfront.go` `writeNewRun`, `Role: f.call.role`.
   - I found `TestCallKeyDir` (role parsing and keying), `TestHandsOff` (the guard over a request that already has `Role`) and `cardfixer_test.go:239` (which builds the request by hand). None of them pins that `writeNewRun` puts `call.role` into the request. If it were dropped, `handsOff` would treat a fixer's own call as having no role and hand it off, a spawn loop.
   - Evidence: `detachfront_test.go:404`, `cardfixer_test.go:39`–`:43` and `:239`. I did not open `detachfront_integration_test.go`, which may cover it.

2. **Severity LOW, class design.** I found no test that the three goto sites wire `fixerEndingGuard` with `endRunFixers`.
   - Location: `internal/loomcli/drivewiring.go:281`, `arm.go:464`, `arbiterconsume.go:28`.
   - `TestFixerEndingGuard` pins the guard in isolation. `wiring_seams_test.go` changed by 5 lines, and I did not open it, so it may pin the wiring.
   - Evidence: grep of the three call sites.

No other uncovered behaviour surfaced. D8 cases (crash re-step, goto, resume of a halt that appended history, fresh entry) and the `attempts + 1` cap are each pinned. I flag no duplicated coverage: the row-wrap, store, config and card-runner tests each cover distinct behaviour.

**Tool calls (2 used)**
1. A grep of test names across the diff's main test files, to get per-behaviour coverage. The manifest listed no test files.
2. A grep for `fixing`, `LensReviewOnly`, the goto-guard wiring sites, request `Role` use, tree rewrite, card-fork tests and config defaults in tests, to settle the uncertain rows.
