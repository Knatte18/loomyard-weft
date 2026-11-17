```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. I traced each guard, refusal and test row the cards add from a real entry point. One finding.

**MEDIUM — Card 10 (`stalemate-escalation`), D6's replay rule: a pending `stalemate` escalation is not re-escalated once `review_stalemate_rounds` is 0.**
- **What the card does:** `Bouncer.stalemated` answers "false when `StalemateRounds` is 0" first. Only after that does it answer "true when round N's own escalation record holds cause `stalemate`".
- **How the state is reached:** a round escalates `stalemate` and the run awaits. The parent then sets `review_stalemate_rounds: 0`, which `LoadConfig` admits per Card 12, and runs `lym loom resume`.
- **The path taken:** `Call` finds the round judged, and `standsAsRecorded` holds, so it goes `settle`, then `settleUnconverged` (no decision file), then `settleUndecided`. There `stalemated` returns false before it reads the record.
- **Why existing code does not catch it:** the escalated-record branch at `internal/shedadapters/bouncer.go:1361` re-escalates only when `windowOpen || verdict == verdictCircling`. A CONTINUE round below the soft-cap window therefore falls to `bouncer.go:1370-1375`. It calls `ensureFocus` and returns Stuck, so another round runs over an escalation no decision answered.
- **What D6 requires:** "a round whose own pending escalation record holds cause `stalemate` escalates with that cause again", with no condition on the key.
- **Fix:** read the escalation record first, then apply the `StalemateRounds == 0` check.
- **Test gap:** no row in `TestBouncer_Stalemate` reaches this (key 0 with a pending stalemate record and a CONTINUE verdict below the window).

Tool calls:
- `grep` of `internal/burlerengine/profile.go` for path resolution, of `landingshed`/`burlercli`/`treadleengine` for `Engine.Fix` callers, and of `warpexclude.go`. My context lacked the profile path root, the other users of the fix table, and the warp excludes.
- `grep` of `burlerengine/geometry.go` and the loomcli `WorktreeRoot:` sites. My context lacked whether burler profile paths and `Env.WorktreeRoot` share the worktree root, as Cards 3 and 12 assume.
- `grep` of `shuttleengine/gate.go` for `Passed`, plus the helpers the cards cite (`roundSeed`, `ParentReviewFilesOf`, `ReadParentReport`, `ReadParentAsks`, `Bouncer.reviewFindings`). My context lacked whether a pass-on-cap entry sets `GateOutcome.Passed` false, and whether the cited helpers exist.
