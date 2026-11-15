```yaml
round: 1
fixed: [F1, F2, F3, F4]
```

## Changes

- F1, fixed differently: the proposal changed `Aside.Cancel` itself, which `cancelAside` in `internal/battencli/cancelcmd.go` also sets.
  With the proposed change, `lym batten cancel --remove` over a kept-pair cancelled run read `cancelled` and was refused by the terminal refusal that names that same command.
  I added `Aside.UnfinishedCancel` in `internal/runfsm/act.go`, which drops only a cancel intent with no finish, and `resumeAside` in `internal/tendercli/runintents.go` uses it.
  `Aside.Cancel` keeps its old meaning.
  `TestAct` Judge rows pin `Cancel` over a finished cancel (running, act), `UnfinishedCancel` over an unfinished one (running, act) and `UnfinishedCancel` over a finished one (cancelled, wait, terminal).
- F2: `cancelMarkerRefusal` in `internal/battencli/cancelmarker.go` words a zero-intent interrupted reading as an undecodable record to repair by hand.
  The `TestCancelMarker` undecodable row asserts that wording.
  The row's edit landed in the F4 commit, so the F2 commit alone leaves that row red.
- F3: `TestPauseAndResume_WriteLiftAndRefuseOverTheHub` resumes the slug `away`, owned by `other-box`, and asserts the refusal names the machine and `lym adopt away`, over the real `ReadRun` and `Judge`.
- F4: `//testtiming:keep` lines with the stated reasons sit above `TestCancelMarker_ReadsNothingFromAPrimeWithNoScratchRoot`, `TestJudgeCancelMarker` and `TestLog_ChangesOnly`.

- Gate repair: the comment in `refuseByAct` (`internal/loomcli/ownercheck.go`) and the `Aside.UnfinishedCancel` comment (`internal/runfsm/act.go`) are rewritten one sentence per line, under the length limit.

## Self-check

- fix-code-correctness, BLOCKING: `Aside.Cancel` change breaks the cancel verb over a finished cancel; fixed with my own change (the `UnfinishedCancel` field).
- fix-consistency, MEDIUM: same finding; merged into the one above.
- fix-consistency, LOW: the F2 commit holds no test change; not changed, the commits are already made and the final tree is green.
- fix-proposal, BLOCKING: same finding; merged.
- fix-affected-set, BLOCKING: same finding; merged.
- fix-affected-set: the contained-test list could not be rerun, as the redundancy report is not on disk; no finding.

## Disputed

None.

## Deferred

None.

## Rulings

None.
