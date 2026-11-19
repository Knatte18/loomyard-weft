---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-scope
    location: internal/shedadapters/parentseat.go#askSubject
    summary: "The code-review ask goes out at round start and names only the round before's review and fixer report, so no round's own review and fixer report reach the parent before its Bouncer judges, and the last round's never do; the board entry asks for exactly that checkpoint, and no Added after Discussion entry licenses the change the summary calls a design decision."
    fix: "Move the Code-Review parent checkpoint to after the round's fixer report exists and before the round's Bouncer judges it. The ask line names that same round's review file and fixer report, and the Bouncer waits on lym loom parent-review answer (satisfied or a findings file) up to code_review_parent_timeout_min, capped as CapParentReviewWait caps it. The parent's findings must reach the Bouncer's judgement. Keep the discussion seat's round-start ask unchanged. Update codeParentSeat, the shedadapters seat tests, contracts/orchrole/orch-role.md, docs/overview.md and the code_review_ask_parent template comment to match."
  - id: F2
    severity: BLOCKING
    class: design
    origin: assist:assist
    location: internal/driftengine/judgeparent.go#agreementGate
    summary: "The parent-agreement entry is bound to the spawn id, not to the rulings the parent saw. A judge respawned under the same spawn id, after a timeout or a lost chair while the ask is open or done, can write new rulings that pass on the parent's answer to the old summary. After a findings answer, any later rewrite of judgement.yaml passes whether or not it carries the changes."
    fix: "Record with the ask the rulings it presented, such as a digest of judgement.yaml or of the summary taken when the ask is sent, and keep it in the ask's seed or a field beside it. In agreement, pass a done satisfied ask only while judgement.yaml still holds those rulings. A changed judgement after a satisfied answer, or after a findings answer, sends a new ask naming the new summary instead of passing. Add rows to TestParentAgreementGate for a judgement changed after a satisfied answer and one rewritten after a findings answer."
  - id: F3
    severity: BLOCKING
    class: decision
    origin: lens:code-affected-set
    location: internal/driftengine/config.go#ConfigSpec
    summary: "judge_ask_parent, judge_parent_timeout_min, code_review_ask_parent and code_review_parent_timeout_min are CategoryNone. PATTERN-config-layers makes a key that switches off a review or an approval gate, and a timeout whose expiry lets that gate pass, a Guard key, closed to the local layer. The board entry asks for these keys in run_settings, which needs the local layer, so the constraint and the entry conflict. The discussion and pr_review precedent is no written exception."
    fix: "Leave the code as it is and defer to the run's parent: the choice is the parent's and lies outside the fixer's write surface. The parent either declares a change to PATTERN-config-layers that exempts opt-in parent-ask keys and their timeouts from Guard, or records with lym loom decision add that the four keys become CategoryGuard (hub file only) and the run_settings route is dropped. Once the parent decides, apply it in driftengine.ConfigSpec, loomengine.ConfigSpec, the configreg test expectations, both templates' comments and orch-role.md."
  - id: F4
    severity: MEDIUM
    class: decision
    origin: lens:code-scope
    location: internal/driftengine/config.go#configFile
    summary: "The board entry names the key drift_judge_ask_parent; the diff ships judge_ask_parent and judge_parent_timeout_min in drift.yaml. The summary gives a naming reason, but no record entry accepts the change."
    fix: "Leave the code as it is and defer to the run's parent. The parent either records with lym loom decision add that drift.yaml's judge_ask_parent and judge_parent_timeout_min stand for the entry's drift_judge_ask_parent, or asks for the rename. On a rename, change the yaml tags, ConfigSpec, template.yaml, doc.go, the configreg expectation, the config_test rows and orch-role.md together."
  - id: F5
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/driftengine/verify.go#parentAgreement
    summary: "The judge's ask deadline is judge_parent_timeout_min from the first turn end, uncapped, while the judge's own table times out at judgeTimeout (60 minutes from spawn). With the default 30 and a judge that took more than about 30 minutes to rule, or any value of 60 or more, the judge times out while waiting. The run then stops at a Stuck instead of the template's and orch-role.md's promise that a parent who does not answer leaves the rulings as written."
    fix: "Cap the wait parentAgreement hands ParentAgreementGateEntry so the deadline falls before the judge's timeout with room to end its turn: the judge session's OpenedAt (rec.Scope.JudgeSession) plus judgeTimeout, less a margin, less now. The cap works as CapParentReviewWait does for the review chair, and logs a Warn naming judge_parent_timeout_min when it caps. With no room left, record the ask not-sent with a reason and pass. Add a TestDriftVerify_ParentAgreementCarriedWhileAsking or TestParentAgreementGate row pinning the capped deadline."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/parentseatwiring.go#codeParentSeat
    summary: "No test pins what the production seats are built with: the code seat's Tag, Subject and AskEachRound, the discussion seat's Tag, Subject and RecordWrittenAt, or that each seat's AskEnabled reads its own key. The mutation survivors at lines 62-64 and 76-78 show it. TestAskParentSetting_ReadsTheBooleanSetting now passes only codeAskParentKey, so discussion_review_ask_parent is asserted nowhere."
    fix: "Add a tier-0 table test in parentseatwiring_test.go that calls parentSeats with a parentSeatInputs whose setting lookup records the module and key it is asked for, and whose awaitCap is a stub. For each perch, assert the seat's Tag, Subject (prefix), AskEachRound, whether RecordWrittenAt is set, and the key AskEnabled() asks for: discussion_review_ask_parent for PerchDiscussion and code_review_ask_parent for PerchCode."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/driftengine/verify.go#tableWithParent
    summary: "No test shows that the judge's table carries the parent-agreement entry after the judgement entry. TestDriftVerify reaches tableWithParent only with the key off, and the parent tests call parentAgreement or the gate closure directly, so dropping the append in tableWithParent would fail no test."
    fix: "Add a row to TestDriftVerify_ParentAgreementCarriedWhileAsking, or a sibling, that calls tableWithParent with JudgeTable stubbed to an empty table and the key on. Assert that the returned table's Gate holds the judgement entry, then an entry named ParentAgreementEntryName. With the key off and no ask held, assert that only the judgement entry is there."
  - id: F8
    severity: LOW
    class: consistency
    origin: assist:assist
    location: internal/driftengine/judgeparent_test.go#TestJudgeAskLine
    summary: "The redundancy report lists TestJudgeAskLine, TestParentAskLine and TestLoomSegmentSettings_ParentSeatsLandOnTheirSegments as contained, yet each pins what its cover does not, and none carries the keep marker. Their covers build the expected line through JudgeAskLine or ParentAskLine itself, so only the line tests pin the literal text. TestWire_ReviewKeysReachTheEnv checks that a seat is present but not which seat lands on which segment."
    fix: "Add a //testtiming:keep line directly above each func line. Above TestJudgeAskLine: //testtiming:keep pins the drift-judge ask line's literal text, which TestParentAgreementGate builds through JudgeAskLine. Above TestParentAskLine (internal/shedadapters/parentseat_test.go): //testtiming:keep pins the parent ask line's literal text, which the asking-rule test builds through ParentAskLine. Above TestLoomSegmentSettings_ParentSeatsLandOnTheirSegments (internal/loomcli/parentseatwiring_test.go): //testtiming:keep pins which seat lands on which segment, where the wire test checks only presence."
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/driftengine/judgeparent_test.go#TestParentAgreementGate
    summary: "TestParentAgreementGate has no row for a held pending-post ask, which agreementGate posts again; none for a Resolve error, which sendJudgeAsk records as no-parent; and none for an answer landing between the record read and the timeout, where agreement recurses on the done ask."
    fix: "Add three rows to TestParentAgreementGate. A prior ask in state pending-post with a future deadline is posted once to its recorded strand and left open. A seat whose Resolve returns an error records no-parent, posts nothing and passes. For the race, the row's now func answers the open, past-deadline ask with findings through burlerengine.AnswerParentAsk on its second call, and the row expects the entry to fail with the rewrite findings, not to pass as timeout."
  - id: F10
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shedadapters/parentseat_test.go#TestBurlerReviewProducer_Call_ParentSeatAskingEachRound
    summary: "Nothing pins that round 1 of an AskEachRound seat names the subject alone, with no round-0 review or fixer report path. The new test covers only round 2."
    fix: "Make TestBurlerReviewProducer_Call_ParentSeatAskingEachRound table-driven over round 1 and round 2. The round-1 row asks with no prior ask and expects the line ParentAskLine(CodeParentAskLineTag, parentSlug, parentSubject, ask). The round-2 row keeps the current expectation."
  - id: F11
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/driftwiring.go#driftDeps
    summary: "No test pins the judge's parent seat that driftDeps builds: ask files under the anchor, the run's slug, and NotifyParent routing."
    fix: "In the loomcli test that already covers driftDeps (TestDriftDeps_CarryTheDecisionRecord or a sibling), assert that the returned Deps.JudgeParent is non-nil. Its Files must equal burlerengine.ParentReviewFilesOf(location.AnchorPath()), and its Slug must equal seedSlug(location.WorktreeName)."
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/parentreview.go#parentAnswerRefusalText
    summary: "The not-latest refusal still says the decision record may have changed and tells the caller to review the record again. The verb now answers Code-Review and drift-judge asks too, so for those the way forward names the wrong subject."
    fix: "Reword the ParentAnswerNotLatest branch so it names no decision record. Say that a newer ask N awaits an answer, and give the way forward as: review again what ask N's own request names, then answer with --ask N. Update the assertion in parentreview_test.go that checks this text."
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/burlerengine/parentreview.go#ParentAsk
    summary: "The ParentAsk doc still says an ask is to review a round, and that Seed is the round-1 focus file's write time. The record now also holds the drift judge's asks, seeded drift-judge plus the spawn id."
    fix: "Reword the ParentAsk doc as one ask to the parent orch, from a review round or the drift judge. Reword the Seed field's doc to say it names what the ask belongs to: a review generation's round-1 focus file write time, or the drift judge's seed carrying its spawn id."
  - id: F14
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shedadapters/parentseat.go#parentAskDue
    summary: "Code-Review and drift-judge asks now share the run's one ask record. A Discussion-Review generation re-entered by a goto keeps its seed but finds a foreign ask as latest, so latest.Seed != seed asks the parent again although the decision record was not rewritten."
    fix: "In parentAskDue, judge against the latest ask of this seat's own kind, not the run's latest. Filter the record to asks whose Seed is not the drift-judge seed and that this seat recorded: carry the seat's Tag on the ask or match it by the seat's own seeds. Add a TestBurlerReviewProducer_Call_ParentAskingRule row where a foreign-seed ask is latest and the discussion seed's own latest ask is satisfied."
---

### [BLOCKING] The code-review ask misses the round's own review and fixer report

**Location:** internal/shedadapters/parentseat.go#askSubject
**Issue:** The board entry asks for each code review round to mail the parent a `[code-review request]` naming the review and fixer report, and to wait for `lym loom parent-review answer` before the Bouncer judges the round.
The seat instead asks at round start: `burlerreview.go:295` says it asks "before the probe and before any spawn", and `roundParentAsk` runs at `burlerreview.go:375`.
`askSubject` returns the bare subject in round 1, and from round 2 names only `round-1`'s paths.
So the parent never sees a round's own rulings before that round's Bouncer judges, and the last round's fixer report never reaches the parent.
`_lym/landing/summary.md` names this as a design decision and states the trade-off, but the decision record holds no `Added after Discussion` entry that licenses it.

### [BLOCKING] Parent agreement is not bound to the rulings it agreed to

**Location:** internal/driftengine/judgeparent.go#agreementGate
**Issue:** The ask is keyed by `judgeAskSeed(spawnID)`, and the spawn id survives a resume: `judge` keeps `mark.SpawnID` while the session mark is open.
If the judge's chair times out or is lost while the ask is open, the resume can run a fresh judge that writes new rulings under the same spawn id.
F5 makes that likely under the defaults.
`agreement` then passes on a satisfied answer to the old summary.
After a findings answer it passes on any write of judgement.yaml after `AnsweredAt`, because the check is `info.ModTime().After(ask.AnsweredAt)` alone.
Either way, rulings the parent never saw are stored.
The entry asks the judge to wait for the parent's agreement or changed rulings before it writes the judgement.
The summary declares only the narrower bound that "the entry checks that the judge rewrote".

### [BLOCKING] The new parent-gate keys are open to the local layer against PATTERN-config-layers

**Location:** internal/driftengine/config.go#ConfigSpec
**Issue:** `PATTERN-config-layers`' Guard category takes "a review or an approval gate and a timeout whose expiry lets that gate pass".
`judge_parent_timeout_min`'s expiry passes the `parent-agreement` gate (`agreement`: timeout passes).
`judge_ask_parent` and `code_review_ask_parent` switch a parent review on or off.
All four keys are `CategoryNone` (`internal/driftengine/config.go` ConfigSpec; `internal/loomengine/config.go:690-691`).
The board entry asks for them in `run_settings`, which needs the local layer.
The constraint wins over the entry (instruction 1), but satisfying it drops a requested route.
So the call belongs to the parent: amend the PATTERN entry, or record that the keys are hub-only.
The `discussion_review_*` and `pr_review_ask_parent` keys are `none` too, but that precedent is a convention, and the rule document outranks it.

### [MEDIUM] The drift key carries another name than the entry's

**Location:** internal/driftengine/config.go#configFile
**Issue:** The board entry names `drift_judge_ask_parent`; the diff ships `judge_ask_parent` in `drift.yaml`.
The summary's reason, matching `judge_model` and `judge_interactive`, is sound, but it is the writer's own claim, and no record entry accepts it.

### [MEDIUM] The judge's parent wait outlives the judge

**Location:** internal/driftengine/verify.go#parentAgreement
**Issue:** `wait` is `JudgeParentTimeoutMin` minutes, measured from the first turn end that parses a judgement.
The judge's table times out at `judgeTimeout`, 60 minutes from spawn (`internal/driftengine/judge.go:25`).
Nothing caps the one against the other, unlike `CapParentReviewWait` for the review chair.
A judge that took more than about 30 minutes to rule, or any `judge_parent_timeout_min` of 60 or more, reaches its own timeout while waiting.
The pass then ends in the shed's Stuck instead of the outcome `template.yaml` and `orch-role.md` state: "a parent that does not answer by the deadline leaves the rulings as written".
The summary lists the missing cap as a bound, but the documented behavior is false on that path.

### [MEDIUM] Production seat construction is untested

**Location:** internal/loomcli/parentseatwiring.go#codeParentSeat
**Issue:** Dropping `seat.AskEachRound = true`, either `Tag` assignment, either `Subject` or `RecordWrittenAt` fails no test, as the mutation survivors at lines 62-64 and 76-78 show.
`TestLoomSegmentSettings_ParentSeatsLandOnTheirSegments` builds its own seats.
`TestWire_ReviewKeysReachTheEnv` checks presence and then clears the seats.
`TestAskParentSetting_ReadsTheBooleanSetting` now asserts only the code key, so a seat reading the wrong key, or the discussion seat losing its key, goes unseen.

### [MEDIUM] The judge's table is never shown to carry the agreement entry

**Location:** internal/driftengine/verify.go#tableWithParent
**Issue:** The reach report lists `tableWithParent` only under `TestDriftVerify`, which runs with the key off.
The append under `if asks` is the one line that puts the new gate on the judge, and no assertion would catch its removal.

### [LOW] Contained tests that pin text lack the keep marker

**Location:** internal/driftengine/judgeparent_test.go#TestJudgeAskLine
**Issue:** The redundancy report lists `TestJudgeAskLine` (covered by `TestParentAgreementGate`), `TestParentAskLine` (covered by `TestBurlerReviewProducer_Call_ParentAskingRule`) and `TestLoomSegmentSettings_ParentSeatsLandOnTheirSegments` (covered by `TestWire_ReviewKeysReachTheEnv`) as contained.
The covers compare against the formatter's own output, or only against presence.
So each of these tests pins what its cover cannot see, and the rubric asks for a `//testtiming:keep` marker naming that.

### [LOW] Gate rows missing for re-post, resolve failure and the answer race

**Location:** internal/driftengine/judgeparent_test.go#TestParentAgreementGate
**Issue:** The prior asks in the table are only open or done, and the fixture's `Resolve` never errors.
The `case ask.State == ParentAskPendingPost` re-post, the `Resolve`-error no-parent path, and `agreement`'s recursion when `EndParentAsk` returns a done ask (mutation survivor at line 162) have no row.
The injected `now` func can stage the race deterministically.

### [LOW] Round 1 of a seat asking each round is unpinned

**Location:** internal/shedadapters/parentseat_test.go#TestBurlerReviewProducer_Call_ParentSeatAskingEachRound
**Issue:** `askSubject`'s `round < 2` branch on an `AskEachRound` seat has no row.
A change that named round-0 paths in the first ask would pass.

### [LOW] The judge's seat wiring is untested

**Location:** internal/loomcli/driftwiring.go#driftDeps
**Issue:** No loomcli test references `JudgeParent`.
A seat pointed at another record, or one carrying the wrong slug, goes unseen.
The asker and the answer verb must share that record, because `parent-review answer` resolves it from `target.AnchorPath()`.

### [LOW] The stale-answer refusal still names the decision record

**Location:** internal/loomcli/parentreview.go#parentAnswerRefusalText
**Issue:** The ParentAnswerNotLatest branch (around line 167) says "the decision record may have changed since ask %d; way forward: review the record again".
The diff widened the verb's help and file header to cover Code-Review and drift-judge asks, but left this message discussion-only.

### [LOW] The ParentAsk doc is stale for drift-judge asks

**Location:** internal/burlerengine/parentreview.go#ParentAsk
**Issue:** The type doc says "one ask to the parent orch to review a round".
The `Seed` doc says it is the write time of the generation's round-1 focus file.
The drift judge now records asks in the same record, with the seed `drift-judge <spawn id>`.

### [LOW] A foreign ask as latest makes the discussion seat ask again

**Location:** internal/shedadapters/parentseat.go#parentAskDue
**Issue:** `ParentReviewFilesOf(location.AnchorPath())` is one record for every seat, both review seats and the judge's.
`parentAskDue` reads `record.Latest()` across all of them.
After a code-review or drift-judge ask, a Discussion-Review generation re-entered with the same seed sees `latest.Seed != seed` and asks again, though the record was not rewritten.

## Rejected

- Correctness lens F1, duplicate drift-judge mail when `MarkParentAskPosted` fails: a logged double fault that mirrors `shedadapters.postParentAsk`'s deliberate rule that the parent seat never fails the round. It has no reachable common path.
- Correctness lens F3, mtime resolution in the rewrite check: the judge rewrites judgement.yaml at least one await call and one agent turn after `AnsweredAt`, far beyond any filesystem's mtime tick. F2 replaces the mtime check anyway.
- Tests lens item 7 and the mutation survivor at judgeparent.go:71, the `Attempts` budget: pinning an arbitrary retry count tests a constant, not a behavior. F7's table test covers that the entry is carried.
- Mutation survivors at judgeparent.go:138 and :144: the negated conditions only change which Warn is logged. No test can reasonably kill them.
- Assistant F6, a failed no-parent write after a failed post passes the gate: this is the same deliberate never-fail rule as the review seat's `postParentAsk`. The leftover pending-post record is the latest ask until the next ask supersedes it, and no answer can land on a mail that was never delivered. It is not a correctness defect of this diff.
- Assistant F2 at BLOCKING scope: the key name is kept as F4 at MEDIUM `decision`. `drift.judge_ask_parent` carries the entry's meaning under the module's own naming, and only the parent can accept the departure or refuse it.
- Assistant F5 at BLOCKING: the missing keep markers are housekeeping the rubric files as findings, with no gate or PATTERN rule making them blocking. Kept as F8 at LOW.
- Assistant rejection of an uncapped drift deadline: the restart path the assistant relies on is a Stuck the parent must resume, not the documented outcome. Kept as F5.
