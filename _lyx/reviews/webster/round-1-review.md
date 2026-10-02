---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/websterengine/integrationfix.go:153
    summary: a recordFixStrand failure returns with the started fix strand still live and its GUID unrecorded, so nothing ever stops it; this is a handled error path, not card 9's accepted process-kill residual
  - id: F2
    severity: MEDIUM
    location: internal/websterengine/rebaseline_test.go:294
    summary: card 3's second #330 Seen case, a record-batch handle binding as the rewrite followed by a Rebaseline naming a pending card, has no test
  - id: F3
    severity: LOW
    location: internal/websterengine/integrationfix.go:107
    summary: infrastructure errors after recordFixStart but before any spawn (plan fingerprint, report path, reports dir, stale report removal) spend the one attempt, and the next run settles it as an interrupted attempt whose commits were never checked
  - id: F4
    severity: LOW
    location: internal/websterengine/gitwrap.go:230
    summary: commitsBetween walks every reachable commit, so after a clean parent merge the recorded fix commits and the way-forward list include the parent branch's own commits
  - id: F5
    severity: LOW
    location: contracts/stencils/webster/webster-template-integration-fix.md:2
    summary: the stencil banner says the rendered prompt is written to a prompt file under _lyx/webster/prompts/, but attemptIntegrationFix hands it straight to the strand's Spec
  - id: F6
    severity: LOW
    location: internal/webstercli/validate.go:8
    summary: new and changed Go comments in several files keep fixed-column wraps, against CONSTRAINTS.md's Comment Line-Break Convention
  - id: F7
    severity: LOW
    location: internal/webstercli/cli_test.go:466
    summary: card 2 asked to update the existing mid-run validate scope test for the begun-based switch; its doc still says Run scopes by completed cards and it has no begun, non-terminal row
---

Review range: no `_lyx/loom/rework/round-<N>/` exists, and `_lyx/shed/webster-recovery/status.json` names `product.parent` = `main`, so the subject is `git diff $(git merge-base main HEAD)..HEAD` (merge base `812600b3a`, commits `5e3330908`..`2d9e055e4`).
The plan's `## verify:` command was run in full and exits 0.

### [MEDIUM] A recordFixStrand failure leaks a live, unreclaimable fix strand

**Location:** `internal/websterengine/integrationfix.go:153`

**Issue:** After `StartFix` returns a handle, `recordFixStrand` takes the lease, reloads state and saves the GUID.
Any error there (lease acquire, `LoadState`, the vanished-record guard, `SaveState`) returns `nil, err` straight away.
The strand is a reed tmux session that outlives the process, so it keeps running and may commit to the task branch.
state.json records the attempt with an empty `StrandGUID`, so run entry's reclaim cannot see it.
The next run settles the attempt as interrupted while the strand may still be committing.
The error does not name the GUID either, so an operator cannot remove the strand by hand.
Card 9's accepted residual covers only "a process killed inside the strand's startup window".
Here the process is alive and holds the handle, so the strand can and should be stopped.
The other infrastructure returns are safe: `fixCommitsSince`, `runVerifyCapture` and `triageIntegrationFailure` all run after the wait has removed the strand, and the attempt they leave in flight is settled as failed at the next stage run.

**Fix:** On a `recordFixStrand` error, call `removeStrandIfLive(deps.Reed, handle.StrandGUID())` before returning.
Name the GUID in the returned error, and add the removal error to it if the removal also fails.

### [MEDIUM] Card 3's record-batch binding variant of the #330 test is missing

**Location:** `internal/websterengine/rebaseline_test.go:294`

**Issue:** Card 3's test list asks for the #330 Seen case "and a second case [that] does the same with a record-batch handle binding as the rewrite".
Only the begin-batch canonicalization case exists (`TestRebaseline_AfterBeginBatchRewroteBegunCard_Regression330`).
Record-batch's binding restamp (`recordbatch.go:601`) is a separate sanctioned-rewrite site, and no test shows that it moves a begun card's hash.

**Fix:** Add a test where a record-batch handle binding rewrites a begun card.
It asserts that the card's `CardHashes` entry moved to the rewritten bytes, and that a later `Rebaseline` naming only an edited pending card succeeds.

### [LOW] Pre-spawn infrastructure errors spend the attempt

**Location:** `internal/websterengine/integrationfix.go:107`

**Issue:** `recordFixStart` saves `IntegrationFix` first.
After that, `fingerprint`, `filepath.Abs`, `os.MkdirAll` and the stale-report `os.Remove` can each return an error before any strand starts.
The record stays in flight with no strand behind it.
The next run's `settleInterruptedFix` then ends it as failed, with the detail "the run ended while the fix attempt was in flight, so its commits were never checked", which misdescribes what happened.
Step 0 keeps a wiring fault from spending the attempt; a transient file-system fault before the spawn has the same character.

**Fix:** Capture the plan fingerprint and prepare the report path before `recordFixStart`, so only the spawn and what follows it run against a recorded attempt.

### [LOW] Fix commit lists include a merged parent branch's commits

**Location:** `internal/websterengine/gitwrap.go:230`

**Issue:** `commitsBetween` runs a plain `git rev-list base..head`.
`checkFixCommits` accepts a clean parent merge between the reported head and HEAD.
`fixCommitsSince` then lists every commit the merge brought in from the parent branch.
`IntegrationFixRecord.Commits` and the stuck reason's "fix commits" therefore name commits the strand never made, although the field is documented as "the strand's commits".

**Fix:** Walk the first-parent chain (`rev-list --first-parent`).
A merge commit in `preFixHead..reportHead` stays listed, so `checkFixCommits`' parent-count refusal is unaffected.

### [LOW] The integration-fix stencil banner claims a prompt file that is never written

**Location:** `contracts/stencils/webster/webster-template-integration-fix.md:2`

**Issue:** The banner says the template is "written to a prompt file under _lyx/webster/prompts/".
`attemptIntegrationFix` passes the rendered prompt as `shuttleengine.Spec.Prompt` and writes no prompt file.

**Fix:** State that the rendered prompt is handed to the fix strand's spawn as its launch prompt.

### [LOW] Fixed-column wraps in new and changed Go comments

**Location:** `internal/webstercli/validate.go:8`

**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires semantic line breaks in new and changed Go comments.
These diff lines still break mid-clause at a column:
- `validate.go`: the changed file-header sentence (lines 8–9) and `scopedValidate`'s doc comment;
- `runlevel.go`: `reclaimEntryTimeStrands`' changed opening sentence and `Run`'s changed DispatchScope inline comment;
- `beginbatch.go`: `BeginBatch`'s changed "DispatchScope, not completedCards" inline sentence;
- `planglyph.go`: the new `forthcoming` paragraph on `ValidateDispatch`, `cardsByID`'s doc and the new `forthcoming` paragraph on `resolvePass`;
- `render.go`: the changed file-header asset and renderer list, and `IntegrationFixTemplate`'s doc;
- `wiring.go`: `setRunner`'s changed doc line;
- `planglyph_test.go`: the new helper and test doc comments.

The `doc.go` hunks the focus named are one sentence per line, with breaks only at clause boundaries.
They are not a defect.

**Fix:** Reflow each listed comment to one sentence per line, breaking further only at an independent-clause boundary.

### [LOW] Card 2's mid-run validate test was not updated

**Location:** `internal/webstercli/cli_test.go:466`

**Issue:** Card 2 asks to update the existing mid-run validate scope test for the begun-based switch.
The card calls it `TestValidateCmd_MidRunScopesToPendingCards`; in the tree it is `TestValidateCmd_ScopeFollowsRunProgress`.
It was not touched.
Its doc still says "Run scopes that gate by the run's own completed cards", which is now false.
It has no row for a begun, non-terminal batch.
`TestValidateCmd_Regression329_ForthcomingCreateTargetPassesPending` covers the switch, but in a different test.

**Fix:** Change the doc to begun-batch scoping, and add a row where batch 1 is begun but not terminal and validate answers `pending`.

## Focus verdicts without a finding

- `runIntegrationStage`, `recordTriageResult` and `settleInterruptedFix` match card 9's step list:
  - the in-flight settle runs before the `OK` early return;
  - there is no `StartFix` for a non-done Master, a flaky verdict or a pre-existing verdict;
  - no `-1` record is written on `fixed`;
  - `Result` is set in the same leased save, and a spent attempt keeps its recorded `Result`;
  - no lease is held across the wait or a verify run.
- `finishFixAttempt` decides success by the post-fix verify and triage, never by the strand's report.
  A new regression identity yields a `regression` verdict and therefore `failed`.
  `Cleared` and `Remaining` are correct in every branch.
  The verify runs at the HEAD `checkFixCommits` reconciled.
- `checkFixCommits` matches card 7's bullets in order, and every card-7 and card-9 refusal test exists.
  In the hub geometry, `planDirRel` resolves the linked plan directory outside the worktree to `""`, and the `_lyx` rule still covers the entry itself.
- Every sanctioned-rewrite site reaches `restampFingerprint`, and `Rebaseline` reaches only `restampBaseline`.
  The refused foreign-edit tests exist and leave `CardHashes` unchanged.
- `resolvePass` selects forthcoming cards by ID from the reloaded plan after a rewrite and from `plan` otherwise, never from `current`.
  Only their Create targets and Rename New sides join the exclusion sets.
- `validateCmd` passes its lease-loaded state, a nil batcher with state returns `ErrNilBatcher`, and the scope switches at the first begun batch.
- No reference to webstercli's deleted `completedCards` remains.
  `commitsBetween` and `commitChangedPaths` duplicate nothing in `gitrepo`: `ChangedFilesSince` is an aggregate diff against HEAD, not a per-commit list.
- Both stencils carry every rule cards 4 and 5 require, and their property tests assert them.
  The reworded header in `2d9e055e4` stays accurate.
