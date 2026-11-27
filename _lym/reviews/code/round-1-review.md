---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/battencli/lifecycle_integration_test.go:404
    summary: The new CommandRows group subtest calls t.Parallel although its step stepCommandRows_IgnoreACommandSetInTheTasksTrackedLayer redirects the process-global logger through logcapture.Capture, against PATTERN-test-speed's parallel rule and the repository's logcapture convention.
    fix: "Take the CommandRows_IgnoreACommandSetInTheTasksTrackedLayer row out of the CommandRows group in TestBattenIntegration_Rows. Add a top-level test TestBattenIntegration_CommandRowsIgnoreACommandSetInTheTasksTrackedLayer in lifecycle_integration_test.go. It calls tiergate.Require(t, 1), builds h := hubforge.CopyHub(t, hubforge.Shape{Anchor: \".\"}), and runs posixOnly(stepCommandRows_IgnoreACommandSetInTheTasksTrackedLayer) on h, returning at once when that is nil. It never calls t.Parallel and carries the comment \"It does not call t.Parallel: logcapture redirects the process-global logger.\", the wording overview_test.go:694 uses. Keep every assertion of the step unchanged."
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/battencli/lifecycle_integration_test.go:347
    summary: The doc comment says a step that leaves the hub changed for good runs last in its group, but stepSeedChild_IgnoresPrimesCommittedBattenRecords, stepMarkWatched_HoldsTheMarkerOnlyWhileItsNoticesReachTheDriversParent and stepCreateRow_LeftoverBranchIsRewordedForPrime each leave the hub changed and run before other steps of their group.
    fix: "Replace the doc-comment line at lifecycle_integration_test.go:347 with \"A step that leaves the hub changed in a way a later step would trip on, such as the dirtied prime, runs last in its group.\""
  - id: F3
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md#Measurements
    summary: Measurements names neither the command and tier of its six runs nor the set-up difference behind the 33 to 75 s it compares against; it blames the machine, though the report it cites measured lym gate test --tier 1 at a 1-minute load average of 23 to 67 with other gate slots held.
    fix: "In _lym/landing/summary.md under ## Measurements, replace \"`go test -count=1` reports for the integration test run\" with the exact command the six runs used, naming its tier. Replace the sentence \"The machine ran these well below the 33 to 75 s the earlier measurement recorded, so the absolute figures differ from it.\" with a sentence that names the set-up difference. The earlier report's tier1 calls ran under a 1-minute load average of 23 to 67 with other gate slots held (docs/research/speed/test-time-measure-2026-11-27/report.md, Machine and load). Give the load these six runs ran under."
---

### [BLOCKING] A parallel group holds a step that takes over the process-global logger

**Location:** `internal/battencli/lifecycle_integration_test.go:404`, the `CommandRows_IgnoreACommandSetInTheTasksTrackedLayer` row, and the group subtest's `t.Parallel()` at `:413`.

**Issue:** `stepCommandRows_IgnoreACommandSetInTheTasksTrackedLayer` calls `logcapture.Capture` (`:1395`).
That function sets the process-wide logger output and resets output and verbosity in cleanup (`internal/testkit/logcapture/logcapture.go:47-51`).
`PATTERN-test-speed`, "Tests run in parallel" (`pattern/PATTERN-test-speed.md:223-224`), says a test that touches process-global state does not call `t.Parallel`, and a comment names that state.
The package already follows this for every other `logcapture` user (`overview_test.go:694`, `codetipsettle_test.go:22`).
The diff adds a new `t.Parallel` site over the step, so six other groups now log into the step's buffer and have the logger reset under them by its cleanup.
Its rewritten doc comment names no such state.
The top-level test called `t.Parallel` before the diff, so the conflict predates it.
The diff widens it and writes new lines that break the listed constraint, and the constraints section makes every such violation BLOCKING.
Today the step's `Contains` assertion still passes (code-correctness lens), so the defect is the rule, not a failing run.

### [LOW] The doc comment's ordering rule is broader than the table keeps

**Location:** `internal/battencli/lifecycle_integration_test.go:347`

**Issue:** Several steps run before other steps of their group but leave the hub changed:
- `stepSeedChild_IgnoresPrimesCommittedBattenRecords` commits batten records on the prime (`recordleak_integration_test.go:53-60`).
- `stepMarkWatched_…` leaves an orch strand recorded on the prime (`:1823`).
- `stepCreateRow_LeftoverBranchIsRewordedForPrime` leaves a prime branch (`:1125`).

No later step trips on any of them, so no result is wrong.
But "A step that leaves the hub changed for good runs last in its group" is false as written, and a later regrouping would follow it.

### [LOW] Measurements does not name its set-up against the figure it is compared with

**Location:** `_lym/landing/summary.md`, `## Measurements`

**Issue:** The board entry's Aim quotes 33 to 75 s of tier1 test time from `docs/research/speed/test-time-measure-2026-11-27/report.md`.
That report measured `lym gate test --wait --no-carry --tier <n> ./<pkg> -- -count=1 -v` on a heavily loaded machine, a 1-minute load average of 23 to 67 at tier1, with up to four gate slots held.
The summary says only "`go test -count=1` ... for the integration test run", which names no tier.
It explains the gap with "The machine ran these well below", which names no set-up difference.
The before-and-after comparison itself is sound: three runs each, from the same set-up.
My own tier1 run of `TestBattenIntegration_Rows` alone gave groups of 1.82 to 2.70 s with `TeardownLocal` longest, which agrees with the table's longest-group column.

## Rejected

- Steps relying on another group's leftovers: every step builds its own precondition on its group's hub (code-tests and code-correctness lenses), and no assertion goes vacuous.
- Lost or duplicated steps: the old 36 rows map one to one onto the new groups' 6+1+3+6+7+7+6.
- Start-cost bound under parallel load: `startCostBound` is 30 s against row work of milliseconds; it stays a gross-regression budget, which `PATTERN-test-speed`'s Time-is-injectable exception allows, and the `testtiming:keep` reason still holds.
- Stale subtest paths: `pattern/PATTERN-pair-remote.md:140` is updated, and no other doc, tool config or `-run` filter names a Rows subtest path; the research reports that name old steps are measurement reports.
- `pairPushCounts`, `spawnPairPush` and `clihelp.ExecuteIn`: mutex-guarded per path, set once in `TestMain`, and cwd passed through the context, so none is shared mutable state across groups.
- Summary's longest-test claim: the parent test's own time leaves out its parallel subtests, so a rows group or `TestCancelIntegration_RemoveOfABeamLeavesAPlainProject` being longest fits the table.
