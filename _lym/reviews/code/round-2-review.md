---
verdict: APPROVED
prior:
  - id: F1
    status: resolved
    answer: FixPlanAwait and FixVerifyAwait now build gatefixer.Deps with deps.ShuttleCfg and deps.Geom.AnchorRoot, which webstercli wires from c.shuttleCfg and c.geom, and the new TestFixVerifyAwait row asserts DeadReasonInterrupted.
  - id: F2
    status: resolved
    answer: waitOnly passes afterFixer as replace, so the fixer entry's Final replaces an interrupted fixer; another entry failing over one leaves the record live and known. Rows FinalOverALeftLiveRecord and AnotherEntryFailingOverAnInterruptedFixerLeavesItLive drive both paths.
  - id: F3
    status: resolved
    answer: A run error with no harvested verdict now returns an error wrapping runErr, attached or spawned, and the RunErrored and AttachedRunErrored rows pin errors.Is on it with no outcome.
  - id: F4
    status: resolved
    answer: Accepted the fixed-differently disposition. The Blast radius line must be one word, and the paragraph under it names shuttle's boxCapture readings, the orphan sweep, the bouncer judge, the Claude box reader and tender revive, reed's OOM notices and webster's fixer waits.
  - id: F5
    status: resolved
    answer: The interrupted note no longer claims that no replacement ran.
  - id: F6
    status: resolved
    answer: A failed replacement spawn is logged through logger.Warn and carried on fixerOutcome.replaceErr into the note, pinned by AFailedReplacementSpawnLeavesTheInterruptionStandingWithItsError.
  - id: F7
    status: resolved
    answer: Accepted the fixed-differently disposition. The sweep keeps a stopped record that resumableAfterInterruption accepts, which is the record goneStrandReason reads, and TestSweepOrphans_AgeGuardAndLiveGuid pins the interrupted dir kept and a died one removed.
  - id: F8
    status: resolved
    answer: The AnInterruptedFixerIsReplacedOnItsOwnFiles rows record Expired() during the wait on the replacement and assert false.
  - id: F9
    status: resolved
    answer: boxCapture warns once per strand GUID through warnedBoxDisagreements and logs at Debug after that.
  - id: F10
    status: resolved
    answer: FixVerifyAwait's doc comment states the order asking, gone strand (died or interrupted), then timeout.
  - id: F11
    status: resolved
    answer: Rows GoneStrandWithNoRunRecordIsDied and "a failed plain capture keeps the styled reading" cover both branches.
  - id: F12
    status: resolved
    answer: TestInputBoxText has the two-line wrapped prompt row and the ADiedJudge test has NeverStartedWithNoStartError.
---

# Code review, round 2

All twelve round-1 findings are resolved, and this incremental round found no new BLOCKING defect on a changed hunk.
`go vet` is clean on the changed packages.
`lym gate test ./internal/gatefixer ./internal/websterengine ./internal/shedadapters` passed at tier 0: 2315 passed, 0 failed, 43 skipped.

## Focus verdicts

- **Fixer wait's run-record location (PATTERN-cwd-resolution).**
  Every production constructor of `gatefixer.Deps` passes both `ShuttleCfg` and an absolute anchor:
  - loomcli's gate fixer seam passes `in.anchorRoot`, which is `reedGeom.AnchorPath`;
  - the weaver wiring passes `reedGeom.AnchorPath`;
  - gatecli's card fixer passes `rig.anchorRoot`, which is `reedGeom.AnchorPath`;
  - webster's `FixVerifySpawnOrAttach`, `FixVerifyAwait` and `FixPlanAwait` pass `deps.Geom.AnchorRoot`, which hubgeom and standalonegeom set.

  `goneStrandReason` no longer depends on the process cwd. Not a defect.
- **Restarted driver onto a left-live record.**
  Three paths meet that record:
  - the fixer entry's Gate replaces through `waitThenRun(..., true, true)`;
  - its Final closure, reached when `probeExpiry` sees the unknown record, now replaces too, since `waitOnly` passes `afterFixer` as `replace`;
  - an earlier entry failing over the record leaves it live and known, so Expired stops reporting it and the fixer entry replaces it later. An earlier entry passing over it leaves it unknown, so Expired fires and Final replaces it.

  Replacements stop once the set holds more than attempts plus one interrupted records.
  `LeftLiveByARestartedDriver`, `FinalOverALeftLiveRecord` and `AnotherEntryFailingOverAnInterruptedFixerLeavesItLive` drive these paths. Not a defect.
- **Judge runs that end with no verdict.**
  Both a start error that RunGated returns unchanged and a run error on an attached judge now fail the call with an error wrapping it.
  A died outcome fails with `ErrNotStarted` or `ErrAgentDied`.
  The engine records neither as a history entry, so neither spends bounce budget.
  A timed-out judge still degrades to a budget-spending Stuck.
  The board entry asks only about a judge that dies before its verdict, so the timeout case is in line with it.
- **Orphan sweep and replacement spawn errors.**
  The sweep now keeps a stopped, interrupted record that carries a session and is not set aside, so the fixer wait can still read it.
  A later start of the role marks the record `NoResume`, after which the sweep removes it.
  A failed replacement spawn is logged and named in the writer's note.

## Rejected

- A failure by an earlier entry over an interrupted fixer leaves the record live while the writer reworks.
  The fixer entry later spawns the replacement on the old report without first judging the reworked tree.
  This wastes one fixer at most, with no wrong verdict, and it is not BLOCKING, so this incremental round does not raise it.
- An interrupted stopped run whose role never starts again keeps its run directory past the sweep indefinitely.
  This retains disk only and is not BLOCKING, so it is not raised.
