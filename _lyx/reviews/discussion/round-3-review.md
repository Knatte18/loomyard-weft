---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:96
    summary: D7's read-alive disjunct lets an earlier launch's process-ending SessionEnd end a same-id resumed launch died on any not-alive registry tick before SessionStart(resume) lands, contradicting the bound at line 101
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:108
    summary: D8's post-clear reading does not say whether it re-takes readiness and operator activity, and the background's one-fresh-reading guard clause it departs from is not among the Constraints' amended places
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:134
    summary: The Constraints paraphrase of the PATTERN.md line drops of lyx's own and places the residue reading inside a box-reading definition that applies at every reading, so as written it contradicts line 131 and D8's own bound
---

### [MEDIUM] An earlier launch's `SessionEnd` can end a resumed launch died

**Location:** `_lyx/discussion/decision-record.md:96`, against the bound at `:101` and the Death criterion at `:158`.

**Issue:** Line 96 counts a process-ending `SessionEnd` when its line lies past the first-tick size, *or* when this wait read its session id alive.
The second condition keys on the session id, not on the process.
On a resume that keeps the session id (Q7 leaves this open, and the corpus records it), the sequence goes:
tick 1 takes the size, which already holds the earlier launch's `logout`/`prompt_input_exit`/`other` line, and the registry has no entry yet, so the reading is unproven;
the resumed process registers, and a later tick reads that id alive, which satisfies the disjunct for the old line;
the old line is still the newest signal until the new process writes `SessionStart(resume)`, which the support log's Rejected alternatives say comes after the process starts;
on any tick in that window where the registry does not read the process alive, bullet 1 of D7 reads dead.
`processLiveness` in `internal/shuttleengine/claudeengine/prober.go` reads unproven, which is not alive, for an unreadable registry or entry, an unreadable start time, an entry without `procStart` and a reused pid.
So one transient unreadable read of the registry in that window ends the resumed run `OutcomeDied`, attributed to the earlier launch's line.
Line 101 claims an earlier launch never fails a resumed launch, because "this wait neither read the earlier process alive nor saw its `SessionEnd` land".
That reasoning is about the process, while the rule at line 96 is about the id, so the bound doesn't hold.
The Death criteria cover the old line only "with the resumed process not yet registered", which leaves the read-alive window untested.
The disjunct came in with round 2's F1 fix, which closed the not-yet-registered half of that finding and left this half open.
The disjunct buys nothing for D7's own cases: a process this wait read alive writes its `SessionEnd` after that read, which happens at or after the first liveness tick, so the line already lies past the first-tick size.

**Fix:** Count a process-ending `SessionEnd` only when its line lies past the events-file size this wait took at its first liveness tick, with the size taken before that tick's registry read, and drop the read-alive disjunct.
Add a Death criterion: an earlier launch's process-ending `SessionEnd` as the newest signal, after this wait has read the resumed process alive, followed by an unproven registry reading, ends nothing.

### [MEDIUM] D8's second reading is not bounded like the guard reading it stands in for

**Location:** `_lyx/discussion/decision-record.md:108`, `:110`, and Constraints `:130`–`:134`.

**Issue:** `pattern/PATTERN-pane-text-injection.md`'s guard bullet says "One fresh reading takes three facts: the session is ready, no operator was active in the pane within `operator_quiet_s`, and the input box is empty and readable."
Under D8, the write goes out on a different reading: the one taken after the clear key and the redraw settle.
D8 says shuttle "reads the box again" and proceeds on an empty box or the residue reading.
It never says whether that reading re-takes readiness and operator activity, or only reads the box.
The settle is a new window between the last full guard reading and the paste, and an operator who starts typing or opens a menu in it is caught only if that second reading re-checks them.
Constraints lists three amended places in the background: the lead sentence, the lone-`%` sentence and the explicit-key list.
The "one fresh reading" clause, which D8 departs from by writing on a second reading after a key of its own, is not one of them.

**Fix:** State in D8 that the post-clear reading is a full guard reading: ready, no operator activity within `operator_quiet_s`, and the box empty or residue.
Add the guard bullet's "one fresh reading" clause to the Constraints' list of amended places, naming the post-clear reading as the write's guard reading.

### [NIT] The Constraints' PATTERN.md line paraphrase contradicts D8's own bound

**Location:** `_lyx/discussion/decision-record.md:134`, against `:131` and `:113`.

**Issue:** Line 134 restates the `PATTERN.md` line as "paste-submit only and no Enter into a box not holding exactly the sent text".
The real line reads "no Enter of lyx's own into a box not holding exactly the sent text".
Without "of lyx's own", the paraphrase is broken by D8's own bound (line 113) and by Constraints line 131: when a real character survives, the paste-submit's Enter submits it beside the sent text.
The same sentence also says the residue reading "joins the `%` reading inside that definition at the guard reading only".
But the `%` reading lives in `Claude.InputBoxText` and `IdleSession` (`claudeengine/session.go`) and applies at every reading, the post-paste confirm included.
The residue reading is a separate shuttle-side reading taken only after the clear.
The line's wording, "unchanged", is in fact right: the paste-submit's Enter is part of reed's atomic paste-submit, not an Enter of lyx's own, and the confirm sends no key into the merged box.
Only the stated reasoning is off.

**Fix:** Quote the `PATTERN.md` line word for word.
State that the residue reading is taken only after D8's clear, beside the provider's box reading rather than inside it, and that the paste-submit's Enter belongs to the atomic paste the `%` reading already admits.

## Focus verdicts

- D7's `SessionEnd` death path on a resumed launch: a resumed launch can be ended died by the old line, and the Death criteria do not cover that window (F1).
- D8's confirm reading: not a defect.
  D8 line 114 states that the confirm keeps its exact-text reading and takes no residue reading.
  It bounds that path: `ErrSentTextInDraft`, the box quoted, no key, the pending send recorded.
  This matches `confirmPasted`'s `strings.Contains` branch and `pasteVerified`'s `savePendingSend` in `internal/shuttleengine/send.go`.
  Line 115 states what clears it: the operator, with later sends held as a draft and the box quoted.
- `PATTERN-pane-text-injection` amendments: the lead sentence, the `%` sentence and the key list are named.
  The guard's one-fresh-reading clause is missing (F2), and the `PATTERN.md` line's paraphrase does not match (F3).
- Support log, Turn 8 line 30 and the Rejected alternatives on registry evidence (lines 83–87) and treat-as-empty (lines 70–72): each states the rule D7 line 95 and D8 now state, not a superseded one.
  No interview or ledger line records D7's `SessionEnd` offset rule at line 96.
  The support log's `## Review rounds` section still reads `_No rounds yet._` and belongs to the gate, so I don't flag that here.
