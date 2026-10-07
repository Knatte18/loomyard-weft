---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:34-39,88-91
    summary: D1 and D6 give the fixer failure edges dies, times out and never starts, but not the fixer run reaching done before the marker exists, which leaves the reviewer live and contradicts the no-live-half bound
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:37-39,93-95
    summary: The resume branch in BurlerProducer must stop a live half and write the marker after an attached reviewer completes, but only the engine is told a strand remover and no Decision says how the producer gets the strand guid, the remover, or the shared parse-audit-marker step
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:64-65,105-106,180-186
    summary: D7 secures Result.Verdict by parsing before the marker, but the segment judge and the bouncer facts re-read the review file from disk after the fixer, and the only bound on a fixer write to it is the rendered write surface, which no Open risk names
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:93-94
    summary: D6 classifies a half by its own output file as live, done or gone and says only a finished reviewer run leaves a marker, but a run whose output file exists and whose record still reads running is live, and a finished run is never part of the both-live branch
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:82
    summary: D5 bound says the fixer cannot be released early because only Go writes the marker, but the marker is a plain file and the actual bound is the fixer write surface in its prompt, the same prompt-only class the Open risks already admit for pre-handoff edits
  - id: F6
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:76,158
    summary: D5 places the standalone marker under the told state directory at the review path relative to the target while the Constraints place it under .lyx there, and burler standalone instruction files already live under the state directory's .lyx
---

### [MEDIUM] Fixer done before the marker is an unhandled edge

**Location:** `_lyx/discussion/decision-record.md` D1 (lines 34-39), D6 (lines 88-91), acceptance criterion at line 194.
**Issue:** D6 stops the reviewer when the fixer "dies, times out or never starts while the reviewer runs", and D1 says "the round completes when the fixer's run does".
A fixer run completes as done when its single output file, the fixer-report, exists at a turn end, whatever the marker says.
The record itself accepts that the fixer's wait discipline is prompt-enforced (Open risks line 182), so a fixer that writes its report without waiting is a foreseeable case, not an impossible one.
In that case D1 returns the round while the reviewer is live, and D7 sets Verdict and Findings from a parse that never ran.
That breaks the bound "a round never ends with one half still live" (D6) and the acceptance criterion "no round ends with a live half".
**Fix:** In D6, add the fixer reaching done before the marker was written as a failure of the round: Go stops the reviewer and removes its strand, the round returns the fixer's outcome as an error naming the skipped handoff, and Verdict and Findings stay empty.
Add the case to the acceptance criterion at line 194.

### [MEDIUM] The producer's resume branch lacks the capabilities D1 gives only the engine

**Location:** `_lyx/discussion/decision-record.md` D1 (lines 37-39), D6 (lines 93-95).
**Issue:** D1 names the capability Go needs to stop a peer run, a strand remover told to `burlerengine`, and it settles which half's outcome the round returns.
D6's resume branch stops a live half and removes its strand from `BurlerProducer`, which `NewBurlerProducer` builds with an attach seam only.
`Attach`/`AttachGated` remove strands only of respawn-eligible candidates, never of an attachable one, so no existing call does this.
No Decision says the non-waiting probe returns the live half's strand guid, or that the producer is told a remover.
The same branch has Go write the marker "when the attached reviewer's run completes with a strictly parsing review", which D5 and D7 give to the engine's reviewer goroutine, including the cluster fork audit.
Without a stated shared step, the resume path re-implements parse, audit and marker write and can drift from the spawn path.
**Fix:** In D6, state that the probe result carries each live half's strand guid and that `BurlerProducer` is told the same strand remover as the engine.
State that the review-ready step (done outcome, fork audit, strict parse, marker write) is one function that both the engine's reviewer goroutine and the resume branch call.

### [MEDIUM] The review file the judge reads has no bound on a fixer write

**Location:** `_lyx/discussion/decision-record.md` D4 (lines 64-65), D7 (lines 105-106), Open risks (lines 180-186).
**Issue:** D7 parses the review once, before the marker, so `Result.Verdict` cannot be changed by the fixer.
But `BurlerProducer` returns the review path as the round's output pointer, and `shedadapters/bouncerfacts.go` re-parses the on-disk review files (`computeRoundFacts`) for the segment's judge after the fixer has run.
The fixer starts after the marker with the working tree writable (`FixScopeSource`) or the overlay scope, and the only bound D4 states is that its rendered write surface lists the fixer-report alone.
That is a prompt-level bound on the one file that decides convergence, and the Open risks name prompt enforcement only for edits before the handoff.
**Fix:** In D7, have Go keep the bytes it parsed before the marker and, once the fixer's run reached done, compare the review file with them; a difference makes the round return an error naming the file, with Verdict and Findings empty.
Add that to the acceptance criteria as a rejected-write case.

### [LOW] Half classification and the both-live marker sentence disagree with the shuttle disposition

**Location:** `_lyx/discussion/decision-record.md` D6 (lines 93-94).
**Issue:** The classification "live, done or gone by its own output file" reads as if an existing output file means done.
In the shuttle attach disposition a run whose record still reads running and whose output files all exist is attachable, so a reviewer repairing its review in the parse gate is live with its file on disk.
"Done" can only mean no live run and the file present, since a finished run's directory is removed.
The sentence "removes any marker found first (only a finished reviewer's run leaves one)" then describes a marker that can only be a stale one from an earlier attempt, because a finished reviewer is never in the both-live branch.
**Fix:** Define the three classes by the probe result and the file: live is an attachable run, done is no live run with the output file present, gone is neither.
Reword the marker clause to say a marker found in the both-live branch is stale and is removed.

### [LOW] D5's bound overstates what stops an early release

**Location:** `_lyx/discussion/decision-record.md` D5 (line 82), Open risks (line 182).
**Issue:** "It cannot release the fixer early, because only Go writes the marker" holds for lyx code, but the marker is a plain file under `.lyx` that any session with write access can create.
What keeps the fixer from creating it is its prompt's write surface, which is the same prompt-only class as the pre-handoff rule in Open risks.
**Fix:** Reword the bound to say the verb is read-only and only Go code writes the marker, and add the marker to the Open risks line on prompt-enforced discipline.

### [NIT] Standalone marker location differs between D5 and Constraints

**Location:** `_lyx/discussion/decision-record.md` D5 (line 76), Constraints (line 158).
**Issue:** D5 says the standalone marker sits "under the told state directory at the review file's path relative to the target".
The Constraints say it sits under `.lyx` there, and `Engine.Run` already joins `.lyx` onto the told state directory for its instruction files.
**Fix:** Make D5 say under `.lyx` in the told state directory, matching the Constraints.

## Focus verdicts

The directive's five questions, with the verdict reached on each:

1. Gate and parse order. `ReviewGateEntry` is `PassOnCap: true`, and `evaluateGate` records an exhausted entry as let through with the gate passed, so the reviewer's run never reports a failing gate.
   D3, D5, D6, D7 and the acceptance criteria at lines 191, 194 and 195 all match that and the strict parse after it; not a defect.
2. Stopping a blocked run. D1 names the capability (a told strand remover) and the outcome rule (the failing half's outcome, never the stopped half's died outcome or error) for the engine.
   The gap is the producer side and the early-fixer edge, recorded as F2 and F1.
3. Marker lifecycle. Joining each half's goroutine before `Run` returns and removing the marker at the start of `Run` stops a stopped attempt from releasing a later attempt's fixer, and the path is defined for hub and standalone geometry; not a defect beyond F2's shared step and F6's wording.
4. Resume classification. The non-waiting probe is named in Scope and the concurrent attach in D6; the classification wording and the marker clause need F4, and the stop capability needs F2.
5. Fixer write surface. D4 bounds the fixer by prompt only; F3.

## Focus departures

None. Every directive question was reached and answered.
