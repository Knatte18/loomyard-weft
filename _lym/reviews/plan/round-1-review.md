---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: lens:guard-reach
    location: _lym/plan/06-drift-autonomy.md:11
    summary: D3's ended-ask halt and the resume's re-ask reach only DriftVerify.judge (diff lines); passCommand (diff command) runs the same judge and parent agreement gate and stores the rulings after a timeout, not-sent or no-parent ask under autonomy
    fix: "Add `internal/driftengine#DriftVerify.passCommand` under card 6's **Edit:**. In its Intent, run the `endedUnanswered` check after `runAgent` and before `storeCaseRulings`, with the same halt and reason as `judge`. On an open session mark with an ended unanswered ask, take the `reaskParent` resume in place of `runAgent`, so `storeCaseRulings` stores the stored judgement on a satisfied answer. Add `TestDriftVerify` rows under `DiffCommand`: one ended state halts, and a resumed call re-asks over the same digest. The plan gate admits this target with every other fix of this review overlaid."
  - id: F2
    severity: BLOCKING
    class: scope
    origin: lens:affected-set
    location: _lym/plan/15-beam-await-drained.md:7
    summary: card 15 makes NewBeamAwait and beamAwaitEntry require Autonomous, HoldsCommits, WorkRemains and Paths.DrainMarker but leaves shedrecipe's validPawlDeps fixture and the BeamAwait row of simpleEntryCases without them, so shedrecipe's BeamAwait entry tests fail
    fix: "Add `internal/shedrecipe#validPawlDeps` and `internal/shedrecipe#simpleEntryCases` under card 15's **Edit:**. Say in the Intent that `validPawlDeps` stubs the three seams and sets `Paths.DrainMarker`, and that the BeamAwait row's `validatedFields` names the four new fields. The plan gate admits both targets."
  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/07-loom-autonomy-forcing.md:33
    summary: "card 7 rests on a false premise: docs/overview.md:1483 says discussion_review_parent_timeout_min sets the deadline of the Discussion-Review round's ask, not of a PR-Review ask, so the planned rename would misdocument that key and leave pr_review_parent_timeout_min with no doc line"
    fix: "In card 7, replace the bullet about `docs/overview.md`'s `discussion_review_parent_timeout_min` sentence. Leave that sentence as it is. After the `pr_review_ask_parent` line (docs/overview.md:1478), add one sentence: `pr_review_parent_timeout_min` sets the minutes from a PR-Review ask to its deadline, 60 by default, and is open to `run_settings`."
  - id: F4
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/06-drift-autonomy.md:65
    summary: "reaskParent blocks the Drift-Verify row in a once-a-second poll of the ask record for up to judge_parent_timeout_min (30 by default, settable per run) with no back-off, and no PATTERN-daemon-wakeups entry lists the wait; Drift-Verify is not among the rows the step watchdog reads as active, so a run_settings judge_parent_timeout_min at or above step_idle_timeout_min gets the waiting step killed"
    fix: "In card 6, have the reaskParent wait start at one second and double to a minute while the ask record is unchanged, as the PR-Review row's wait does. Add `pattern/PATTERN-daemon-wakeups.md` under **Extends:** with a bullet for this wait. State in the Intent what a step-watchdog kill mid-wait leaves: the ask still open and the mark open, and that the re-entry continues the wait instead of spawning a judge. The plan gate admits the Extends target."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/11-pr-review-ending-autonomy.md:26
    summary: "under recorded autonomy, Continue on a clean gated head makes sessionStopWanted stop the session before judge whatever the standing, so judge's stays-open mail goes to rec.StrandName while that strand is stopped, and resumeSession restarts it after (prreview.go:660-673, 686-697); the ask-again mail can miss the resumed session"
    fix: "Add `internal/landingshed#PRReview.wake` under card 11's **Edit:**. Have the not-satisfied branch's mail go out after `resumeSession`, to the strand name the record holds once the session is back. Add a `TestPRReview_Wait` row asserting the mail names the resumed session's strand. The plan gate admits the target."
  - id: F6
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/14-goto-clears-pr-review.md:26
    summary: "clearPRReviewOnGoto clears StrandGUID and the session fields without stopping a live session. PR-Review's endSession leaves the session live when the row's context was cancelled (prsession.go:243-246), as on a pause, so a goto onto PR-Review of a paused run orphans that session and the re-entry starts a second one"
    fix: "In card 14, give `clearPRReviewOnGoto` a `stopSession func(guid string) error` seam. Have it stop the recorded session's strand through the shuttle stop verb, when the record names one that is live, before it clears the fields. GotoGuard's comment already admits ending processes once a guard admits. Add a `TestClearPRReviewOnGoto` row whose record names a live strand: the stop runs once before the clear."
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/06-drift-autonomy.md:19
    summary: "card 6 makes internal/driftengine/doc.go untrue: it says the parent agreement entry passes on a timeout or an ask that never reached the parent (doc.go:90-93), and it describes judge_ask_parent and judge_interactive only by their layered meaning (doc.go:16); the card lists doc.go under Uses only"
    fix: "Move `internal/driftengine/doc.go` from card 6's **Uses:** to **Edit:**. In the Intent, have doc.go state the forcing of the two keys under the run's autonomy, the Drift-Verify halt on an ended unanswered ask and the resume's re-ask over the same rulings. The plan gate admits the move."
  - id: F8
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/07-loom-autonomy-forcing.md:33
    summary: "card 7 forces raddle_auto_approve true and discussion_interactive false, which lets a raddle proposal file draft tasks with no operator approval and runs the Discussion lead with no operator, but its Intent states neither what the forcing now lets through nor what bounds it"
    fix: "Add a Bound paragraph to card 7's Intent. Under the run's autonomy, read at each call through `hubgeom.Autonomy`, a raddle proposal files draft tasks with no operator approval, and the Discussion lead runs with no operator in its pane. The bound: only while the run's worktree chain resolves `autonomy: true`; releasing the filed drafts stays the operator's; no layer file is written; and a raddle run past its first Raddle-Write call keeps its recorded mode."
  - id: F9
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/14-goto-clears-pr-review.md:9
    summary: "the reset runs inside the guard, before Relaunch's Admit (goto.go:244-248) and before Goto's status write, so a refused Admit or a failed write leaves the record's autonomy and session cleared and committed with the run unmoved; a run left at Finalize then reads no autonomy for its closed-pull-request halt"
    fix: "In card 14, run the reset only after the move is written. Give `shedengine.GotoRequest` and `RelaunchRequest` an `Admitted func() error` that Goto and Relaunch call after the status write, and Relaunch only when the move was not in effect. Set it beside `Guard` at the sites that build their guard through `gotoGuardEndingFixers`, and drop the reset from `prReviewResettingGuard`. Add `internal/shedengine#Goto`, `#GotoRequest`, `#RelaunchRequest`, `#Relaunch`, `internal/shedverbs#RunGoto`, `internal/loomcli#loomCLI.specFor` and `#loomCLI.loomConsumeAttempt` under **Edit:**, and `internal/shedverbs/doc.go` under **Uses:**. Give the driver's site in `goDriverDepsFor` (drivewiring.go:289) the same hook. The plan gate admits these targets with every other fix overlaid."
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/14-goto-clears-pr-review.md:9
    summary: "card 14's guard clears and commits the pull request record, which makes Goto's own comment untrue (goto.go:70: 'It sets no transient mark and commits nothing'), and the card does not list Goto"
    fix: "Add `internal/shedengine#Goto` under card 14's **Edit:**, and have its comment name the one exception: a recipe's reset of the entered row's records, which may commit. The plan gate admits the target. If F9's fix lands, the same edit describes the post-write hook instead."
  - id: F11
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/14-goto-clears-pr-review.md:28
    summary: "atOrPastPRReview's false branch and its test row ('a run halted before PR-Review resets nothing') cannot occur: CheckGotoTarget runs before the guard and refuses a target past the current row (goto.go:114-119), so a goto onto PR-Review always finds the run at or past it"
    fix: "In card 14, drop the `atOrPastPRReview` seam, its status read and its error row. Have `prReviewResettingGuard` reset whenever inner admits a target of `loomshed.NamePRReview`. Drop the 'run halted before PR-Review' row from `TestPRReviewResettingGuard`."
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/12-parent-review-ask-verb.md:62
    summary: "the row 'an answer to an earlier PR-Review ask that a newer one replaced is refused the same way' reaches the PR-Review way forward only when the newer ask has ended; with the newer ask open, ParentAnswerNotLatest takes the existing 'awaits an answer' text (parentreview.go:165-168)"
    fix: "In card 12's TestParentReviewAnswerVerb rows, state that the newer PR-Review ask has ended (timeout) for that row. Add a row with the newer ask open, asserting the existing 'a newer ask, ask N, awaits an answer' refusal."
  - id: F13
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/05-parent-ask-clear.md:21
    summary: "ClearParentAsk's refusals of a missing number and of an open ask guard states its one caller never produces: card 6's reaskParent calls it only after endedUnanswered found the ask present and ended"
    fix: "In card 5, drop the two refusals and their test rows. Have `ClearParentAsk` set `Cleared` on the numbered ask, and return an error that names the number when the record lacks it, as a caller error with no planned row."
  - id: F14
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/01-beam-autonomy-key.md:8
    summary: "card 1 makes internal/beamsetting/beamsetting.go's package doc untrue ('Those settings are pull_request and land_notice') without listing that file; its Intent also edits tasksetting_test.go's file comment, which no target covers"
    fix: "Add `internal/beamsetting/beamsetting.go` under card 1's **Edit:** and name `autonomy` in its package doc; the plan gate admits it. For tasksetting_test.go, drop the Intent's promise to edit its file comment. The edit-test-file check refuses the file as an Edit target and asks for every member of the file instead."
  - id: F15
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/10-pr-review-session-autonomy.md:104
    summary: "TestPRReview_Session's autonomy row asserts that the prompt holds the stencil's wording 'lym loom parent-review ask', a test of stencil prose; the Review criterion of D4 for the PR-Review stencil is met by reading it"
    fix: "In card 10's first new TestPRReview_Session row, drop the prompt-wording assertion. Keep the spec's `Interactive` false, the record's autonomy true and the one PR-Review ask."
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/13-pr-continue-autonomy.md:6
    summary: "card 13 changes what pr-continue means under autonomy, but prcontinue.go's file header (prcontinue.go:1-7) still says the agent runs it to tell the run the operator is satisfied, and the card does not list the file"
    fix: "In card 13's **Edit:**, replace `internal/loomcli#prContinueVerb` and `internal/loomcli#loomCLI.prContinueCmd` with `internal/loomcli/prcontinue.go`, since the card changes the header, the verb and the command. Have the Intent name the header rewrite. The redundant-file-target check refuses the file beside those member glyphs; the swap passes the plan gate."
  - id: F17
    severity: NIT
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/05-parent-ask-clear.md:10
    summary: "internal/burlerengine/doc.go says an ask moves only to an end state and never moves after (doc.go:52); card 5's Cleared mark on an ended ask goes undocumented, and the card lists doc.go under Uses only"
    fix: "Move `internal/burlerengine/doc.go` from card 5's **Uses:** to **Edit:**. Have it state that a cleared ended ask answers nothing and that its seat asks again. The plan gate admits the move."
---

### [BLOCKING] Diff-command judge keeps D3's old pass on an ended ask

**Location:** `_lym/plan/06-drift-autonomy.md:11`
**Issue:** D3 says that under autonomy "a judge ask that ends without the parent's answer stops the run at Drift-Verify", and that the resume sends a new ask over the same rulings.
Card 6 adds the halt and the `reaskParent` resume only in `DriftVerify.judge` (`verify.go:453-481`).
`passCommand` (`verify.go:1019-1077`) spawns the same judge through `runAgent`.
`tableWithParent` gives that judge the parent agreement entry, whose `agreement` passes on timeout, not-sent and no-parent (`judgeparent.go:218-243`).
`storeCaseRulings` then stores the rulings.
For every drift run with `diff: command`, D3's halt is not carried.

### [BLOCKING] shedrecipe's Beam-Await fixture lacks the new required seams

**Location:** `_lym/plan/15-beam-await-drained.md:7`
**Issue:** card 15 makes `NewBeamAwait` require three new seams and `Paths.DrainMarker`.
`internal/shedrecipe/fixture_test.go:67-96` `validPawlDeps` fills none of them.
It drives the BeamAwait rows of `simpleEntryCases` (`entries_simple_test.go:172`, the row near line 239), which then fail to build the producer.
The card lists the other seam surfaces (envkit, both waiver lists, `requirePawl`, `beamAwaitEntry`) but not this fixture.

### [MEDIUM] Card 7 renames the wrong overview sentence

**Location:** `_lym/plan/07-loom-autonomy-forcing.md:33`
**Issue:** `docs/overview.md:1482-1484` reads: "`discussion_review_ask_parent` has each Discussion-Review round ask … `discussion_review_parent_timeout_min` sets the minutes from that ask to its deadline".
So "that ask" is the Discussion-Review round's ask.
Today a PR-Review ask has no deadline: it is a creel post (`prreview.go:386-389`).
Following the card would make the overview name the new key as the Discussion-Review timeout.
The new key would then get no line of its own beside `pr_review_ask_parent`.

### [MEDIUM] The resume's re-ask wait polls without back-off inside Drift-Verify

**Location:** `_lym/plan/06-drift-autonomy.md:65`
**Issue:** `reaskParent` holds the row in a loop of one-second sleeps until the new ask ends, up to `judge_parent_timeout_min`.
That key defaults to 30 (`driftengine/template.yaml:12`) and a run's `run_settings` can raise it.
PATTERN-daemon-wakeups wants a long wait to back off while idle and to be listed in its background, and this wait does neither.
The step watchdog kills a step with no activity after `step_idle_timeout_min` (60 by default).
`loomengine/template.yaml:40` names Beam-Await, PR-Review and Sampler-Session as the rows that read as active; Drift-Verify, with no judge spawned during this wait, is not among them.
The card does not say what a killed wait leaves behind or what the re-entry does.

### [MEDIUM] The stays-open mail goes to a stopped session

**Location:** `_lym/plan/11-pr-review-ending-autonomy.md:26`
**Issue:** `wake` stops the session whenever `sessionStopWanted` holds: `Continue` set on a clean tree at the gated head (`prreview.go:660-667`, `:690-691`).
Only then does it call `judge`, and it resumes the session afterwards (`:668-672`).
Card 11's not-satisfied branch mails `rec.StrandName` from inside `judge`, so the mail lands while the strand is stopped.
`ensureSession` may then give the resumed session a new strand name (`prsession.go:89-93`).
That mail is the session's only prompt to ask the parent again, so losing it stalls the review until a hold notice reaches the parent.

### [MEDIUM] A goto onto PR-Review can orphan a live session

**Location:** `_lym/plan/14-goto-clears-pr-review.md:26`
**Issue:** `endSession` stops the session when a call ends with a verdict, but leaves it live when the context was cancelled (`prsession.go:243-246`).
A pause cancels the context, so a run paused at PR-Review keeps its session.
Goto admits a paused run, since it refuses only a running one (`goto.go:107-109`).
The card's `clearPRReviewOnGoto` empties `StrandGUID` without stopping that strand.
The re-entry's `ensureSession` then sees no strand and starts a second session, while the first stays in its pane, outside the record.
`settleSendBack` clears the same fields only after `sendBack`, whose path stops the live session first (`prreview.go:147-150`).

### [MEDIUM] Card 6 leaves driftengine's package doc untrue

**Location:** `_lym/plan/06-drift-autonomy.md:19`
**Issue:** under autonomy, card 6 turns an ended ask into a halt and adds a resume that re-asks.
The package doc still says the parent agreement entry passes on a timeout or an ask that never reached the parent, and it describes the two keys only by their layered values.
The card lists `doc.go` under `Uses:` only.

### [MEDIUM] Card 7 forces an approval bypass with no stated bound

**Location:** `_lym/plan/07-loom-autonomy-forcing.md:33`
**Issue:** forcing `raddle_auto_approve` true removes the operator approval gate on a raddle filing.
Forcing `discussion_interactive` false removes the operator from the Discussion.
The rubric's attack-surface rule asks the introducing card to state what it lets through and what bounds it.
Card 7's Intent lists the forced values and the wiring, but neither of those.
D3's Bound and Rationale supply the content: drafts' release stays the operator's, and the forcing writes no layer.

### [LOW] The reset runs before the move is certain

**Location:** `_lym/plan/14-goto-clears-pr-review.md:9`
**Issue:** `Relaunch` runs the guard and then `Admit`, which can refuse or report the move as already in effect (`goto.go:244-248`).
`Goto` writes the status only after the guard (`goto.go:117-139`).
Card 14's reset clears and commits inside the guard.
So a refused or failed move leaves a cleared record behind.
A run left past PR-Review then reads no autonomy at Finalize's closed-pull-request branch, which card 11 gates on the record's autonomy.
A retried goto repairs it, so the window is narrow.

### [LOW] Goto's comment says it commits nothing

**Location:** `_lym/plan/14-goto-clears-pr-review.md:9`
**Issue:** card 14 edits `GotoGuard`'s comment but not `Goto`'s, which still says "It sets no transient mark and commits nothing" (`goto.go:70`), although the guard's reset now commits.

### [LOW] `atOrPastPRReview` guards a state that cannot occur

**Location:** `_lym/plan/14-goto-clears-pr-review.md:28`
**Issue:** `CheckGotoTarget` refuses a target past the current row before the guard runs.
So a target of PR-Review always means the run is at or past PR-Review.
The seam, its status read and its "halted before PR-Review" row add a branch no input reaches.

### [LOW] One answer-refusal row does not reach the PR-Review way forward as written

**Location:** `_lym/plan/12-parent-review-ask-verb.md:62`
**Issue:** `RecordParentAsk` ends no earlier ask.
When a newer PR-Review ask is still open, the refusal is `ParentAnswerNotLatest` with an open latest ask.
`parentAnswerRefusalText` words that case "awaits an answer" (`parentreview.go:165-168`), not the card's PR-Review way forward.
The row must make the newer ask ended to reach the card's branch.

### [LOW] `ClearParentAsk` refuses states its caller never passes

**Location:** `_lym/plan/05-parent-ask-clear.md:21`
**Issue:** the one caller passes only an ask that `endedUnanswered` found present and ended.
The refusals of an open ask and of a missing number, and their test rows, guard states that no production path produces.

### [LOW] Two file comments card 1 falsifies or promises are untargeted

**Location:** `_lym/plan/01-beam-autonomy-key.md:8`
**Issue:** `internal/beamsetting/beamsetting.go:1-3` names the per-task settings as pull_request and land_notice.
Card 1 adds a third key there but does not list the file.
Card 1's Intent (line 43) also edits `tasksetting_test.go`'s file comment, while its targets name only two test functions of that file.

### [LOW] A planned row asserts stencil wording

**Location:** `_lym/plan/10-pr-review-session-autonomy.md:104`
**Issue:** the first new `TestPRReview_Session` row asserts that the rendered prompt holds `lym loom parent-review ask`, which is text of the new autonomous block.
The D4 acceptance criterion for the stencil is a `Review:` line, and the rubric's test-economy rule makes such an assertion a finding.
The row's other assertions already pin the behaviour: the spec's interactivity, the recorded autonomy and the ask.

### [LOW] prcontinue.go's header stays operator-only

**Location:** `_lym/plan/13-pr-continue-autonomy.md:6`
**Issue:** the header says the PR-Review agent runs the verb "to tell the run the operator is satisfied with the review".
Under recorded autonomy, card 13 makes the verb depend on the parent's recorded answer, but leaves this header unedited.

### [NIT] burlerengine's doc does not mention a cleared ask

**Location:** `_lym/plan/05-parent-ask-clear.md:10`
**Issue:** the package doc says an ask moves only to an end state.
Card 5 adds a `Cleared` mark that a reader of the record must know about, and lists `doc.go` under `Uses:` only.

## Rejected

- repo-invariants finding 3 (creators before their first production caller fail the reachability guard across a batch boundary). The rubric's card-order rule flags only a card that uses something a later card creates. Creating a symbol on one card and calling it on a later card is the order the plan spec asks for. Batching is Webster's, and the card gate runs tier0 only.
- No finding against card 18's autonomy line as a Go constant beside `presenceLine`. `orchengine.RenderRoleFile` renders role files, not pointers, and the line stands where the presence line already stands in the orch's own embedded path.

Focus items settled with no finding:

- Fidelity: D1 is carried by cards 1 and 2; D2 by 3 and 4; D3 by 5, 6 and 7, except F1; D4 by 8 to 14; D5, D7, D9 and D10 by 17; D6 by 15; D8 by 16 and 17; D11 by 1 and 3.
  D4's goto clearing is card 14, and Finalize's closed-pull-request halt is card 11.
  The drain marker is card 15, the `role-slack.md` marker rule and the glossary lines are card 17, and the judge re-ask is card 6.
  Every Constraint has a carrying card or needs none; `shellguard/rows.go` holds no row against the named verbs.
- Card order: no card uses a symbol a later card creates. Card 10's stencil naming `lym loom parent-review ask`, which card 12 adds, is text that no scan checks.
- Attack surface: cards 12 and 16 state their bounds, and card 13 only tightens; card 7 is F8.
- Edit typing: no bare `.go` path stands for named symbols, and the changed signatures (`RenderPRReviewPrompt`, `prContinueVerb`) carry the re-sign arrow.
- Board and filing: no card edits this task's entry, files a task or promotes a note; card 17's `lym selfreport create` is no task.
- Research steps: none; card 3 names `TaskSettingChain` and `LookupAutonomy`, and card 10 names its stencil blocks and seams.
