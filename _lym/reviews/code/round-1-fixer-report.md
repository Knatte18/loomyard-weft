---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "The proposal keeps the hold until a turn end is read after the strand was seen awaiting, but a probe that only sees awaiting then quiet cannot tell a reply read from a question that settled unanswered, so a settled question would hold the run to its deadline. The own change keeps the two quiet probes and also requires the agent's pane to read idle (SessionCycler IdleSession with StartupReady, as startHoldReleased reads it) before the release. A turn the reply wakes without writing a turn start leaves the pane busy, so the hold stays, and a settled question still releases. The premise could not be recorded against a real asyncRewake wake here, so no claudeengine corpus row was added."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed-differently
    reason: "An export_test.go accessor in shuttleengine is invisible to the other packages' tests, so the proposal cannot reach the loom, burler, webster and batten wiring tests. Runner.HasReplyProbe is an exported accessor beside the existing Runner.PauseGeometry, and the wiring tests assert it on the wired runner. burlerCLI gained a runner field, set in wireHub, since its engine hides the runner. batten's test calls primeShuttle on a hub with a .git directory."
  - id: F9
    disposition: fixed
---

# Fixer report, round 1

Every finding is fixed. Commits, in order: F1 `creelcli`, F2 with F5 and F7 `shuttleengine`, F3 with F6 `creelengine`, F4 `gatecli`, F8 `wiring`, then two commits for the self-check's findings.
F9 edits `_lym/landing/summary.md`, which no commit covers.

## What changed

- **F1.** `creelcli.AwaitsReplyProbe` signs as the caller's own strand when `LYM_STRAND_NAME` names a non-orch strand, and falls back to the program only on `ErrStrandClaimOutsidePane`. A creelcli row runs it from a worker strand's pane and asserts true for that strand's open question.
- **F2.** `releaseReplyHold` also needs `agentBusy` false: a pane that reads mid-turn keeps the hold, and the next probe interval reads it again. `TestRun_Wait_ReplyHold` gained rows for a reply read with no turn start (ungated and gated, with the gate not run while busy) and for a turn start after the reply, which ends the hold so only that turn's end finishes the run. That last row pins the hold clear at `wait.go:332-334`.
- **F3.** `releaseGoneHolder` settles each open question its removed strand sent as unanswered with the new reason `asker-gone`. `sweepSubjects` lists such askers so the sweep reaches them. The open-question test is one shared `isOpenQuestion`, used by `AwaitsReply` too. A `TestStore_AwaitsReply` row removes an asker and waits for the settle.
- **F4.** The gate card fixer's runner gets `SetReplyProbe(creelcli.AwaitsReplyProbe(location, cardFixerProgram))`. Weaver's runner at `weavercli/wiring.go:44` is a reset runner that calls `ProbeGated` only and waits on no agent run, so it stays unwired. The summary's Bounds list matches.
- **F5.** A sibling test has a gate that fails once with `Attempts` 2. After the release it asserts one gate evaluation, one re-prompt and no probe asked before the next turn end.
- **F6.** Two `TestStore_AwaitsReply` rows: a question returned unread to its asker, and unread mail answering another strand's question.
- **F7.** A sibling test starts the hold, begins a pause, lets the hold release under it, and lifts it later. It asserts no gate runs while covered and one runs on the first uncovered tick.
- **F8.** See the disposition above. Assertions sit in the loom, burler, webster and batten wiring tests.
- **F9.** The summary's blast radius names shuttle, the creel daemon protocol and the parent-directive stencils. The summary's other sections were updated for F2, F3, F4 and the new tests.

## Gate repair

The gate flagged one wrapped comment and four over-long comment lines in `battencli/wire_seams_test.go`, `creelcli/goclient.go`,
`creelengine/sweep.go`, `creelengine/verbs.go` and `shuttleengine/replyhold.go`.
Each is now split at a sentence or clause boundary; no code changed.

## Reach gate repair

The reach gate found three new symbols no mapped test reaches.
`TestClaimsNonOrchStrand` (creelcli) now covers `claimsNonOrchStrand`, and `TestNewStrandRig_RunnerHoldsAReplyProbe` (gatecli) covers `newStrandRig`.
`GoClient.AwaitsReply` stays unmapped: reaching it needs a live daemon, which only the higher-tier creelcli row drives.

## Disputed

Nothing disputed.

## Deferred

Nothing deferred.

## Rulings

No parent ruling reached me.

## Self-check

Four forks returned reports, saved as `round-1-fixer-report.lens-<lens>.md`.

- fix-code-correctness, MEDIUM: the `asker-gone` settle would end a chase of a question another hub or machine sent. Fixed: `isOpenQuestionOfThisHub` excludes `SendingHub` and `SendingMachine` messages. No row was added, since a cross-hub sweep row needs the daemon's inbound machinery; the guard is covered by reading only.
- fix-code-correctness, LOW: the F5 row probably passes with `run.replyHold = nil` removed. Fixed: the row now asserts that no probe runs between the release and the next turn end, which a stale hold would break.
- fix-code-correctness, LOW: a pane that never reads idle holds a settled question's run to the deadline. Fixed: the summary's Bounds list says so.
- fix-consistency, LOW: `creelengine/doc.go` omits `asker-gone`. Fixed.
- fix-consistency, LOW: the sweep comments describe holders only. Fixed.
- fix-consistency, LOW: the summary's Tests section was stale. Fixed.
- fix-consistency, LOW: `shuttleengine/doc.go` omits the idle-pane condition. Fixed.
- fix-consistency, LOW: `noticeUnansweredLocked` has no text for `asker-gone`. Fixed differently: no branch added. The sender of such a message is unlisted, so the function returns before it builds any text (`chase.go`, `noticeUnansweredLocked`), and a branch for it would be dead code.
- fix-proposal, MEDIUM: the same cross-hub defect as the first fork's MEDIUM. Merged with it and fixed.
- fix-proposal, LOW: `Runner.HasReplyProbe` and `burlerCLI.runner` exist only for tests. Fixed differently: kept; `Runner.PauseGeometry` is the precedent, and the finding itself judges the export_test route unable to cross packages.
- fix-affected-set, LOW: the shuttle CLI's runner and `orchcli start`'s runner get no probe, so the Bounds claim was too strong. Fixed: Bounds names them as runners of runs whose prompts the caller supplies.
- fix-affected-set, LOW: webster's standalone wiring has no probe. Fixed: Bounds says it has no orch to ask.
- fix-affected-set, LOW: the card fixer's probe has no wiring assertion. Fixed differently: no row added; `newStrandRig` builds runners also used for stop-only calls, and the finding names no behaviour that would break.
