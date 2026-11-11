---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D7 and its acceptance criterion
    summary: D7's keep-every-parent-finding rests on chair prose alone; a parent answer done before the table ends is copied but never consolidated, the merge rule leaves the origin open, and the D7 criterion asks a test to check LLM output
    fix: In D7, add that the merged finding keeps origin parent. Add a Go check that runs once the review table ends with the review written and the round's ask done with findings, before the review is accepted. Give each item of the parent report an id (P1, P2 and so on) when D4 writes it; the chair tags each kept or merged parent finding origin parent and names the ids it covers. The check refuses the review through the existing review gate, naming the missing ids, when any id is uncovered. Add a D7 acceptance criterion that a review missing a parent id fails that gate, and keep the existing D7 criterion as the check's test.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D5 (table-end close) and D4 (copy)
    summary: The ask's close and the report copy hang on the review table ending, so the accepted-review hand-on path of a restarted row skips both, and the record never orders the close before the copy
    fix: "In D5, replace the table-end close with one locked step that runs when the round's review is accepted, on a fresh table and on the hand-on path of a restarted row whose review is already accepted (internal/shedadapters/burlerreview.go:257-259). That step sets a pending-post or open ask to timeout, then copies a done report beside the review (D4), in that order. In D4, change the copy's trigger to that step. Add an acceptance criterion: a row restarted after its review was accepted closes the round's open ask, and refuses a later answer."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D5 (await verb) and D7 (part fill)
    summary: The chair cannot learn what D5 and D7 need it to know; the await answers only ready, no step names the parent report's path, and D7 fills the part only in an asking round, so a not-sent round's chair never writes the review line D5 requires
    fix: "In D5, state that the await's ready true answer carries the ask's state, the not-sent reason when the state is not-sent, and the parent report's path when it is done. In D7, replace filled only in a round that asked the parent with: filled whenever the ask record holds an ask for the round's seed and round, not-sent included, whatever discussion_review_ask_parent now reads."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D4 refusals and D6
    summary: D4's late or superseded answer refusals always point at decision add, which refuses an orch the stored strand admits but that is no longer the resolved parent, and which misleads a fork whose ask a newer open ask superseded
    fix: "In D4, have the refusal of an --ask that is not the latest, while the latest ask is pending-post or open, name answering that ask with --ask <latest> as its way forward. Keep decision add --by parent as the way forward otherwise, naming the run's resolved parent orch as the one that runs it. In D6, add to the bound that a fork of an orch admitted only as the stored strand cannot run decision add."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D2, rule on a not-sent ask
    summary: D2's not-sent rule does not say the round must be later, so a restarted row of the round whose own ask went not-sent matches both D2's ask rule and D3's post nothing rule
    fix: "In D2, change the rule to: A round asks when the latest ask has the same seed, was made in an earlier round, and is not-sent."
  - id: F6
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2, seed identity paragraph
    summary: The seed is told apart by the round-1 focus file's write time, but the record never says the stored stamp keeps the file's full precision, while lym stamps elsewhere are whole-second RFC 3339
    fix: "In D2's seed paragraph, state that the ask record stores the round-1 focus file's modification time at the file system's full precision (RFC 3339 with nanoseconds), compared exactly with the file's current modification time."
  - id: F7
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D5, await verb
    summary: D5 adds the writing verb lym loom parent-review await with no caller guard and no stated bound, as the rubric's attack-surface rule requires
    fix: "Under D5's await bullets, add: Bound: any caller may run the await; it writes only timeout, and only to an ask at or past its deadline, which D4's answer check also does, so it ends no ask earlier than its deadline."
  - id: F8
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md D2, Facts the asking rule rests on
    summary: The facts on the generation record's number and on CarryRecords keeping it explain the rejected generation key; no asking rule reads the generation number
    fix: "In D2's facts list, delete the first bullet (the generation record's Number) and, from the third bullet, the clause and CarryRecords carries the old generation record, so the generation number stays. The support log's rejected alternative on keying by generation already records why."
---

### [MEDIUM] D7's keep-every-parent-finding has no check, and a done answer can go unconsolidated

**Location:** `_lym/discussion/decision-record.md` D7 (lines 193-197) and its criterion (line 250); D5 (lines 159-160); D4 (line 131).

**Issue:** The walkthrough, interactions and bound-breaker lenses each found this case on their own.
The chair can write its review without running the await, or give up after a `ready: false`.
A parent answer made before the table ends then lands `done`.
D5's table-end close touches only `pending-post` and `open` asks, and D4 copies the report beside the review as if the chair had consolidated it.
The review holds none of the parent's findings.
D2's third asking rule then asks the parent to "confirm their fixes", which Burler-Fix never made.
If the Bouncer approves that round, the findings are lost, against D7's stated rationale that a reviewer agent does not overrule the parent.
D7 also allows a merge without saying which `origin` the merged finding carries, yet its criterion requires `origin: parent` on each parent finding.
No production code reads finding origins: `internal/burlerengine/verdict.go` only declares the field, and the review gate is the parse gate at `internal/burlerengine/review.go:271-276`.
So no test can meet the D7 criterion as it stands.

### [MEDIUM] The ask's close and copy are missed on the accepted-review hand-on path, and the close is not ordered before the copy

**Location:** `_lym/discussion/decision-record.md` D5 (lines 159-162) and D4 (lines 131-132).

**Issue:** `internal/shedadapters/burlerreview.go:257-259` hands a restarted row on to the fixer when the round's review is already accepted, without running any table.
A driver that dies after the review is written but before Go closes the ask leaves the ask `open` on that path.
A parent answer made before the deadline is then accepted as `done`, and no review or committed copy holds its findings.
D5's backstop covers only answers after the deadline.
Separately, the record never orders the close before the copy, the interactions lens's path (b).
A copy that runs while the ask is `open` copies nothing.
An answer can then land `done`, and the forward-only lock refuses the close.

### [MEDIUM] The chair cannot learn the ask's state, reason or report path, and a not-sent round gets no consolidation part

**Location:** `_lym/discussion/decision-record.md` D5 (lines 149-157, 167) and D7 (lines 193-194); criteria at lines 244-245.

**Issue:** D5 defines the await's answer only as `ready: true` or `false`.
Yet D5 requires the chair to write a line naming why the parent review did not arrive, and the criteria require that line to name `timeout` or the `not-sent` reason.
D7 has the chair read a `done` parent report that D4 puts beside a per-run ask record under `.lym`, and no decision names how the chair learns its path.
The assistant's paths, by contrast, are named in the chair's prompt at compose time (`internal/burlerengine/assist.go:119`).
D7 fills the parent part "only in a round that asked the parent".
A `not-sent` round sent nothing, so its chair gets no instruction to write the line D5 requires.
A restarted row with the key off has the same gap.

### [LOW] D4's refusal way forwards misdirect in two cases

**Location:** `_lym/discussion/decision-record.md` D4 (lines 124-130) and D6 (lines 177-188).

**Issue:** D4 admits the strand stored in the ask record, but `lym loom decision add --by parent` resolves the parent at call time (`internal/loomcli/decision.go:685-700`, `decisionParent` at `:1684-1695`).
So a fork of an orch that is no longer the resolved parent can answer, but D4's refusal and D6 send it to a verb that refuses it.
A fork answering ask 3 after round 4 opened ask 4 is likewise sent to record a decision rather than to answer ask 4.
PATTERN-refusal-way-forward wants the way forward to be a step the receiver can perform.

### [LOW] D2's not-sent rule lacks the later-round condition

**Location:** `_lym/discussion/decision-record.md` D2 (line 74) and D3 (lines 99-100).

**Issue:** A review row of round k, restarted after its own ask went `not-sent`, matches D2's "same seed and `not-sent`" rule, which asks.
It also matches D3's rule that a row finding any later state posts nothing.
D2's stated purpose, "takes effect at the next round", means a later round only.

### [LOW] The seed's write-time stamp has no stated precision

**Location:** `_lym/discussion/decision-record.md` D2 (lines 62-64).

**Issue:** lym's run stamps are whole-second RFC 3339 (`internal/shedengine/run.go:627-629`).
An ask record that stores the focus file's modification time in that form never equals the file's nanosecond modification time.
Every round would then read the seed as new and ask again.
A restarted row would not find its own ask, which D3 looks up by seed and round.

### [LOW] The await verb has no stated bound

**Location:** `_lym/discussion/decision-record.md` D5 (lines 153-157).

**Issue:** The rubric requires every new verb's Decision to say what the verb can skip or let through and what bounds it.
The await writes an ask's state to `timeout` and has no caller guard.
D5's only Bound line covers the deadline, not the verb.
The behaviour is safe, since it ends no ask before its deadline, but the record does not say so.

### [NIT] D2's facts list keeps generation facts no rule reads

**Location:** `_lym/discussion/decision-record.md` D2 (lines 49-54).

**Issue:** The asking rules key on the seed and on `history`, never on the generation number.
The facts that the generation record's `Number` is the generation, and that `CarryRecords` keeps it across a clear, served the generation key that round 1 rejected.
The support log's Rejected alternatives already records that reasoning.
The remaining facts, that a new generation and a clear both re-seed round 1, stay, since the seed rule rests on them.

## Rejected

- bound-breaker F2, a raised `review_timeout_min` not taking effect until a driver restart: the loop runs each step as a child process (`internal/shedverbs/loopchild.go:20`), and that child rebuilds the wiring (`internal/loomcli/wiring.go:612`), so config is re-read every step, as the existing-paths lens traced.
- bound-breaker F5, the copy's exit set: on the exits that return before the copy no copy exists, so the criterion "a failed review exit archives that copy" is not falsified, and the restarted row's chair still reads the `done` report through the await.
