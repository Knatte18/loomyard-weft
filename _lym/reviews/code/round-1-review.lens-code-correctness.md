```yaml
lens: code-correctness
outcome: reported
```

**Cases run**

- `driveNotices` (`drivehandback.go:56-71`): watched + blocked + `Judged` true, no relay/question → run-id line. Correct. Watched + blocked + Judged + `Relay` set → relay alone (len(notices) non-zero). Correct. Watched + awaiting + Judged → nothing. Correct. Watched + blocked + not judged → nothing. Correct. Unwatched done → nothing. Correct.
- `goDriver.run` reset of `judged` (`drive.go:249-251`): fresh step with `attempt == ""` → reset to false. Correct. Step consuming an arbiter resume (`attempt` non-empty) that stops at a *different* producer no agent judges → `judged` stays true from the earlier stop. Incorrect (F3).
- `settleStop` tackler/arbiter branch (`drive.go:313-326`): arbiter `Judge` error → `judged` set true before `arbitrate`, which hands back with Judged true. Correct. Hand-on with empty `Question` → Judged true into `handBack`. Correct. `moved` key differs from `stop` then `continue` → `judged` stays true for the new stop. Incorrect (F3).
- `resumeOpenAttempt` → `settleStop`: rule table routes back to tackler/arbiter case, setting `judged`. Correct.
- `handBack` (`drive.go:1002`): copies `d.judged` into `h.Judged` before `handBackRun`. Correct.
- `overviewFrameEditedFindings` (`gates.go:318-333`): st nil → nil. Correct. Batches empty → nil. Frame hash empty with `PlanFileHashes` recorded → nil while rebaseline refuses. Mismatch (F2). Overview missing → frame "" ≠ recorded → finding. Correct. Overview without Card Index → unreachable here (plan already parsed; planparser requires Card Index). Correct.
- `ValidatePlan` wiring (`gates.go:284-288`): `st` is the `*runplan.State` used by `DoneCards`; error returned. Correct.
- `CommitWithTrailers` (`commit.go:164-171`): key "Bad Key" → refused before staging. Correct. Value "a\nb" → refused. Correct. Empty trailer slice → same as `Commit`. Correct.
- `commitWeftLocked` with trailers (`recordsgit.go:224-246`): born warp → message ends `Warp-SHA` then each trailer via `appendTrailerLine` on the trailer block. Correct. Unborn warp → trailers without Warp-SHA. Correct per doc. Nothing staged changed → `committed` false, `forceEmptyCommit` false → trailers silently dropped, returns ("", false, nil). Incorrect for the reset caller (F1).
- `fabricSync` with variadic trailers (`sync.go:23-38`): nil `open` path unchanged; trailers reach `CommitWithTrailers`. Correct.
- Reset transition (`reset.go:177-185`): hub, `--to batch-start` with `Recoveries>0` → state.json changes → commit carries Warp-SHA(new tip)+Run-Transition. Correct. Hub, `--to pre-fix`/`report-head`/`last-batch-head` with `PreFixHead` already "" (or batch-start with Recoveries 0 and RecoveryStartSHA "") → `SaveState` rewrites identical bytes → nothing committed → no transition, code branch already moved. Incorrect (F1). Archive-only start → archive moves files → commit lands. Correct. Standalone → runID "" and open nil. Correct.
- `wireHub` runID (`wiring.go:217`): `ResolveRunID(loc, SelfRunID)` → `WorktreeName`, the slug `lastTransitionCommit` / `ReturnToPreviousTransition` match on. Correct.

**Findings**

- **F1 — MEDIUM — design — `internal/webstercli/reset.go:177-185` (with `internal/fabricengine/recordsgit.go:251-265`)**
  Summary: A reset whose state.json bytes do not change commits nothing, so no `Run-Transition` is written and the run's last transition still names the dropped tip — the defect the change set out to remove.
  Case: `fabricSync` → `CommitWithTrailers` inherits `commitWeftLocked`'s no-op rule: `forceEmptyCommit` is set only by snapshot tags, so an unchanged pathspec returns ("", false, nil) and the trailers are dropped. `reset.go` only blanks `PreFixHead` (and for batch-start zeroes `Recoveries`/`RecoveryStartSHA`); when those already hold those values, `SaveState` (`websterengine/state.go:208-219`, plain `WriteJSON`, no timestamp) writes identical bytes. E.g. `reset --to last-batch-head` with no pre-fix head recorded, or `--to batch-start` on a batch that never recovered, after `ResetPairCode` already moved the code branch.
  Evidence: `recordsgit.go:246` (`forceEmptyCommit := len(snapshotTags) > 0 && !unborn`), `recordsgit.go:258-263` (`!committed` → `"", false, nil`), `reset.go:150-160` (only state mutation), `CommitEmptyWithTrailers` (`recordsgit.go:170-176`) exists precisely because "It always lands a commit … so the trailers have a commit to sit on".
  Fix direction: make a reset's transition commit land even when nothing changed (force an empty weft commit when trailers are given).

- **F2 — LOW — design — `internal/loomshed/gates.go:318-320`**
  Summary: The gate's trigger is narrower than the rebaseline refusal it claims to mirror, so the two still disagree.
  Case: `Rebaseline` refuses an overview change outside the Card Index whenever `PlanFileHashes` is recorded, with no begun-batch condition (`rebaseline.go:286-310`), and with `PlanOverviewFrameHash` empty `recordedOverviewFrame` falls back to the baseline copy's frame or refuses outright as "no recorded overview frame" (`rebaseline.go:70-80`, `117-128`). The gate returns nil whenever `PlanOverviewFrameHash == ""` or `len(st.Batches) == 0`. A record from before frame hashes existed, or a started run with no begun batch, still passes the plan gate and blocks at Webster.
  Evidence: `gates.go:319`; `rebaseline.go:70-75`, `293-305`.

- **F3 — LOW — design — `internal/sheddriver/drive.go:249-251` and `drive.go:322-325`**
  Summary: `judged` outlives the stop it describes, so it can mark a stop no agent judged.
  Case: after an arbiter resume, `attempt` is non-empty on the next pass, so `judged` is not reset; if that step parks at a new blocked stop that hands back by rule (no agent), the watched driver still sends the run-id line. Likewise when `tackle`/`arbitrate` returns a moved stop with a different key and `settleStop` continues with `stop = moved`. The doc says `judged` is "whether the tackler or the arbiter was asked to judge the stop in hand" (`drive.go:168`); the code does not hold that. Effect: a duplicate parent notice beside batten's for an unjudged stop.
  Evidence: `drive.go:249-251`, `drive.go:322-325` (`stop = moved; continue`), `drive.go:579-583` (moved-run return).

- **F4 — LOW — design — `internal/sheddriver/drivehandback.go:62-63`**
  Summary: The stated reason for the new watched branch does not match batten's hold logic; in practice every watched judged blocked stop now sends two notices.
  Case: the diff's comment says batten holds its notice of such a stop "so the parent would otherwise hear nothing", but `stopHold` ends the hold once a park marker at or after the episode start exists (unless a hand-over is recorded), at an arbiter hand-on, and immediately when no spawns are left and no attempt exists. `handBackRun` always parks a blocked stop, so batten's notice goes out after the driver's line for both the hand-on and refusal rows the diff adds — not only "while batten's hold ends without the arbiter's hand-on" as the summary says. The comment misstates the behaviour.
  Evidence: `battenshed/stophold.go` (`if parkFound && !parkAt.Before(since) {... return false, parkAt}`; `reading.Attempt && !reading.HandOnAt.IsZero()`; `!reading.SpawnsLeft && !reading.Attempt`), `drivehandback.go:172-181` (Park after the report).

**Tool calls**
1. `drive.go:440-720` and `reset.go:100-240` — `tackle`/`arbitrate`/`resumeOpenAttempt`/`checkBlockedStart` bodies and the reset verb's state mutations before `fabricSync`; only diff hunks were in context.
2. `gates.go:256-300`, `SaveState`, and grep hits in `rebaseline.go`/`planparser` — the `st` type in `ValidatePlan`, whether `SaveState` can write identical bytes, and the Card Index parse precondition.
3. `rebaseline.go:100-130` and `284-320` — trigger conditions of the overview refusal, which the manifest had only as doc text.
