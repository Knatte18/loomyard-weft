```yaml
lens: code-tests
outcome: reported
```

**Task:** for each behaviour the diff adds or changes, name the test that pins it, or none.

**Behaviour → pinning test**
- **Detached run, join and replay** (`TestGateTest_DetachedRun`, `detachfront_integration_test.go`).
  - A run that outlasts the cap answers `done: false`. The next call prints its result, and a third call starts afresh (line 316).
  - A call joining a run still waiting for slots names the holders (342).
  - A changed dirty-path set (377) or changed HEAD tree (404) starts a fresh run and ends the superseded runner.
  - A recorded exit 75 replays as exit 1 with `replayedBusyLine` (426).
  - A runner killed without an exit file gets a fresh run (439).
- **Start failure and claim ordering** (same test).
  - A runner that never starts exits 1 with its log and an expired lease (462).
  - A failed launch exits 1 at once (489).
  - A late claim within the bound still answers inside the cap (523).
  - A held key lock past the margin answers `done: false` in the phase `starting` (537).
  - The runner's refusals (no lease, passed lease, process file already claimed, malformed request) are in `TestGateRunner_Refusals` (`detachrun_test.go:17`).
- **Join predicate and Windows start reading:** `TestJoinable` (`detachfront_test.go:17`, with a Windows start-reading row at line 36).
- **Lease behaviour**
  - A passed lease abandons the run (`detachfront_integration_test.go:643`).
  - The lease moves to the later deadline when a short-cap front joins a long-cap run (718 and 744).
  - The runner's own lease and signal paths: a lease that passes while a tier runs (`test_integration_test.go:2044`), in the slot wait (2059), and a SIGTERM giving 128+15 with the abandoned mark (2081 and `detachfront_integration_test.go:690`).
  - A SIGTERM to the front exits 143 while the runner goes on (664).
- **`--wait`, `RunGate`, no worktree, resolver fallback, outside a hub:** `detachfront_integration_test.go` lines 562, 589, 630, 575 and 604.
- **Registration under the task slug with `RoleGate`:** line 797. This also covers the prime registering nothing and teardown ending the runner.
- **D5, long wait and call id**
  - `TestLongWait` and `TestChooseCap` (`detachfront_test.go:159` and `:265`).
  - The end-to-end long cap is at `detachfront_integration_test.go:770`.
  - `CallID` and `Abandoned` on records are asserted at `test_integration_test.go:1343` and `:1429`, and at `:2022` for the lease path.
  - `lym gate report` counts abandoned records apart in `report_test.go:33`, and `runlog_test.go` covers old records without the new fields.
- **D6 card-verify** (`detachfront_integration_test.go:834-964`)
  - Phases: running (834) and slots with holders (878).
  - `4` and `4-<slug>` join one run (851).
  - A lease cancels a card line that holds a slot (906) and a `--wait` gate line through its child (921).
  - A plan edited between the front's read and the runner's is refused and replayed (942).
  - A dead runner's line group is killed (964).
  - `cardverify.Runner`'s context, phase, tied tree and strand drop are at `runner_test.go:46` and `:195-269`.
  - `verifyrun.RunTied` is covered in `run_tied_integration_test.go`.
- **Strand dropped in `battencli`:** `TestCommandRunners_DropTheStrand` (`deploy_integration_test.go:61`).
- **D4 resolver and validation**
  - `TestKeepWarm` covers the numbered role, prime strand, unmatched suffix and fallback.
  - `TestKeepWarmCaps` (`prepare_test.go:227`) covers the cap bounds.
  - `CheckLoad` naming `lym config claude` is at `shuttleengines_test.go:373`.
  - `TestLoadConfig_KeepWarmMaxReruns` covers the shuttle key.
- **`lym shuttle await` cap:** `TestAwaitCap` (`cli_test.go:788`) covers the explicit cap, a worktree, no worktree and the fallback.
- **D7 `AgentGateCall`:** `config_test.go:759-781`. The rendered card, batch and `webster-fix` calls are covered by the changed `cardgate_test.go` and `verifyfixer_test.go`.
- **D9 shellguard rule:** `rows_test.go:38-52`. Stencil prose is exempt.

**Findings**

1. **LOW, design.** No test pins a zero resolved cap running in-process.
   - Location: `internal/gatecli/detachfront.go:247`, where `answer.Cap <= 0` returns not capped.
   - Evidence: the `TestGateTest_DetachedRun` subtests listed above have no zero-cap row, and the seam in `detachfront_integration_test.go` has no such case. Deleting the branch would leave every test green.

2. **LOW, design.** No test shows that an inherited slot under a strand runs in-process (D2).
   - Location: `internal/gatecli/detachfront.go:239`.
   - Evidence: `test_integration_test.go:1194` covers the inherited slot, but I did not find it setting a strand. `detachfront_integration_test.go` has no inherited-slot row.
   - Without a strand the call is uncapped anyway, so the branch can be removed with all tests green.

3. **LOW, design.** The stdout-then-stderr copy order in the replay (D1) is not asserted.
   - Location: `internal/gatecli/detachfront.go:503-505`.
   - Evidence: the replay tests at `detachfront_integration_test.go:316` and `:426` check content and exit code. I found no assertion on the order of the two streams, which the harness's separate stdout and stderr buffers could not show anyway.

4. **LOW, design.** No test pins the delivered mark's write-after-successful-copy rule (D1, focus).
   - Location: `internal/gatecli/detachfront.go:503-508`.
   - Evidence: `delivered` appears in tests only as a `joinState` field (`detachfront_test.go:105`) and through the "third call starts a fresh run" path. No test makes the copy fail and checks that the run is not marked delivered.
   - The code marks it delivered regardless of copy errors, so a test would also expose that.

5. **LOW, design.** The runner's Info spawn log and the Windows `proc.HoldNamedJob` hold in `runCard` have no assertion (`detachfront.go:128`, `cardverify.go:373`).
   - Evidence: the registration test at `detachfront_integration_test.go:797` covers the record but not the spawn log. No test touches the job hold.

**Tool calls (2)**
1. `grep` of test names across the touched test files, to list the subtests. My context held only the diff of the non-test sources.
2. `grep` for `Abandoned`/`CallID`, `startReading`, `ValidateKeepWarm`, the numbered role, `RoleGate`, `AgentGateCall` and the `TestAwaitCap` body, to confirm which behaviours have a test row. My context lacked the test sources.
