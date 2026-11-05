---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lym/plan/10-beam-await-entry.md:45
    summary: "Beam-Await's entry detections miss an accepted landing whose re-check the loop left before the first removal, when the run was halted and resumed, so the run waits for a land request nothing writes again"
    fix: "In 10-beam-await-entry.md Intent, add a third entry detection beside the two in the bullet list: the dobby-stop marker stands with no land request standing. awaitRun.landChecks is the only writer of that marker, and the accept, recheckStanding and refuse remove it only after a pass or a restart, so a standing marker with no land request is a landing whose re-check the loop left before its removals. A resume of a halted run rewrites the board status to Beam-Await before the row runs, so the board-hold detection does not see it, and the await marker still stands, so the ready-and-no-await-marker detection does not either. Read at entry only, like the other two, it feeds the same two branches: the ready record standing runs recheckStanding, the ready record not standing takes the lifted-hold branch. Add to Reaching inputs: await marker standing, dobby-stop marker standing, no land request, board at Beam-Await, ready record standing. Add two TestBeamAwait rows for that state, one with Recheck passing that ends Done with no land request and one with no ready record that lifts the hold and starts the dobby. Name the third signal in the Shared Decisions bullet of 00-overview.md that starts 'Beam-Await's entry finds an accepted landing'."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/plan/15-beam-ready-refusals.md:36
    summary: "Refusal 5 quotes the resolver's Halt, whose way forward ends in lym loom resume, a step that does nothing for a run waiting at Beam-Await"
    fix: "In 15-beam-ready-refusals.md Intent step 5, replace 'A Halt is refused quoting it' with a refusal built from landingshed.OriginMissing(origin), the helper card 13 creates. It names the demand (the parent's beam.yaml pull_request or GitHub's rules), the missing origin fact, and the way forward 'give the hub a GitHub origin, an upstream for the parent branch and no push skip, then run lym beam ready <slug> again', since the resolver's own Halt text ends with 'then run lym loom resume'. Every Halt of ResolvePullRequest comes from a missing origin fact, so OriginMissing covers the same inputs. Add plan:internal/landingshed#OriginMissing to card 15's Uses; lym loom validate-plan --overlay with that Uses line added reports the same findings as the {} baseline. In the TestMarkReady row 'a required parent whose origin has no upstream' assert the message names lym beam ready and not lym loom resume, and add a row for a merge parent with the same missing fact."
  - id: F3
    severity: LOW
    class: design
    location: _lym/plan/15-beam-ready-refusals.md:37
    summary: Refusal 6 names no refusal or test for a failed GitHub read, which is what a hub with a recorded pull request, no usable token or a changed origin reaches
    fix: "In 15-beam-ready-refusals.md Intent step 6, add that a PullRequestSide error or a ReadMergeState error refuses naming the cause and the way forward 'fix it, then run lym beam ready <slug> again; lym beam pr status <slug> reads the same state', with no record written. Add a TestMarkReady row: a recorded pull request whose prkit Status call fails is refused naming the cause and lym beam ready, and no ready record is written."
  - id: F4
    severity: LOW
    class: consistency
    location: _lym/plan/03-pr-merge-state.md:43
    summary: "The blocked cause lists each required check whose conclusion is not success, so a required check that concluded neutral or skipped, which GitHub counts as satisfied, is named as owed"
    fix: "In 03-pr-merge-state.md Intent, change the ReadMergeState cause rule to name a required check only when its conclusion is not success, neutral or skipped, the conclusions GitHub counts as satisfied for a required check; an empty (unfinished) or missing conclusion, failure, cancelled, timed_out, action_required and stale are owed. Add one TestReadMergeState row: a blocked pull request whose required check concluded skipped lists no such check and names the other owed one."
  - id: F5
    severity: LOW
    class: design
    location: _lym/plan/12-beam-describe-verb.md:45
    summary: "judgeAwaitingRun refuses a run that is done as a separate branch, but a done status names Beam-Down, never Beam-Await, so the branch has no input that reaches it"
    fix: "In 12-beam-describe-verb.md Intent, change judgeAwaitingRun to refuse a beam with no pawl run and one whose run is not at Beam-Await, and drop 'or is done': Shed persists the next row on a Done outcome and keeps a row's own name with state done only for an empty on_done, so the status of a finished pawl run names Beam-Down and the not-at-Beam-Await refusal covers it. In the TestDescribe refusals list, change 'a done run' to 'a run at Beam-Down'."
  - id: F6
    severity: LOW
    class: consistency
    location: _lym/plan/15-beam-ready-refusals.md:39
    summary: "Two doc comments the plan makes stale are not named by any card Intent, the ready paragraph of beamcli/doc.go and the caller list in pullrequest/doc.go"
    fix: "In 15-beam-ready-refusals.md Intent, extend the doc bullet so internal/beamcli/doc.go's ready paragraph is rewritten: it now refuses, in order, a run not at Beam-Await, open work, an unmerged parent, a missing or invalid description and the pull request conditions, is final, and no verb removes the record; the paragraph's current sentence that it refuses what land's entry check refuses and reports a beam already flagged is not the whole contract. In 03-pr-merge-state.md Intent, state that internal/pullrequest/doc.go's first sentence stops listing PR-Review, Finalize and pair teardown as the callers and names no caller list, since the lym beam pr verbs and the retired-row move call the package too. Both files are already Edit or Extends targets of those cards, so no target changes."
---

### [MEDIUM] Beam-Await's entry misses a landing whose re-check the loop left, after a halt and resume

**Location:** `_lym/plan/10-beam-await-entry.md:45`

**Issue:** Card 10 detects an accepted landing in two states, the board naming the hold and the ready record standing with the await marker absent.
In `internal/shedengine/run.go` step 3b, a resume of a paused, blocked or failed run calls `persistMarked`, which runs `CommitStatus` and rewrites the board status to Beam-Await before the row runs; `Goto` leaves the run paused for the same reason.
So after a halt the board-hold detection reads nothing.
`recheckStanding` removes the await marker only after its re-check passes, so when the loop leaves between `landChecks` and the removals, for example a process killed repeatedly until the driver's repair cap hands the run to the tackler and a halt, the await marker still stands and the second detection reads nothing either.
The state is then the ready record standing, the dobby stopped, the dobby-stop marker standing and no land request; the row takes today's ordinary path and waits.
The dobby's land episode stays recorded (Constraints), so nothing writes a land request again, and ready has no way back, so only a hand deletion of the marker files frees the run.
A running re-step does not rewrite the board, so the board-hold detection catches that variant; the gap is the halted resume.
The accept path with a land request does not have the gap, since the request stands until the last removal and the next wake accepts it.
`landChecks` is the only writer of the dobby-stop marker (`internal/pawlengine/beamawait.go`), so a marker with no land request at entry is an unambiguous signal.

### [MEDIUM] Refusal 5 quotes a way forward that does nothing at Beam-Await

**Location:** `_lym/plan/15-beam-ready-refusals.md:36`

**Issue:** For a `required` or `merge` parent with a hub whose origin is not on GitHub, has no parent upstream or has the push skip set, `ResolvePullRequest` returns a `Halt` whose text ends `then run lym loom resume` (`internal/landingshed/pullrequestmode.go`, the `halt` format).
Card 15 refuses `lym beam ready` quoting that text.
A run waiting at Beam-Await has nothing to resume, and the decision record's rationale for D8 is that every message names a step that works where it is read.
The refusal should name the fact and `lym beam ready <slug>` again; card 13's `OriginMissing` returns exactly that fact for every input the Halt arises from.
The overlay check of the fix adds only a `Uses` line to card 15 and reports no finding beyond the baseline.

### [LOW] A failed GitHub read has no refusal in step 6

**Location:** `_lym/plan/15-beam-ready-refusals.md:37`

**Issue:** Step 5 states what a `PullRequestMode` or `BeamHasCommits` error refuses with, and step 6 reads GitHub through `PullRequestSide` and `ReadMergeState` with no statement for their errors, which a recorded pull request on a hub with no usable token, a changed origin or no network reaches.
`pullrequest.Open` refuses a non-GitHub origin and a missing token, and `ReadMergeState` returns a failed call.
Such a beam cannot pass, which is right, but the plan names neither the message nor a test row, so the refusal is left to whatever the propagated error says.

### [LOW] The blocked cause names satisfied required checks as owed

**Location:** `_lym/plan/03-pr-merge-state.md:43`

**Issue:** The cause for `blocked` names each required check whose conclusion is not `success`.
GitHub counts `neutral` and `skipped` check runs as satisfied for a required check, so a pull request blocked only by a review would list a skipped required check as owed, and one blocked by a failing check would list the skipped one beside it.
`lym beam pr status` and refusal 6 both print the cause text, so the operator would chase a check that needs nothing.

### [LOW] judgeAwaitingRun refuses a done run that cannot name Beam-Await

**Location:** `_lym/plan/12-beam-describe-verb.md:45`

**Issue:** In `internal/shedengine/run.go` a Done outcome with a non-empty `on_done` persists the next row, and only an empty `on_done` keeps the row's own name with state done.
Beam-Await's `on_done` is Finalize, so no status names Beam-Await with state done; a finished pawl run names Beam-Down, which the not-at-Beam-Await refusal already covers.
The `or is done` branch and the test row of a done run reach no state of their own.

### [LOW] Two stale doc comments are named by no card

**Location:** `_lym/plan/15-beam-ready-refusals.md:39`

**Issue:** `internal/beamcli/doc.go`'s `ready` paragraph says the verb refuses what `land`'s entry check refuses and reports `already`; after card 15 it refuses six more conditions and is final.
Card 15 names only the merge-parent paragraph and the verb's help for the doc rewrite.
`internal/pullrequest/doc.go` opens by naming PR-Review, Finalize and pair teardown as the callers; cards 8, 13 and 14 add callers, and card 3 edits that file without saying the sentence changes.

## Rejected

- Focus 1, beyond F2 and F3: `lym beam ready` can pass its pull-request-mode refusal with no GitHub origin under `on` and `off`, since card 11's beam resolver wants no pull request there and `parentRuleRequired` reads no rule when `OriginMissing` is set, so `pullrequest.Open` is never called; under `required` and `merge` it refuses through the resolver's halt (F2). A beam with no commits beyond its parent passes under every mode, since the resolver answers no pull request before it judges the origin, as Finalize does. With no usable token the rule read is no rule and `on`, `off` and `merge` parents pass, the `required` parent names `lym beam pr open`, which then fails on the token with its own cause; Finalize reads the same facts the same way.
- Focus 2, the literal scenario: a crash between the accept's removals is settled by card 10's removal order and the ready-and-no-await-marker detection, including after a resume from a halted status; the gap is the window before the first removal on the no-land-request path (F1).
- Focus 3: `judgeAwaitingRun` passes running, paused, failed, blocked and awaiting at Beam-Await. Failed is reachable, since `run.go` persists a producer error as `failed`, and `TestDescribe`'s running and failed rows enter reachable states; the done branch is F5.
- Focus 4: the two retired-row mechanisms are reconciled by cards 1 and 5, which name `internal/shedengine/doc.go`, `internal/loomrecipe/doc.go` and `internal/loomcli/doc.go`, and `RetiredRowNames` reads the recipe's own map; `loomrecipe.RetiredRows` stays tapestry and darn data. The old hold boundary, the Parent-Align merge-in sentence in `beamcli/doc.go` and `mergeParentCmd`, the reroute text and the BeamDescribe row are each rewritten by a card (5, 6, 7, 12, 15); the two leftovers are F6.
- Focus 5: card 3 reads `Repositories.GetCombinedStatus` as well as check runs, so a required context reported only as a commit status is read; the rules read covers repository rulesets only, as `RequiresPullRequest` does today, so classic branch protection is outside it consistently.
