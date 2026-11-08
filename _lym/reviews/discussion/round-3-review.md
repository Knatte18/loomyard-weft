---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:62
    summary: "D2 and D4 count a goto into Raddle-Write as a fresh entry as freshRaddleEntry judges it, but freshRaddleEntry never matches a goto entry that shedengine writes, so a goto neither moves the last proposal aside nor is covered by the criteria"
    fix: "Add a Decision line to D4 (decision-record.md, after line 91) that corrects freshRaddleEntry in internal/loomcli/raddlewiring.go:63. The goto branch must match a goto entry as internal/shedengine/goto.go:140 and :263 write it, which has Producer set to the target row, Outcome goto and RoutedTo empty. The test is last.Outcome is OutcomeGoto and last.Producer is NameRaddleWrite; the Raddle-File Stuck branch keeps RoutedTo. Rebuild the fixtures GotoIntoTheRow and GotoElsewhere in internal/loomcli/raddlewiring_test.go:41-42 the same way, since they now carry Producer Raddle-File and RoutedTo Raddle-Write, an entry no verb writes. Add an acceptance criterion under D4 that after lym loom goto --to Raddle-Write on an arras run the last proposal is moved aside and the seat starts a new session. Reword D2 line 62 to say a goto into the row re-arms the perch, without the clause as freshRaddleEntry judges it, unless the correction is recorded."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:62
    summary: D2 says every fresh entry into Raddle-Write re-arms the perch with the segment's full budget, but with no Subject seam that holds only when the perch had settled, and a goto from a perch that has not settled keeps the spent budget and replays the recorded round
    fix: "Rewrite the last bullet of D2 and the criterion at line 249. State that the re-arm covers a perch whose last verdict settled, as after a Raddle-File return or a goto from Raddle-File. For a goto into Raddle-Write while the perch has not settled, state what holds today and accept it with its bound. Bouncer.Call finds a recorded unsettled verdict (internal/shedadapters/bouncer.go:442-470), so it clears nothing and settles round N again over the new proposal. Shed's goto keeps the spent bounce count (internal/shedengine/run.go:641), and a seed Stuck carries GenerationStart only when a Subject seam answers (bouncer.go:1621-1633). A budget-halted perch therefore halts on its next Stuck, and lym loom resume grants one more budget. Name that path in the bound, or add a Decision that gives the perch a Subject seam over the proposal so a changed proposal opens a full generation. Do not leave the universal claim standing."
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:98
    summary: D4's bound and the D2 criterion say the soft-cap rule accepts a round whose open findings carry no design-wrong tag, but the rule counts only open BLOCKING or MEDIUM entries not tagged mechanical, so open LOW or NIT entries and BLOCKING or MEDIUM entries tagged mechanical are filed unreviewed
    fix: "In D4 lines 98-101 and the D2 criterion at line 250, replace open findings carry no design-wrong tag with no open BLOCKING or MEDIUM ledger entry is tagged design-wrong, as designWrongKeys in internal/shedadapters/bouncerfiles.go:282-293 counts them. Add to the bound that the accepted round may still hold open LOW and NIT entries of any tag and BLOCKING or MEDIUM entries the judge tagged mechanical, that the proposal is filed with them, and that the judge's tag is the only thing between a BLOCKING mechanical entry and the filing. Keep the sentence that they are dropped from the record and stay in the round's ledger."
  - id: F4
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:147
    summary: D6's bound justifies the lock's open admission as the filing of the run's own approved shape, but the lock sees only before and after and cannot tell the approved parts from any other discussed drafts, and the admission stays open while the run is halted at Raddle-File
    fix: "Rewrite the Bound paragraph of D6 (lines 147-151). Name the writers as any board writer, since the lock reads no caller. Name the rows as any run status whose producer half is Raddle-File, whatever its state half, so a run halted at Raddle-File keeps the admission open until its entry changes. Replace the claim that the lock passes the filing of the run's approved shape with the true bound: the admitted write leaves only draft discussed members, the fixed project body, no release and no run, so the operator's release stays the gate. Then record, with evidence read from internal/loomshed/raddlefile.go Call (AlreadyFiled compares the slugs of the run's own proposal at lines 144-149, and verify runs the proposal checks otherwise), what Raddle-File does when the converted entry's parts differ from its proposal. The current text says it then takes the filed-entry path, which holds only when the slugs match. State the outcome and its way forward, so Plan does not find it out."
  - id: F5
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:133
    summary: D6 requires order and the part list in dependency order and says the lock reads the parts from the write's result, but dependency order is not unique and the lock cannot re-derive the proposal's tie order, so the check is under-determined
    fix: "In D6 lines 133-145 name the lock's input as the after list that checkRunLocks in internal/boardengine/runlock.go:151 receives, with the filed parts being its entries absent from the loaded list. Define the order check as order and the part list equal the sequence in which those added entries appear in after, with each part after the in-family parts it depends on. State that the lock does not recompute a canonical order, because dependencyOrder in internal/loomshed/raddlefile.go:464-495 keeps proposal order for ties and the lock never sees the proposal."
  - id: F6
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:156
    summary: D6's retry rule writes the mark, clears the status and then writes the store, but a crash after the store write and before the mark reaches Raddle-File with the store already done, where no store write or filing step runs and finish would end the converted project done
    fix: "In D6 lines 156-159 state that the mark and the status clear run at the start of Raddle-File's Call whenever the converted entry is found filed, whether or not the store is done. When internal/loomshed/raddlefile.go:137 finds the store done, Call skips the filing steps (line 174) and goes straight to finish. Add that in the family outcome finish skips EndEntry on every path, retries included, because EndRaddleEntry (internal/boardengine/raddlefile.go:265) would mark the converted project done. Add a criterion that a Raddle-File call over a store-done family filing with no mark writes the mark and leaves the project with no status."
---

### [BLOCKING] A goto into Raddle-Write is not a fresh entry for freshRaddleEntry

**Location:** `_lym/discussion/decision-record.md:62`, `internal/loomcli/raddlewiring.go:63`
**Issue:** D2 names a goto into the row "as `freshRaddleEntry` judges it", and D4 and the D2 criterion at line 249 rest on it.
`freshRaddleEntry` takes a goto only when `last.RoutedTo` is Raddle-Write.
`shedengine.Goto` appends `HistoryEntry{Producer: req.Target, Outcome: OutcomeGoto, ...}` with no `RoutedTo` (`internal/shedengine/goto.go:140`, `:263`).
Other readers of goto entries match on `entry.Producer == row` (`internal/samplerengine/record.go:55`).
A real goto into Raddle-Write is therefore never fresh, so the old proposal stays at its path and the seat's new session starts over an existing output file.
The only test of the branch builds an entry no verb writes (`raddlewiring_test.go:41`).
Raddle-File's own halt names `lym loom goto --to Raddle-Write` as the way forward, so this is the main recovery route of an arras run.
The record cites the function as a settled premise and records no correction.
Round 2's F4 was fixed on that premise ("as `freshRaddleEntry` judges"), and round 1's note on the Raddle-File halt's goto relied on it too; neither round tested the function against a goto entry as the engine writes it.
Nothing a prior fixer disputed or deferred is re-raised here, and no earlier fix regressed.

### [MEDIUM] The re-arm claim holds only for a perch that settled

**Location:** `_lym/discussion/decision-record.md:62-66`, `:249`
**Issue:** D2 gives the re-armed perch a fresh cycle with the full budget on every fresh entry.
That holds after a Raddle-File return or a goto from Raddle-File: the Bouncer's earlier Done ends Shed's episode count (`internal/shedengine/run.go:666`), and with no Subject seam the settled verdict is cleared (`bouncer.go:775`).
It fails when the perch had not settled, for example a budget halt or an escalation the parent answers with `lym loom decision add` and a goto to re-split.
`Call` clears only a settled generation (`bouncer.go:442-470`), so it re-settles the recorded round over the new proposal.
A goto grants no budget (`run.go:641`), and `GenerationStart` is set only with a Subject seam (`bouncer.go:1633`).
D7 itself sends the operator to that goto.

### [MEDIUM] The soft-cap bound misstates what the rule lets through

**Location:** `_lym/discussion/decision-record.md:98-103`, `:250`
**Issue:** `designWrongKeys` counts an entry only when it is open, its severity is BLOCKING or MEDIUM, and its fix tag is not mechanical (`bouncerfiles.go:282-293`).
The record says the rule accepts "a round whose open findings carry no design-wrong tag".
An open LOW design-wrong entry is accepted, and so is an open BLOCKING entry tagged mechanical.
The attack-surface bound must say what the route lets through; this one says less than the code does.
The criterion at line 250 ("never settles ... a round whose open findings include a design-wrong one as an accept") is false for a LOW entry.
The record does state the route, the dropped carry-over and the default-false guard key; those parts are settled.

### [MEDIUM] D6's admission bound overclaims

**Location:** `_lym/discussion/decision-record.md:147-151`
**Issue:** The bound says another writer's exact write is acceptable because it is "the filing of the run's own approved shape".
The lock reads only the entries before and after (`runlock.go:151`), so it cannot know the approved shape.
It admits any discussed draft parts whose list the converted body and `order` repeat.
The admission keys on the producer half of the status alone, so a run halted at Raddle-File keeps it open.
The claim that Raddle-File then "takes the filed-entry path" holds only when the foreign parts carry the proposal's slugs: `AlreadyFiled` compares the slugs of the run's own proposal (`raddlefile.go:144-149`).
The writers and rows are now stated only by implication, and the effect on Raddle-File of a differing filing is a fact the record leaves for Plan to find.

### [LOW] D6's order check is under-determined

**Location:** `_lym/discussion/decision-record.md:133-145`
**Issue:** "In dependency order" has several valid answers when parts are independent.
The filing order comes from `dependencyOrder`, which keeps proposal order for ties.
The lock cannot reproduce that order, and the record gives it no other definition to check against.

### [LOW] D6's retry rule skips the store-done case

**Location:** `_lym/discussion/decision-record.md:156-159`
**Issue:** The rule orders the mark, the status clear and then the store write.
A crash after `FileRaddle` returned and before the mark leaves a done store.
The resumed call then reads `StoreDone` true and skips every filing step, so a rule placed in the store-writing path never runs, and the hook's start write leaves the run status on the converted project.
`finish` also ends the entry done (`raddlefile.go:516`), which for the converted project would mark it done and make D8 refuse its release.
D6's "instead of ending it done" covers the first pass only.

## Rejected

- D6's filing write would be refused by the member-join check (`checkProjectField` in `internal/boardengine/project.go:99`): it judges the project in the after state, where D6 has already cleared the status, so the check passes.
- A crash between the board write and the mark leaves a run status on the converted project that D8 refuses: the filing write clears the status, the failed stop commits no status (`internal/shedengine/failedstop.go:43`), and the resume's start write is followed by the retry that clears it; D6's acceptance criterion at line 262 holds for the store-failure case.
- The perch's hard cap and soft cap need their own wiring in the record: `reviewbudget.Resolve` supplies both for every perch name, so D2's keys are enough.

## Focus departures

None.
