---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md#D2
    summary: D2's parent-answer channel cannot say which decision-class finding was answered and one of its two sources never fires, while D3 forces every held decision finding the block shows an answer beside to resolved, so an unanswered parent question drops out of the review.
    fix: In D2, drop the settled-decision-file source and state that a parent's answer reaches round N only as an Added after Discussion entry; render the entries new since round N-1's snapshot once in the block, not beside each finding. In D3, have the reviewer mark each held decision-class finding either resolved with an answer naming the added entry that settles it, or open and carried; the round-check entry refuses resolved on a held decision-class finding only when its answer names no entry the block listed. Rewrite the D2 and D3 acceptance criteria to match, dropping the decision-file criterion and adding a case where one of two held findings is answered and the other stays open.
    origin: handler
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md#D5
    summary: D5 records a count of Added after Discussion entries as D2's answer baseline, but lym loom decision replace removes added entries when it folds them, so a later answer can sit at or below the stored count and is never rendered; the existing decision snapshot compares titles for this reason.
    fix: In D5, record the titles of the decision record's Added after Discussion entries, as the existing decision snapshot does, in place of their number; in D2, render the entries whose titles round N-1's record lacks. Update the D2 acceptance criterion to name entries new since the snapshot, and add a case where a replace folds entries before a new add.
    origin: lens:existing-paths
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md#D3
    summary: The round-check entry in internal/burlerengine is given only the run directory and round number, but its checks need the acceptance record, the generation record's Amendment mark, the parent's answers and the changes file, all owned by shedadapters, which imports burlerengine; validate-review resolves no worktree to reach the decision record either.
    fix: In D3, state that the review producer builds a round context (whether the round is incremental, round N-1's findings with their dispositions and answered marks, the paths the changes file touches or none, and the round's parent-ask ids) and hands it to burlerengine, and that the round-check entry judges the review against that context. State that validate-review rebuilds the same context through one exported shedadapters builder, and that a file outside a run directory gets the shape checks alone.
    origin: handler
  - id: F4
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md#D6
    summary: D6 records round N's post-fix target id on the first settle but places the check only in settleUndecided, yet routeDecisions settles a decision-class round before settle runs, so such a round may hold a snapshot with no post-fix id, a case D6 counts neither changed nor unchanged.
    fix: In D6, state that the post-fix target id is recorded at the first entry to settleUnlessRefused for every judged round, ahead of routeDecisions, and that a round whose snapshot record holds no post-fix id counts as changed. Add that case to the D6 acceptance criterion on rounds counted as changed.
    origin: lens:existing-paths
  - id: F5
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md#D4
    summary: D4 claims no reviewer or fork can claim the origin parent exemption, but the reviewer itself writes origin parent and parent_ids, and Go checks neither, so in a round with an answered ask any new finding at any severity on any file can carry the exemption.
    fix: In D4, replace the claim with the true bound, that the exemption is open only in a round holding an answered parent-review ask, and there it trusts the reviewer's origin and parent_ids tags. Either accept that bound as stated, or add that the round-check entry refuses a new origin parent finding whose parent_ids are all covered by another finding of the review.
    origin: lens:bound-breaker
  - id: F6
    severity: MEDIUM
    class: consistency
    location: _lym/discussion/decision-record.md#D1
    summary: D1 lists four dispositions but then treats ruling as a disposition entry that carries reason, and D2 treats a ruling as a sign the parent answered, while D3 forces resolved only for an answer D2's block shows, so whether a ruling counts as an answer is undefined.
    fix: In D1, add ruling to the disposition list as a fifth value legal only on a decision-class finding of a round that routes decisions, carrying reason. In D2 and D3, state whether a ruling disposition counts as the parent's answer; recommended, it does not, and is shown beside the finding only as the fixer's report of a ruling that reached it, while D3's answered rule rests on the added entries alone.
    origin: handler
  - id: F7
    severity: MEDIUM
    class: consistency
    location: _lym/discussion/decision-record.md#D1
    summary: D1 has the fix check refuse a report carrying spawned while Go-written reports carry spawned false, and D2 renders the block only when round N-1's fixer-report passes the fix check, so every round after a fixer-less round, including one whose findings all went to the parent, falls back to a full round with today's text.
    fix: In D2, define round N-1 as complete when its fixer-report either passes the fix check or is a Go-written report carrying spawned false whose frontmatter parses; state in D1 that the fix check's refusal of spawned applies to a fixer-written report only. Add a D2 acceptance case where round N-1's fixer was skipped and round N renders the block.
    origin: lens:interactions
  - id: F8
    severity: MEDIUM
    class: consistency
    location: _lym/discussion/decision-record.md#D3
    summary: D3 says ParseCurrentReview keeps the shape checks alone, but those shape checks refuse any top-level key but verdict and findings, so an incremental review carrying the prior list would fail the strict parse in acceptReview and acceptedReview.
    fix: In D3, state that the shape checks admit prior as a third top-level frontmatter key and carries as a finding key, and that only the round-check entry judges whether either may appear.
    origin: lens:bound-breaker
  - id: F9
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md#D3
    summary: D3 lets a carried finding move its location path only to a file the changes file touches, but leaves the rule open when no changes file exists, as when the snapshot cache is gone.
    fix: In D3, state that with no changes file a carried finding keeps its earlier location path and may not move it, and add that case to the D3 acceptance criterion on carried findings.
    origin: lens:bound-breaker
  - id: F10
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md#D6
    summary: D6 says the decision escalation keeps its place ahead of the judge, but routeDecisions runs inside settleUnlessRefused with the judge's verdict and ledger, after the judge.
    fix: In D6, replace the sentence with one saying the decision escalation keeps its place after the judge and ahead of the refusal check and the settle step.
    origin: lens:bound-breaker
  - id: F11
    severity: NIT
    class: consistency
    location: _lym/discussion/decision-record.md#D4
    summary: D4's Bound paragraph restates the regression rule the paragraph above it already states.
    fix: In D4's Bound paragraph, delete the sentence that begins with "A fix's regression in an untouched file is raised through the changed line".
    origin: handler
---

### [BLOCKING] The parent's answer to a decision-class finding cannot reach the finding it answers

**Location:** `_lym/discussion/decision-record.md` D2, second paragraph, and D3's answered-decision bullet.
**Issue:** D2 names two sources of a parent's answer.
The first is round N-1's settled decision file of cause `decision`, which never exists while round N runs.
A decision file is settled only on an accept (`internal/shedadapters/circling.go:232`).
An admitted accept of a decision escalation goes through `acceptRound` (`internal/shedadapters/bouncerdecision.go:173-174`), which returns Done (`internal/shedadapters/bouncer.go:1313-1330`), so the segment ends.
A refused accept is moved aside by `retireContinue` (`bouncerdecision.go:178`).
The second source is the Added after Discussion entries, which belong to the whole record, not to one finding (`bouncerdecision.go:354-373`).
D2 renders them beside every held finding, and D3's round-check entry then refuses any status but `resolved` on each one.
Counterexample: round N-1 holds decision-class findings F2 and F3, and the parent runs `lym loom decision add` for F2 alone.
Round N shows the entry beside F3 too, the gate refuses `open` on F3, and F3 is never escalated again.
That contradicts D2's own "still with the parent" rule and the purpose of carrying open decision-class findings.

### [MEDIUM] The Added after Discussion count breaks under a decision replace

**Location:** `_lym/discussion/decision-record.md` D5 and D2.
**Issue:** `lym loom decision replace` may remove added entries when it folds them into the main decisions (`loom-discussion-spec.md`, Parts no writer authors).
Counterexample: round N-1's record holds a count of 2, the parent folds both entries, then adds one answer.
The record now holds one added entry, which is not past 2, so the answer is never rendered.
The existing decision snapshot records titles (`writeDecisionSnapshot`, `addedSinceSnapshot` in `internal/shedadapters/bouncerdecision.go`), which survive a fold.

### [MEDIUM] The round-check entry cannot reach the facts it judges from the inputs D3 gives it

**Location:** `_lym/discussion/decision-record.md` D3, the round-check entry and `lym burler validate-review`.
**Issue:** D3 places the entry in `internal/burlerengine/verdict.go`, given the run directory and round number.
To know whether the round is incremental, it needs round N-1's acceptance record (`roundComplete`, `internal/shedadapters/burler.go:36-47`) and the generation record's Amendment mark (`amendment`, `internal/shedadapters/amendment.go:82-88`).
To know whether a decision-class finding was answered, it needs the parent's answers.
These are shedadapters' formats, and shedadapters imports burlerengine (`internal/shedadapters/burler.go:14`), so the entry would need an import cycle or a second parser.
`validate-review` skips hub and worktree resolution (`internal/burlercli/cli.go:66-68`), so it cannot find the decision record.

### [MEDIUM] Rounds that routeDecisions settles never record a post-fix target id

**Location:** `_lym/discussion/decision-record.md` D6, first and placement paragraphs.
**Issue:** `settleUnlessRefused` returns through `routeDecisions` before `settle` runs (`internal/shedadapters/bouncer.go:1037-1045`).
A decision-class round that the parent continues therefore never reaches the place D6 puts the check.
A later count meets a snapshot record with no post-fix id, or computes the id from a tree later fixers have edited.
D6 covers a missing snapshot record, not a record with no post-fix id.

### [MEDIUM] The origin parent exemption is open to any finding the reviewer tags

**Location:** `_lym/discussion/decision-record.md` D4, the `origin: parent` exemption.
**Issue:** D4 says the round-check entry refuses any other new `origin: parent` finding, "so no reviewer or fork can claim the exemption".
The reviewer itself writes `origin: parent` and `parent_ids` when it keeps the parent's findings (`contracts/stencils/burler/burler-review-format.md`), and `Origin` is never validated (`internal/burlerengine/verdict.go:73-78`).
In a round with an answered ask holding P1, a reviewer can tag a new NIT on an untouched file with `origin: parent` and `parent_ids: [P1]`, and it passes.
The bound the record states is stronger than the one it builds.

### [MEDIUM] Ruling is used as a disposition the list does not hold

**Location:** `_lym/discussion/decision-record.md` D1, D2 and D3.
**Issue:** D1 lists `fixed`, `fixed-differently`, `disputed` and `deferred`, then says a `ruling` entry carries `reason`.
D2 shows a held finding with "no `ruling` entry and no recorded answer" as still with the parent, which reads a ruling as an answer.
D3 forces `resolved` only on an answer D2's block shows.
So whether a ruling makes a finding answered, and so forces `resolved`, is left open.

### [MEDIUM] Go-written fixer-reports fail the check D2 requires of round N-1

**Location:** `_lym/discussion/decision-record.md` D1 and D2.
**Issue:** D1 has Go-written reports carry `spawned: false` and has the fix check refuse a report carrying `spawned`.
D2 renders the block only when round N-1's fixer-report passes the fix check.
After a round whose fixer was skipped (`skipFix`, `internal/burlerengine/fix.go:140-146`), the next round is therefore full with today's text.
That includes a round whose findings all went to the parent, which is where D2 should show the parent's answer.

### [MEDIUM] The shape checks D3 keeps refuse D3's own prior key

**Location:** `_lym/discussion/decision-record.md` D3, "`ParseCurrentReview` keeps the shape checks alone".
**Issue:** `checkReviewKeys` refuses any top-level key but `verdict` and `findings` (`internal/burlerengine/verdict.go:184-200`).
An incremental review carrying `prior` fails the review gate, and after the cap `acceptReview`'s strict parse fails the round (`internal/burlerengine/handoff.go:55-58`).
That breaks D3's promise that a review passing on the cap is accepted with its findings.

### [LOW] The carried-finding move rule is unstated without a changes file

**Location:** `_lym/discussion/decision-record.md` D3 and D4.
**Issue:** D3 allows a carried finding to move only to a file the changes file touches.
D4 says that with the snapshot cache gone the BLOCKING rule holds alone, but says nothing of carried findings then.

### [LOW] The decision escalation does not sit ahead of the judge

**Location:** `_lym/discussion/decision-record.md` D6, placement paragraph.
**Issue:** `routeDecisions` takes the judge's verdict and ledger inside `settleUnlessRefused` (`internal/shedadapters/bouncer.go:1037-1038`), and every caller runs after a judgment (`bouncer.go:433`, `457`, `508`).

### [NIT] D4's Bound restates its own regression rule

**Location:** `_lym/discussion/decision-record.md` D4, Bound paragraph.
**Issue:** "A fix's regression in an untouched file is raised through the changed line that causes it, as above" repeats the regression rule two paragraphs earlier.

## Rejected

- Walkthrough's F-B at LOW is merged into F4 at MEDIUM: three lenses traced the same gap, and a wrong count of unchanged rounds misfires the stalemate escalation.
- Bound-breaker's claim 14 (no changes file, BLOCKING rule alone) holds for new findings; only the carried-finding part is kept, as F9.
