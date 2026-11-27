---
verdict: BLOCKING
findings:
  - id: R1
    severity: BLOCKING
    class: design
    location: _lym/plan/11-kwinplace-place-flow.md:72
    summary: "Under mode `off` `reportUpdateFrom` reports the placed window's landed desktop as empty, since nothing moved, so `updatePlaces` (card 9) stores an empty last desktop and gives no home, against D1 and D9; the function also has no input that carries the tiling output name it is told to record."
    fix: "In card 11, change the signature to `func reportUpdateFrom(snap Snapshot, tilingOutput string, report ApplyReport, places Places, req Request, plan Placement, fallback bool, now time.Time) ReportUpdate`. State the two fields per path: after an apply, the landed desktop is the plan's target and the output is `tilingOutput`; under mode `off`, the landed desktop is the one the placed window sits on in the locked snapshot (empty when it sits on several or all) and the output is that window's snapshot output. In card 9, replace the parenthesis on `ReportUpdate`'s landed desktop that says it is empty when nothing moved with this rule, and say in `updatePlaces`' first rule that an empty landed desktop settles the entry and keeps its `LastDesktop` and `Home`. Extend the `TestFlow_Place` mode `off` row to assert that the entry's `LastDesktop` and `Home` are the placed window's snapshot desktop and its `Output` that window's output. The signature change was checked with `lym loom validate-plan --overlay` together with R2's and R3's target changes, with no new blocking finding."
  - id: R2
    severity: BLOCKING
    class: consistency
    location: _lym/plan/08-kwinplace-script-runner.md:116
    summary: "Card 8's contract test skips its `Apply` step unless told to run it, a skip by design that `PATTERN-test-speed` forbids (the one exception is the tier selector's own skip); the card also names no way to tell it."
    fix: "In card 8, move the `Apply` step into its own `tier3` test in `internal/kwinplace/contract_test.go`, added under Create as `plan:internal/kwinplace#TestContract_TheRealKWinAppliesAsRecorded` -> `func TestContract_TheRealKWinAppliesAsRecorded(t *testing.T)`, with no skip and no flag. Say that its doc comment states it moves a throwaway window, and that the operator runs or approves it by name (`lym gate test --tier 3 ./internal/kwinplace -- -run TestContract_TheRealKWinAppliesAsRecorded`), while `-run TestContract_TheRealKWinAnswersAsRecorded` runs the steps that move nothing. Change the sentence saying `contract_test.go` holds only the one test to name both. Checked with `lym loom validate-plan --overlay`, with no new blocking finding."
  - id: R3
    severity: MEDIUM
    class: design
    carries: F5
    location: _lym/plan/12-place-window-child.md:42
    summary: "Every non-prime placement child is still registered under its worktree's slug, beyond D10's registration only when the opening verb runs inside a run; the fixer deferred this saying lym has no signal for that, but `pauserecord.RunProcessEnv` marks a run command and every process it starts, the batten Open-Window row's `lym reed window` included."
    fix: "In card 12, have `Spawn` register the child only when the worktree is not the prime and the calling process's environment holds a non-empty `pauserecord.RunProcessEnv`, as D10 says; add `internal/pauserecord#RunProcessEnv` and `internal/pauserecord#RunProcessMarked` to the card's Uses. Rewrite the Shared Decision run-registration in `00-overview.md` to the same rule (a child opened outside a run belongs to none and ends within its own bounds), and keep the reader analysis for the registered children. In `TestPlaceWindowChild`, every row sets the variable itself with `t.Setenv`: `TaskWindow` marks it and asserts the one `RolePlacement` record, and a new row on a task of its own sets it empty and asserts the marker appears and no `RolePlacement` record exists. The Uses additions were checked with `lym loom validate-plan --overlay`, with no new blocking finding."
prior:
  - id: F1
    status: resolved
  - id: F2
    status: resolved
  - id: F3
    status: resolved
  - id: F4
    status: resolved
    answer: "Settled by the added entry \"A first card runs the KWin probes\"; card 1 carries it, and cards 6 and 8 only use its recordings."
  - id: F5
    status: open
    answer: "The reader analysis is in card 12, but registration still goes past D10. The deferral's premise does not hold: `internal/pauserecord/hold.go` declares `RunProcessEnv`, `tendercli.InstallNetworkGate` sets it for every run command, and batten's row runner passes it on. So the fixer can limit registration as D10 words it, with no parent decision. A child opened outside a run is then unreached by a teardown, which is what D10 settles."
  - id: F6
    status: resolved
  - id: F7
    status: resolved
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: resolved
  - id: F12
    status: resolved
  - id: F13
    status: resolved
  - id: F14
    status: resolved
  - id: F15
    status: resolved
  - id: F16
    status: resolved
  - id: F17
    status: resolved
  - id: F18
    status: resolved
---

The round-1 fixes hold, except F5's registration scope.
Two of them brought in a new defect each: the report builder of F3 loses the place under mode `off`,
and the contract-test extension of F18 skips by design.

### [BLOCKING] Mode off records no desktop for the window it was to record

**Location:** `_lym/plan/11-kwinplace-place-flow.md:72`

**Issue:** Card 11 builds the landed desktop as "the plan's target, `""` when nothing moved", and under `off` the plan is zero.
Card 9's `ReportUpdate` says the same, and `updatePlaces` sets the placed window's `LastDesktop` to where it landed,
which also becomes its home when it had none.
The only `RecordReport` path with nothing moved is mode `off`,
so a first window under `off` is stored with no last desktop and no home.
D1 says that run records the window's place, and D9 says its last desktop is where it landed.
D8's rationale relies on it as the parent's anchor once the parent window is gone.
The next report that shows the window also reads the empty value as a hand move.

The same bullet says the tiling output comes from "the request, the plan and `now`".
Neither `Request`, `Placement`, `Snapshot` nor `ApplyReport` carries it, and the signature takes nothing else that does.

### [BLOCKING] The contract test skips its apply step by design

**Location:** `_lym/plan/08-kwinplace-script-runner.md:116`

**Issue:** "the test skips it unless told to run it".
`PATTERN-test-speed`, "No test skips by design": a skip names a missing prerequisite of the host,
and the tier selector's skip is the one exception.
`tier3` is already run by hand, one named test at a time, so a separate test gives the operator the same choice with no skip.

### [MEDIUM] Placement-child registration still goes beyond D10

**Location:** `_lym/plan/12-place-window-child.md:42`

**Issue:** D10 registers the child "when the opening verb runs inside a run".
The plan registers the child of every worktree but the prime, those of `lym machine start` and a hand-run `lym reed window` included.
`shedverbs`' `run` and `step` carry `clihelp.RunProcessAnnotation`, `tendercli.InstallNetworkGate` then sets `LYM_RUN_PROCESS`,
and `battencli`'s `runRowCommand` hands the environment to the row command, so the opening verb can tell.
Reed strips the mark from pane environments, so a hand run in a pane reads as outside a run.

## Focus questions

- Places-file snapshots: `RecordReport` now reads only a locked snapshot, mode `off` included, and its interleaving has a row.
  `Settle` may take the window-wait snapshot, which D9 allows, and writes its own entry alone.
- Prime start desktop: both modes yield the current desktop; `TestPlan` and `TestFlow_Place` pin `next`.
- Report update: `reportUpdateFrom` and `CurrentOrder` carry it, with the two field gaps of R1.
- KWin facts: card 1 observes each Open-risks fact, and its halt and operator step are what the added decision asks for.
- Registration: R3. The four readers card 12 lists read as it says.
- Bounds: every exec-seam call is bounded, and a timed-out load has a row that asserts unload and release.
- Resolution-failure rows: both inputs reach their branch.
  `configengine.ValidateMachine` checks keys and shapes, not values, so the refused machine value can be set.
- Exit paths: cards 11 and 12 now match D12.
- Places file: the pid-reuse and concurrent-heal cases keep D9's guarantees.
- Test files and recordings: each `tier1` and `tier3` test has a file of its tier.
  The contract test compares nothing with `neve-query-no-output`, `neve-apply-one-row` or `neve-kscreen-laptop-off`.
  The stand-in answers from none of the three, and they are reader fixtures of the `tier0` walk, so I raise no finding.

## Notes below the incremental bar

Seen on unchanged text or below `BLOCKING`, so not raised as findings.

- `Anchor` answers an empty desktop with `true` for a settled parent that has no live window and no last desktop,
  such as one settled after a window timeout.
  Neither D8 nor card 10 says where the child then starts.
- Card 8's `Query` reads `kwinquery.json` under the script lock, and `run` takes that same lock itself with no hook for the read.
- Card 1 does not require a tiled window for `neve-query` or a bottom-row zone for `neve-apply`,
  which card 6's assertions over those recordings need.
- Card 12's window command waits while `PATH` holds only `sh` and `git`, so it must name `sleep` by absolute path.

## Constraint and record

The record asks for the placement rules under tier-1 tests, and the plan puts them at `tier0`.
`PATTERN-test-speed` puts a behavior at the lowest tier its real modules run in, so the constraint wins.
