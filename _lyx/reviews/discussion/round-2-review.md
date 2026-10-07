---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:70
    summary: The review-parse gate is a PassOnCap entry, so D5 releases the fixer on gate pass without a parsing review, and D6, D7 and the acceptance criteria describe an exhausted reviewer gate that cannot occur
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:36
    summary: D1 says no shuttle capability is added, but D6 needs Go to stop a run that is blocked in Wait and remove its strand, and a Run exposes no stop and the engine seam holds no reed access
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:72
    summary: The marker path is one fixed path per round with no attempt token, and D6 ends the round without waiting for the stopped half, so a late marker write from attempt 1 can release the fixer of attempt 2
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:86
    summary: D6 resume needs to know which halves are live without waiting on them, but the existing attach probe blocks on the live run it finds, and the decision does not say how both halves are attached concurrently or what happens to the marker when both are live
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:70
    summary: The marker path rule is stated only for a hub review path, and removal of a pre-existing marker is assigned to the caller step that archives stale outputs, which the standalone lyx burler run has neither a mirrored subpath nor an archive step for
  - id: F6
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:60
    summary: D4 does not bound the fixer's write access to the review file, the overlay fix scope names the review path as writable, and the verdict is parsed from that file after the fixer finishes, so a fixer can edit away a finding instead of disputing it
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:108
    summary: D8 says a disputed finding reaches the next round through prior fixer-report hydration, but that hydration tells the reviewer to re-evaluate deferred items only and no Decision extends it to Disputed
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:34
    summary: D1 says the first terminal event of either run decides the round, but a reviewer reaching done is terminal for that run and never decides the round
---

### [BLOCKING] The review-parse gate cannot do what D5, D6, D7 and the acceptance criteria assign to it

**Location:** `_lyx/discussion/decision-record.md:70`, `:82-83`, `:97-98`, `:182`, `:184`
**Issue:** `ReviewGateEntry` (`internal/burlerengine/verdict.go`) is `PassOnCap: true`, and the reviewer's gate spec is that entry alone (D3).
Once the entry has spent its re-prompt budget, shuttle lets the run through and reports `GateOutcome.Passed` true with the entry in state `let_through`.
Passed is vacuously true when no must-pass entry exists, so the reviewer's `Result.Gate` can never be failing.
Today the strict `ParseReview` after the gate is what fails the round, as a hard error (`engine.go`, "so a file still invalid after the budget fails at the strict parse below").
Four statements in the record rest on the opposite:

- D5 writes the marker "once ... its review-parse gate passed".
  A gate that passed by let-through leaves an unparseable review, so the marker releases the fixer onto a file that does not parse.
  D5's Bound ("only after D3's checks for the current attempt") therefore does not hold, and the acceptance criterion at line 182 ("a parsing review") disagrees with the Decision it tests.
- D6 and the acceptance criterion at line 184 say an exhausted review gate "returns outcome done with that failing gate, no verdict or findings", which `BurlerProducer` maps to a gate-failed `Stuck`.
  That mapping only fires on `Gate != nil && !Gate.Passed`, which the reviewer's run never produces.
  The real outcome of an exhausted review gate is the strict-parse hard error, which `BurlerProducer` routes through `failureExit` (archive and error), not a `Stuck`.
- D7 parses verdict and findings "only after ... the fixer's run reached done", but the marker has to follow a successful parse and the fixer's run finishes after the marker, so the parse moves before the fixer is released.
- D7's "`Gate` carries the failing half's gate outcome" can only ever name the fixer's caller gates.

**Fix:** In D5, make the marker precondition the reviewer's run done, the cluster fork audit passed where it applies, and a strict parse of the review file succeeding after the gate (the check `CheckReviewFile` already performs); drop "gate passed" as the sufficient condition and align the Bound.
In D6, state that a review still unparseable after the gate budget ends the round with the strict-parse error as today, the fixer stopped, and delete the "done with a failing gate" claim from D6 and from the acceptance criterion at line 184.
In D7, say the review is parsed once, by the reviewer half and before the marker, and that `Gate` is the fixer's gate outcome.

### [MEDIUM] D1 claims no shuttle capability is needed, but D6 requires stopping a blocked run

**Location:** `_lyx/discussion/decision-record.md:36`, `:82-84`, `:184`
**Issue:** D1 states that `Start`/`StartGated` and `Wait` already suffice "as Webster's master does".
The master is one run waited on alone; nothing in Webster stops a peer run that is blocked in `Wait`.
D6 and the acceptance criterion require Go to "stop the fixer's run and remove its strand" when the reviewer fails, and to end the round when the fixer dies "without waiting for the reviewer to finish".
A `shuttleengine.Run` offers `Wait`, `Interrupt`, `Send` and `StrandGUID` only; its reed seam is private, `Interrupt` stops a turn not a run, and a blocked `Wait` has no cancel.
`burlerengine`'s `Shuttle` seam (`Run`, `RunGated`) carries no strand removal either.
Removing the strand from outside also makes the blocked `Wait` end in a `died` outcome or, after the status-retry cap, a mechanism error, which the decision does not say to discard in favor of the failing half's outcome.
**Fix:** Replace the "no shuttle capability" sentence in D1 with the actual requirement: `burlerengine` is told a way to remove a strand by GUID (the told-seam shape `websterengine` already uses for `RemoveStrandIfLive`), and the round returns the failing half's outcome, never the stopped half's consequential `died` or error.

### [MEDIUM] A late marker write from a stopped attempt can release the next attempt's fixer

**Location:** `_lyx/discussion/decision-record.md:72`, `:76`, `:184`
**Issue:** D5 fixes the marker at one path per round, a pure function of the review path with no attempt token, and says Go "writes the marker only for the attempt in flight" without saying how.
D6 and the acceptance criterion let the round return when the fixer dies, "without waiting for the reviewer to finish".
Go writes the marker from the goroutine that waits on the reviewer; if the reviewer reaches done just as the fixer's death ends attempt 1, that goroutine can still write the marker after `BurlerProducer` removed it and started attempt 2.
The attempt-2 fixer is then released before the attempt-2 review exists, which is the failure the focus question and the acceptance criterion at line 182 rule out.
**Fix:** Require the round to join the waiting goroutine of the stopped half before it returns (the strand removal in F2 bounds that wait), or carry the attempt token in the marker's content and make the verb treat a marker from another attempt as not ready; state which one, and adjust the acceptance wording "without waiting for the reviewer to finish" to match.

### [MEDIUM] Resume cannot classify liveness with the existing attach probe

**Location:** `_lyx/discussion/decision-record.md:86-90`, `:191`, `:198`
**Issue:** The resume rule acts on the combination of liveness: both live, exactly one live, one gone and one done.
The only probes, `AttachGated` and `AttachIfLive`, wait on a live run they find and return its terminal result; neither reports "live" without blocking.
So "exactly one half live" cannot be observed before that half is waited out, and "stops the live half" cannot be done on a run already waited to completion.
When both halves are live, the two attaches have to run concurrently: the fixer's attach blocks until the marker, which only Go writes after the reviewer's attach returns, so a sequential attach starting with the fixer stalls until the fixer's deadline.
The decision also leaves the marker untouched in the both-live branch while archiving it in the others, without stating why that is bounded (a marker implies a finished and cleaned reviewer, so it should not coexist with a live one).
The Notes for the plan writer say each half's probe "matches its own single output file", which assumes the probe suffices.
**Fix:** In D6, state that resume classifies each half's liveness without waiting (a probe that reports live, done or gone, or an equivalent), attaches to a both-live round concurrently, and in the both-live branch either removes any marker or says why none can exist.

### [MEDIUM] Marker path and cleanup are undefined for the standalone `lyx burler run`

**Location:** `_lyx/discussion/decision-record.md:70`, `:72`, `:148-149`
**Issue:** The marker path is "under `.lyx` at the subpath mirroring the round's review file".
A hub round's review file sits under `_lyx/reviews/<segment>/`, which has a mirror.
The standalone verb takes a free-form `review-path` from the profile (the verb's own example is relative to the target, with state in a separate state directory outside it), so the mirror and its anchor are not defined there.
Removal of a pre-existing marker is assigned to "the caller ... in the same step that archives the round's stale outputs"; `burlercli` has no such step, so a marker left from an earlier standalone run of the same review path would release the fixer at once.
Constraint lines 148-149 restate the hub rule only.
**Fix:** State the standalone marker location in D5 (for example under the told state directory), name the one function that derives the marker path from the review path so the producer and the CLI cannot diverge, and make the engine remove any existing marker at the start of `Run` instead of relying on each caller.

### [MEDIUM] The fixer may edit the review file, and the decision does not bound it

**Location:** `_lyx/discussion/decision-record.md:60`, `:62`, `:98`
**Issue:** D3 bounds the reviewer's write surface to the review file; D4 bounds the fixer only before the marker.
After the marker the overlay fix scope's rules (`fixScopeRules` in `prompt.go`) name the review path as writable ("the target paths plus the two output files (`ReviewPath`, `FixerReportPath`)").
Under the split the review file is the reviewer's output, the verdict and findings are parsed from it after the fixer finishes (D7), and the segment's Bouncer judges it.
A fixer that edits or deletes a finding in the review bypasses the evidence rule D8 sets for a dispute and changes the verdict the round returns.
**Fix:** Add to D4 that the fixer never writes the review file in any fix scope, and that the fixer's rendered write surface lists the fixer-report only; D7's parse can then stay after the fixer's run.

### [LOW] D8's carry-forward of a dispute rests on hydration wording that names deferred items only

**Location:** `_lyx/discussion/decision-record.md:108`, `:186`
**Issue:** D8 says a disputed finding "reaches the next round through the prior fixer-report hydration".
The hydration exists (`priorRoundsBlock`), but it tells the next reviewer to read prior files "to confirm previously-fixed behaviors have not regressed and to re-evaluate their deferred items".
It does not name disputes, and the segment's judge reads no fixer-report.
Whether the fresh reviewer re-raises a dispute therefore depends on wording no Decision or criterion changes, and the only code consumer of a `Disputed` section's content is that sentence.
**Fix:** Either add to D8 that the prior-rounds block names disputed findings alongside deferred items, with a matching line in Scope, or drop the claim that hydration carries disputes and state that nothing but the next fresh review can re-surface one.

### [LOW] D1's "first terminal event decides the round" contradicts the reviewer-done path

**Location:** `_lyx/discussion/decision-record.md:34`, `:82-84`
**Issue:** D1 says "the first terminal event of either decides the round".
The reviewer reaching done is always the first terminal event in a healthy round, yet D6 continues waiting for the fixer in that case; only a failure of either half decides the round early.
**Fix:** Reword D1 to say that the first failure of either half decides the round, and that the round otherwise completes when the fixer's run does.

## Focus verdicts

- Marker across attempts and a resumed process (focus 1): a defect, F3 (late write), F5 (standalone path and cleanup) and F1 (release without a parsing review).
  The hub-shape path is derivable by any process from told paths, and a pre-existing marker is removed before a spawn in the producer path.
- Disputed consumers (focus 2): F7.
  The completion criterion's consumer, the existence check, exists today and no Go code reads the report's sections, so that part is not a defect.
- Start, wait, join, parse and gate order and the gate-to-stuck mapping (focus 3): F1 for the parse and gate order and the mapping, F2 for the stop, F8 for the wording of the decision on which event decides the round.
- Resume liveness combinations and a pre-existing marker (focus 4): the combinations listed are the reachable ones, and a pre-existing marker is archived in every branch but both-live; the gap is how liveness is observed, F4.
