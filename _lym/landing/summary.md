# Darn verify gate: stop at first failed tier, rerun failures, time budget

The darn verify gate, which Darn-Write runs and which Code-Burler-Fix also runs in a darn run, used to spend a full pass of every tier on each attempt.
It also kept going past a failed tier, and nothing told anyone when it ran long.
A step whose gate never re-prompted halted with `gate did not pass after 0 attempts`, which read as a spent budget.
This change addresses all four.

## Stop at the first failing tier

`lym gate test` gains `--stop-on-fail`.
With it, a call ends at its first failed run: no later tier, vet or guard run starts, and a stderr line names the failed run and every run it skipped.
A run that crashed its test runner twice judged no code, so it stops nothing.
The key's lint and report steps still run, and the failure report and closing line follow as before.

Running every tier after a failure is deliberate for a plain call, as `internal/gatecli`'s doc says, so a failure already present at the base hides no later check.
So `--upto` and the landing gates keep that default, and the flag is the separate choice.
The darn wiring (`loomcli.wireTreeScope`) adds the flag to the `darn-write` key's told call and tree-scoped call, through `gateslot.StopOnFailCall`.
Every other key keeps its whole call, so Code-Burler-Fix and the landing gates still run every tier.
The fixer therefore hears of a tier0 failure after tier0, not after tier1 and tier2 have also run.

## Rerun only the failed tests between attempts

Before this change the gate already handed the fixer and the writer findings drawn from the log, not the whole log.
It could not rerun failed tests alone.
Now, in the darn form of `loomshed.NewVerifyGate`, a failed run whose failure report names failed tests alone keeps them in the gate's closure.
The next arrival first runs just those tests through `verifytree.Verify`, with the command `gateslot.FailedTestsCall` renders.
That command holds one `lym gate test --wait --stop-on-fail --tier N <packages> -- -run '^(<tests>)$'` call per tier, joined by `&&`.
- Tests that fail again fail the arrival with their findings alone, prefixed with a note, so the fixer and the writer get only those.
- Tests that pass alone hand the arrival to the full command, which runs once.
- A failure with no test to name keeps nothing, so the next arrival runs the full command at once.
  That covers a build error, a failure outside any test, a vet or lint step, a timeout, a runner crash and a cut report.

`gateslot.FailedTests` reads the failure report through `gateslot.FailureReportHeader`, which `gatecli` now takes its header from.
It reports false unless every failure in the report is a named test of a tier or guard run.

## A time budget

`darn.yaml` gains `verify_budget_min`, defaulting to 30, with category None.
`loomshed.VerifyBudget` counts from the gate's first arrival.
Once it is spent, the gate sends one line to the run's parent orch through `orchcli.NotifyParent`.
The line names the gate, the budget, the current attempt and the latest failed run with its log.
It goes out at an arrival or, through a timer armed only while a verify command runs, during a long run.
So it never fires after the gate stopped running.

## A truthful halt when no re-prompt was sent

Shuttle's gate loop could finalize while the failing entry still had re-prompts left.
That happens when the re-prompt could not be sent, when the pane showed no input box past the bound, or when an entry's wait ran out.
It also happens when the session ended or the deadline passed first, as when a resumed step attaches to a finished session.
`GateOutcome.EndedEarly` now carries that cause.
`shedadapters.gateFailedReason` uses it instead of the attempt count.
With no re-prompt sent, the reason reads `gate failed its first check and the session was never re-prompted, since <cause>; findings: <path>`.
With some re-prompts sent, it reads `gate did not pass after N attempts and the run ended with re-prompts left, since <cause>`.
A spent budget keeps the old wording.
The attempt count stays the number of re-prompts sent, which is what was made.

## Design decisions

- A spent budget only raises a notice and stops no attempt.
  `verify_attempts` already bounds the attempts, so stopping on time as well would cut a run that is close to passing.
  The orch can still pause or cancel the run once it sees the notice.
  The key is classified `CategoryNone` because it gates nothing.
- The rerun lives in the gate, not in `lym gate test`.
  The run records are a domain log no run reads to proceed, so the gate reads the failure report of its own verify log.
  It keeps the tests in its closure, so a restarted driver starts the budget clock and the kept tests afresh.
- The rerun call is the one place a site spells tiers and targets.
  `gateslot.FailedTestsCall` renders it from the gate's own report, and `pattern/PATTERN-gate-slot.md`'s Verified tree section now says so.
- The flag is added in the darn wiring, so `NewVerifyGate` still runs its told command as it is.
  Each pass is recorded under the call that ran.

## Tests

- `gateslot.TestFailedTests` and `TestFailedTestsCall` (tier0) cover the report read and the rendered call.
- `loomshed.TestVerifyBudget` (tier0) covers when the notice goes out.
- `loomshed.TestVerifyGate_DarnFormRunsTheFailedTestsAloneFirst` (tier1) drives the darn form past a failed run with a stand-in `lym`.
- A new `TestGateTest_Scenario` subtest (tier1) covers `--stop-on-fail`.
- New rows cover the reason wording, and `TestGate_RepromptOverPicker` now checks `EndedEarly` for the no-input-box bound.
