---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/14-decision-cause-readers.md (Blocked stops)
    summary: The decision halt is withheld only from arbiterstop.Check and batten's HandBack, while the loom Go driver still hands a blocked Discussion-Bouncer stop to the tackler, which can resume it with a goto to its own row, re-judging the round on every cycle.
    fix: "In card 14, add `**Create:**` `plan:internal/arbiterstop#HandsBack` -> `func HandsBack(producer, reason string) bool`, true when `landingshed.HandsBack(producer, reason)` is, or when reason opens with `shedadapters.DecisionHaltPrefix`. Add `internal/loomcli#loomCLI.specFor` and `internal/loomcli#loomCLI.goDriverDepsFor` under `**Edit:**`, so each passes `arbiterstop.HandsBack` in place of `landingshed.HandsBack` (Spec.HandBack and Deps.HandsBack). `battenCLI.wire`'s HandBack seam and `Check`'s DecisionHalt read the same predicate. `TestCheck_RefusesLandingHalts` asserts `HandsBack` true for a prefixed Discussion-Bouncer error and false without the prefix. The bound sentence names the Go driver's tackler path beside the arbiter and the re-step. The combined overlay with F3, F8 and F9 passes `lyx loom validate-plan --overlay`."
  - id: F2
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lyx/plan/12-bouncer-decision-escalation.md (Carry-over)
    summary: D8 says an accept over a re-judged CONTINUE or CIRCLING verdict carries every BLOCKING finding the brief listed into Open risks, but card 12 carries only the decision findings and the ledger's open entries, so a listed BLOCKING review finding whose ledger entry the judge closed is not carried.
    fix: "In card 12's Carry-over paragraph, for a round whose escalation cause is `decision` and whose decision is an accept, `buildCarryOver` also adds every BLOCKING finding of the round's review (the brief's `blocking_findings`) that is not already carried by key. Each is keyed by the finding's id, with its class, severity BLOCKING and rounds `[round]`, whatever the ledger says. Test row 7 gains an admitted accept over a re-judged CONTINUE whose ledger closes the BLOCKING `design` finding, asserting the carry-over still lists it."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:affected-set
    location: _lyx/plan/16-parent-align-row.md (Gate key, and the closing catch-all bullet)
    summary: The rename enumerates Go identifiers but not the old gate key and row name as string literals in test fixtures, so the fixtures that key tier lists or gate configs by merge-in are missed.
    fix: "In card 16, add `internal/landingshed#fixedGateTierLists`, `internal/landingshed#fixerTestRow`, `internal/landingshed#newFixerRunFixture`, `internal/loomcli#TestLandingDeps_VerifyCommandReadsPlanAtCallTime`, `internal/webstercli#verifyGateConfig` and `internal/websterengine_test#gateScenario.restart` under `**Edit:**`. Widen the closing bullet so string literals of the old row name and gate key in test fixtures also take `Parent-Align` and `parent-align`. The combined overlay passes."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/12-bouncer-decision-escalation.md (Routing)
    summary: Card 12 makes standsAsRecorded false for every decision-cause round, so a settled decision accept reached on the replay entry goes through judgmentRefusal and can be re-judged, not take today's path as the card claims.
    fix: "In card 12's Routing paragraph, `standsAsRecorded` returns false for a round whose escalation cause is `decision` only while its decision file is absent or not settled. A settled `decision` accept then stands as recorded on a replay, as today. Add a `TestBouncer_DecisionEscalation` row for a replay over a settled `decision` accept whose CONVERGED review holds a BLOCKING `decision` finding, wanting Done with no re-judge."
  - id: F5
    severity: LOW
    class: design
    origin: lens:way-forward
    location: _lyx/plan/12-bouncer-decision-escalation.md (Pending accept, held refusal; renderDecisionDetail)
    summary: The held-refusal re-escalation names continue as the way forward even when the held refusal is the dropped-key check, which a continue applies too, so the continue halts the segment at its budget instead of starting a round.
    fix: "In card 12, `renderDecisionDetail` with `held` true over a dropped-key refusal says that a `continue` meets the same check, halts the segment at its budget, and that `lyx loom resume` then re-judges the round. Test row 8 gains the dropped-key held case asserting that sentence."
  - id: F6
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/13-discussion-decision-switches.md (What it can skip or let through)
    summary: On routeDecisions' handled-false paths, a round carrying an escalation record or decision file of another cause, the fix row's silence leaves the round's decision findings with no fixer, no notice and no decision brief, and card 13's bound does not say so.
    fix: "In card 13's `What it can skip or let through`, state the bound. On card 12's handled-false paths, a round carrying an escalation record or a decision file of another cause, which only an older binary or a flipped switch leaves, the round's `decision` findings reach neither the fixer nor a parent notice; the other cause's escalation brief points the parent at the round's review."
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/10-darn-writer-entry-check.md (no-parent stop bullet)
    summary: A writer with no parent that a gate re-prompt or a resume reaches is told to keep the stop while it still holds, with no step that checks for the decision note the stop report asked for.
    fix: "In card 10's no-parent bullet, a writer that a gate re-prompt or a resume reaches re-reads the board for a `Decision for <slug>:` note before it keeps the stop. On finding one, it writes the answer into the record as for a parent answer and carries on."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: pattern/PATTERN-cwd-resolution.md:29
    summary: The Bound packages list in the background file still names parentreview once card 5 deletes the package, and no card edits the file.
    fix: "In card 5, add `pattern/PATTERN-cwd-resolution.md` under `**Edit:**`, and a bullet: the Bound packages list drops `parentreview`. The combined overlay passes."
  - id: F9
    severity: LOW
    class: scope
    origin: handler
    location: _lyx/plan/03-drop-parent-review-gate.md (Create, internal/loomcli bullet)
    summary: TestDiscussionWriteRow_GateListIsDiscussionAlone repeats what TestSequence_FullRunBlocksAtMergeIn already asserts, the Discussion-Write table's gate names, which card 3 already edits.
    fix: "In card 3, drop the Create of `plan:internal/loomcli#TestDiscussionWriteRow_GateListIsDiscussionAlone` and the sentence describing it. State instead that `TestSequence_FullRunBlocksAtMergeIn`'s existing table-gate assertion (internal/loomrecipe/sequence_test.go:165) wants `[]string{\"discussion\"}`. The combined overlay passes."
---

### [BLOCKING] Decision halt still reaches the loom Go driver's tackler

**Location:** `_lyx/plan/14-decision-cause-readers.md`, Blocked stops; evidence at `internal/loomcli/arm.go:308` (`loomCLI.specFor`), `internal/loomcli/drivewiring.go:320` (`loomCLI.goDriverDepsFor`), `internal/sheddriver/driverules.go:110` and `decideDrive`, and `internal/sheddriver/tackler.go` (`conclude`, `resumePrecondition`).

**Issue:** Card 14 says the arbiter, the Go driver's escalation and batten's re-step would each resume a decision halt themselves.
It withholds the halt from `arbiterstop.Check` and from batten's HandBack seam only.
The loom spec's `HandBack` and the Go driver's `HandsBack` stay `landingshed.HandsBack`, so `stopFromStatus` marks the halt as no hand-back.
`decideDrive` then sends a blocked stop to the tackler, which may resume it with a goto to Discussion-Bouncer, its current row.
Each resume re-judges the round, and the refusal can hold again at the budget.
The new halt has a longer history, so its stop key is new and `ResumeReturned` never trips.
The tackler is spawned again, especially since the halt's Reason names `lyx loom resume` as the way forward.
That breaks D8's bound that at or past the hard cap a held refusal halts the run until its parent resumes it, with no limit on the tackler and judge spawns it costs.
The guard-reach fork raised it, and the handler confirmed the path independently in the code above.

### [MEDIUM] A decision accept does not carry the BLOCKING findings the brief listed

**Location:** `_lyx/plan/12-bouncer-decision-escalation.md`, Carry-over; answer key D8 ("An `accept` over a re-judged CONTINUE or CIRCLING verdict passes the round as a circling accept does, carrying every BLOCKING finding the brief listed into `## Open risks`"); `internal/shedadapters/bouncercarryover.go` `buildCarryOver`, which carries open ledger entries only.

**Issue:** `judgmentRefusal` refuses a CONVERGED verdict over any BLOCKING review finding.
So an accept over a round whose review holds a BLOCKING `design` finding passes only after a re-judge to CONTINUE or CIRCLING.
The judge may close that finding's ledger entry, since the fixer fixed it, but no fresh reviewer has confirmed the fix.
Card 12's carry-over adds the round's `decision` findings beside the ledger's open entries.
A BLOCKING finding the brief listed, and the parent accepted over, therefore never reaches `## Open risks`, against D8's explicit sentence.

### [MEDIUM] Card 16 misses fixture literals of the old gate key and row name

**Location:** `_lyx/plan/16-parent-align-row.md`, Gate key and the closing catch-all bullet; evidence at `internal/landingshed/verifygate_test.go:129`, `internal/landingshed/mergein_test.go:543`, `internal/loomcli/landingdeps_verify_test.go:53`, `internal/landingshed/fixerrun_test.go:87,95`, `internal/webstercli/verify_integration_test.go:51` and `internal/websterengine/runlevel_git_test.go:70`.

**Issue:** The card lists Go identifiers and envkit's two gate-key entries.
Its catch-all covers test members that name the identifiers, but not test fixtures that hold the row name or the gate key as string literals.
`fixedGateTierLists` keys the landing tier list `merge-in`, and the renamed fixer test reads it by that key.
`VerifyCommand("merge-in")`, the fixer run fixture's row and gate key, and two gate.yaml fixtures stay on the old key.
These are one gap in the enumeration method, not separate items.

### [LOW] The settled-accept handled-false branch no longer takes today's path on a replay

**Location:** `_lyx/plan/12-bouncer-decision-escalation.md`, Routing; `internal/shedadapters/bouncer.go` `Call` (the `standsAsRecorded` branch) and `settledEntry`'s replay action.

**Issue:** The card claims a settled accept takes today's path.
But it makes `standsAsRecorded` false for every round with cause `decision`, settled or not.
A replay over a settled `decision` accept then reaches `judgmentRefusal`, which refuses a CONVERGED round whose review holds the BLOCKING finding the parent accepted over, and re-judges a settled round.
The entry is narrow: it needs a settle record that is not live, or a harvest over a generation record.
It still contradicts the card's own claim.

### [LOW] Held dropped-key refusal names continue, which halts instead

**Location:** `_lyx/plan/12-bouncer-decision-escalation.md`, Pending accept held-refusal branch and `renderDecisionDetail`; `internal/shedadapters/bouncer.go` `judgmentRefusal` (dropped-key check for round 2 or later).

**Issue:** The held-refusal brief names `continue` as the way forward whatever the refusal is.
Over a dropped-key refusal, a `continue` applies the same check, re-judges and halts the segment at its budget.
D8 accepts that halt, but the brief does not tell the parent about it.
The way-forward fork rated this MEDIUM.
The handler rates it LOW: the path still progresses through the parent's resume, so only the brief's wording is inaccurate.

### [LOW] Decision findings go unrouted on the handled-false paths, with no stated bound

**Location:** `_lyx/plan/13-discussion-decision-switches.md`, What it can skip or let through; `_lyx/plan/12-bouncer-decision-escalation.md`, Routing; `internal/shedadapters/burlerfix.go:67`.

**Issue:** With `decisions_escalated` set, the fix row withholds `decision` findings and posts nothing.
Card 12 skips the `decision` escalation for a round carrying an escalation record or decision file of another cause.
On those paths the findings reach no one.
The state arises only from an older binary or a flipped switch, but the card's bound claims the Bouncer's escalation always covers them.

### [LOW] Darn writer with no parent never re-checks for the requested note

**Location:** `_lyx/plan/10-darn-writer-entry-check.md`, the no-parent stop bullet.

**Issue:** The way forward is a decision note, then `lyx loom resume`.
The stencil re-reads the board only on a parent message, which a writer with no parent never gets.
A re-prompted or resumed writer is told to keep the stop while it holds, with no step that checks whether the note now answers it.

### [LOW] PATTERN-cwd-resolution background file keeps a deleted package

**Location:** `pattern/PATTERN-cwd-resolution.md:29`.

**Issue:** The Bound packages list names `parentreview`, which card 5 deletes, and no card edits the file.

### [LOW] Card 3 plans a test the sequence test already covers

**Location:** `_lyx/plan/03-drop-parent-review-gate.md`, Create and the `internal/loomcli` bullet; `internal/loomrecipe/sequence_test.go:165`.

**Issue:** `TestSequence_FullRunBlocksAtMergeIn` already asserts that the Discussion-Write discussion table's gate names equal the row's gates.
Card 3 already edits that test.
The new `TestDiscussionWriteRow_GateListIsDiscussionAlone` pins the same behaviour a second time.

## Rejected

- affected-set finding 3 (status commit pathspec still stages `shedrun.AlignmentDir`, `internal/loomcli/wiring.go:113`): not a defect. Staging a directory nothing writes is a no-op, and it keeps an in-flight run's archived alignment attempts committed for the reports D4 keeps reading. D1's "wiring in `internal/loomcli`" is the agent's spawn wiring, which card 3 deletes.
