---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed-differently
    reason: "The proposed blast radius \"dobby, the beam's change description and its pull request on GitHub\" breaks contracts/specs/loom-final-summary-spec.md, which pins the blast radius to one word and has every gate refuse another value. The door is one-way as proposed, the blast radius is the one word `beam`, and the change description's body names the description, its commits and the pull request."
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
---

# Fixer report, round 1

All fourteen findings are fixed.
Tier 0 and tier 1 of `internal/dobbyengine`, `internal/beamcli` and `internal/landingshed` pass, and `go vet` is clean.

## What changed

- **F1, F9** (`contracts/stencils/beam/beam-template-describe.md`): the stencil no longer restates the title syntax, the 72-character cap or the Merge danger format; it points at the final-summary spec and keeps the beam-specific content rules.
  F9 takes the review's stencil alternative: a Rules item has the agent write the draft once, whole, and never edit it after it passes.
- **F2** (`internal/dobbyengine/loop.go`, `describe.go`, `doc.go`): `tick` logs a `describeStep` error through the new `describeFailures` episode and goes on to `tidyStep` and `landBeam`.
  A spawn failure posts one line to the nearest orch, then drops the landings through `endDescribe`; a failed post keeps the landings, so the next tick tries again.
  The spawn-failure table of `TestDescribeStep` asserts the line and the dropped landings.
  `TestRun_DescribeFailureLeavesTheTidyRunning` is the tick-level row: a failing describe step does not hold the tidy report.
- **F3** (`describe.go`): `settleDescribe` returns a draft read error only while the deadline stands; past it the strand stops, the landings drop and the timeout line carries the read error.
  The findings-read subtest advances past the deadline and asserts a later landing starts a second agent.
- **F4** (`loop.go`): the kept agents are the tidy and the describer (`readKeptAgents`, `rememberKeptAgents`, `keptDescribe`), and `exitWorktreeGone` stops both through `PrimeTidyShuttle`.
  A worktree-gone row asserts the describer's strand is stopped and logged.
- **F5** (`internal/beamcli/describe.go`, `dobbydeps.go`, `describe.go`): `BeamDescribeDeps.Record` returns `recorded` beside the error.
  `recordDescription` wraps its two post-record failures in `afterRecordError`.
  `settleDescribe` words a recorded-but-not-updated case with the way forward `lym beam pr update <slug>`.
  The refused-record subtest is a table with a row for each case.
- **F6** (`_lym/landing/summary.md`): the door is one-way and the blast radius is `beam`, with the body naming what dobby edits outward and that a revert does not undo it.
  The Failure handling bullet now covers the spawn failure, the recorded-but-not-updated case, the retried post and the step error that holds nothing.
  The file is under `_lym`, so it is edited and not committed.
- **F7** (`loop_move_test.go`): the anchor-move test sets the describe seams and asserts that no describer starts while the record stands and that one starts once it is gone.
- **F8** (`describe.go`, `state.go`): the owed line is stored in `DescribeState.OwedLine` once `Record` has run, posted before the agent is cleared, and a failed post is returned so the next tick posts only that line.
  A subtest fails the post once and asserts one `Record` call and one posted line.
- **F10** (`internal/landingshed/beamdescribe.go`): the description and its restore are written through the package's `writeBytesAtomic`.
- **F11** (`describe_test.go`): the settling step starts from `busy = true` and asserts it stays true.
- **F12** (`dobbydeps_integration_test.go`): asserts the 30-minute timeout, the model of the `sonnet[medium]` spec, a gated describer shuttle and an ungated tidy shuttle.
- **F13** (`describe_test.go`): a subtest of its own, beside the spawn-failure table because its expectations differ, makes the state write after `Start` fail and asserts the strand is stopped and an error returned.
- **F14** (`docs/overview.md`): the module line names both agents, and the beam section names the describer agent too.

A gate then flagged three comments, in `describe_test.go`, `loop.go` and `state.go`, for a wrapped break or a line of 120 characters or more.
Each is now broken at sentence boundaries, one sentence per line, with no change to behavior.

A later gate run listed which tests reach each changed function.
Every changed function it names is reached by a test, and it names no function without one, so no test was added and the code is unchanged.

## Disputed

- Self-check, fix-code-correctness 1 (LOW), a failed state write after `Record` records the draft again.
  Ground: every fix worse.
  Evidence: `internal/dobbyengine/describe.go:300`.
  Any guard against a second record is a state write, and that is the write that failed.
  The success path's `endDescribe` failure re-recorded before this round too.
- Self-check, fix-code-correctness 2 (LOW), `writeBytesAtomic` creates the description at mode 0600.
  Ground: every fix worse.
  Evidence: `internal/landingshed/fixaccept.go:51`.
  The helper is shared, and the change description is already written through it there, so a mode change of the helper alters every record it writes.
  A chmod after the rename would split the atomic write.
- Self-check, fix-proposal 3 (LOW), a post that succeeds before a failed `endDescribe` posts the owed line twice.
  Ground: every fix worse.
  Evidence: `internal/dobbyengine/tidy.go:229`.
  The post goes out before the clear, as the review's F8 asks and as `settleTidy` does, so a failure between the two repeats one line and never loses it.

## Deferred

Nothing is deferred.

## Self-check

- fix-code-correctness, LOW, state write failure re-records: disputed (above).
- fix-code-correctness, LOW, description file mode 0600: disputed (above).
- fix-consistency, LOW, the change description's Failure handling bullet listed two cases: fixed.
- fix-consistency, LOW, the package doc's worktree-gone bullet omitted the stopped agents: fixed in `doc.go`.
- fix-proposal, BLOCKING, the proposed blast radius is longer than one word: fixed differently, as F6 above.
- fix-proposal, MEDIUM, a failed spawn's post lost the only record of the failure: fixed, the post goes out first through `postLine` and a failure keeps the landings.
- fix-proposal, LOW, a duplicate owed line after a failed `endDescribe`: disputed (above).
- fix-proposal, LOW, F9 is a prompt rule only: fixed as proposed; the review names the stencil rule as its alternative, so no change.
- fix-affected-set, LOW, `docs/overview.md` line 848 still named only the tidy agent: fixed.

Every fork returned a report; the four are saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

## Rulings

None reached me.
