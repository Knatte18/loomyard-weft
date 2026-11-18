```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I traced every function, branch and error path the diff adds or changes, using concrete inputs.

**Per-function list (case run → result)**
- `ending`/`done`/`fail`/`stuck`/`halt`/`gateEnding` (stuck.go): cancelled ctx gives `err`=cancel error with `Halt` false, the same as the old `stuckVerdict`. A live ctx with reason "r" gives Stuck/{Reason:r}, and `halt` adds Halt=true. `gateEnding` with err≠nil goes to `fail`, with reason "" and nil err it returns nil. Correct.
- `reportVerdict`: an entry neither ended nor stopped falls through the switch and the loop continues, the same as the old `default: continue`. Ended goes to `halt` and stopped to `stuck`. Correct.
- `reconcileVerdict`: a nil reconcile gives nil. An unreachable origin gives nil plus a Warn. Correct.
- `Finalize.Call`: when `landingSummary` errors, the result is a wrapped error. Its text changed from "parse" to "read", which the summary documents. A nil ending from `landUnderLock` loops with front=true. Correct.
- `landUnderLock`:
  - guard err → `fail`; lockReason → `halt`.
  - A `ParentHoldsTaskWork` error → owner halt. The old `ownerChecks` read gave the same, and the summary documents the loss of the land-check and syncParent variants.
  - parentOpener err → stuck with "error" fields, as before.
  - Retry loop, first merge-in-required: `continue` with retried=true, skips `mergeOnGitHub`, re-runs `commitParentRecords`, `recheckLock` and merge. Second merge-in-required: falls to the guard check and then "parent-side merge failed". The old flow was identical.
  - lockFreed after `mergeAndGate` in either pass → nil (requeue).
  - Success → `pushParent`, which never returns nil, so the `end.err` read is safe.
  - Correct.
- `parentHoldsWork`: a nil seam gives false, nil. Correct.
- `syncParent(held=true)` returns "", nil before any MidMerge read. Correct.
- `pushRecords`/`pushParent`/`recheckLock`/`mergeAndGate`/`gateAndFix`/`runFixer`/`judgeFixerChange`/`haltReviewCapped`/`endSpent`: each old `errorStop` maps to `fail` (cancel check kept), each `fz.halt` to `halt`, and `gateStop(ctx,"",err)` to `fail`. `letThrough` returns nil, which lets the landing proceed. Correct.
- `mergeParent`:
  - `readMergeBefore` err → raw error, as before.
  - Resolver err → wrapped `fail`. The old `mergeIntoTask` wrapped the error and then checked for cancel.
  - OutcomeStuck → non-halting `stuck`, as before.
  - Correct.
- `judgeParentMerge(before, upToDate)`: parentFrom falls back to taskBase when earlier.MergedTip is "". A held read error gives a wrapped error. Correct.
- `commitParentRecords`: nil seam → nil; commit error → stuck with "error" fields. Correct.
- `landCheck(held)`: a nil LandBlockers or held=true gives nil. A blockers read error gives a halt with `LandHaltPrefix` and the same text. Correct.
- `ownerChecks(held=true)` returns `reconcileVerdict(…, nil)`, which may be nil, and the landing proceeds as before.
- `handOver`: when `HandOver` returns a non-ErrHandedOver error, including nil, the result is the wrapped error, as in the old `runHandOver`. ErrHandedOver gives a halt naming it. This removes the struct-field hand-off with no ordering change. Correct.
- `handedOverStop`: nil seam or PushSkipped → nil; read err → transient error; elsewhere → `handOver`. Correct.
- `pullRequestLanding`/`landingForDecision`/`recordedBeamLanding`/`dropRecordedPullRequest`/`openAtOnce`: each `failed` closure keeps its wrap text and cancel check, and GitHub actions keep their names through `pullRequestStatus`. `openAtOnce` with found and BeamRecord==nil runs Edit. Without found it runs Create and no Edit, the same as the old `else if`. Correct.
- `pullRequestStatus`/`closeIfOpen`: an API err returns "open the GitHub client", a Status err returns "read the pull request's status", and a state that is not open returns "", nil. Correct.
- `findOrOpenPullRequest`: found → (ref, true). Not found → Create → (ref, false). This matches PR-Review's old `editTitleAndBody` logic (initial true, false after Create). Correct.
- `pushTask` with a non-transient error gives a halt with prefix plus `pushHaltReason`. Finalize uses `LandingStopPrefix` and PR-Review uses "". A transient error returns `terr`, and `transientFailure` already checks cancel (ctx.go:41-48), so dropping Finalize's extra `errorStop` changes nothing. Correct.
- `commitPRRecord`/`removePRRecord` keep the same wrap texts as the old per-row `commitRecord`. Correct.
- `landingSummary`: the order parse → engines → drift is the same for both rows, and the body gets the drift section appended. PR-Review's `bodyWithCredit(summary.Body, credit)` matches the old `withSection` followed by credit. Correct.
- `githubFailure` (free function plus two wrappers): a token failure skips the transient check and halts, with `LandingStopPrefix` for Finalize and "" for PR-Review. Correct.
- `PRReview.Call`:
  - `settleSendBack` non-nil → skip decide.
  - nil → decide. Every path of `decide` returns non-nil: `closeRecordedPullRequest` always ends in `done`, an error or `githubFailure`; `openPullRequest` ends in `wait`, which loops until non-nil. So reading `end.err` cannot dereference nil.
  - `errors.As` on a nil err is false.
  - Correct.
- `settleSendBack`/`sendBack`: a dirty tree gives a halt. A clean tree gives a non-halting `stuck` with Reroute and Path set only when err is nil, the same as the old code. Correct.
- `wait`/`wake`/`pollStatus`/`pollFailure`/`syncText`/`gateHead`/`judge`/`endReview`:
  - A transient poll failure returns nil, so the wake goes on unpolled.
  - Cancel gives an err.
  - `judge` returning nil with stopped=true resumes the session.
  - `judge`'s old local `stuck` closure halted (halt=true), and the new code calls `halt`.
  - Correct.
- `checkWiring` (reflection):
  - Every `PullRequestDeps` field (deps.go:401-461) is a string, func or interface, so `IsZero` matches the old `==""`/`==nil` checks.
  - The four optional names match the struct.
  - A typed-nil `Sessions` reads as non-zero, as the old `== nil` did.
  - With several fields unset, it names the first in declaration order rather than the old switch order, which only matters on caller misuse.
  - Correct.
- `parseVerdictReport`/`bullets`/`ParseFixerReport`/`ParseFixReviewReport`:
  - An empty file gives lines[0]="" and an error with the same text.
  - Escalate with no "## Conflict" heading reads `sections["Conflict"]` as nil, joins to "" and errors, as before.
  - A Rejected report with no Findings bullets errors.
  - Lines before the first heading go to key "" and are ignored, as before.
  - Read-error wrap texts are unchanged.
  - Correct.
- `startFailure`: a runErr other than NotStarted returns nil and the later runErr handling stays. runErr nil with NotStarted returns `StartError`. Correct.
- `gateCall`/`runMarked`/`check`/`checkHead`: a nil command gives ok=false and err=nil, so the check passes. A command read error gives ok=false and the error. An empty command gives the Warn with the same fields. A base command error is wrapped as before. The mark sequence "", "verify X", "" is kept. Correct.
- `soloSession.table`/`resolveModelChoice`/`requireValues`: the Fixer and FixReview sequence (required, mode, stencil, model) keeps the old order and wrap texts. In `DescribeTable`, the stencil read now runs after `runRecordBullets` instead of before it (see finding 1).
- `newResolver`/`newLandingGate`/`assembleFinalize` closure: error texts and wiring are identical. Correct.
- `readMergeBefore` for ParentAlign: `NewParentAlign` requires `MergeBase` (parentalign.go:100), so the nil branch cannot occur. An unreadable earlier record leaves `earlier` zero, so `AlignedTip` is "", as the old `alignedTip` was. Only the log text changed, as the summary documents. Correct.
- `readRowAttempts` and `ParentAlign.readAttempts`: a nil RowEntry reads entry 0 (the old code would have panicked). An undecodable record returns the reason with err nil, which leads to a halt. A RowEntry error is now read after the scratch move, as documented. Correct.
- `PRReview.sessionSpec` keeps the "pr_reviewer model-spec" wrap for both parse and resolve errors. Correct.

**Findings**

1. NIT · design · `internal/landingshed/describe.go:115-135` (`DescribeTable`). The stencil-presence check now runs inside `soloSession.table`, after `runRecordBullets`. With both a missing stencil and a failing run-record read, the call now returns the run-record error where it used to return the stencil error. This is an error-precedence change that the summary's "Behaviour notes" do not list. Evidence: diff 001 lines 94-101 (the stencil read removed ahead of `runRecordBullets`) and diff 003 lines 659-662 (stencil read inside `table`).

No other wrong result, swallowed error or inconsistent failure state found.

Tool calls: none; the inherited context held every definition the traces needed.
