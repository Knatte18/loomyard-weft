---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:98
    summary: D6's resume branch states only the happy path (both halves live, reviewer completes), so the failure branches and D7's review-file comparison have no bound on resume.
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:94
    summary: D6 never says what the round does when stopping a half fails, yet the bound that no round ends with a live half and no live half is spawned twice depends on that removal succeeding.
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:99
    summary: D6 stops and restarts the fixer whenever exactly one half is live, which is the usual resume state (reviewer finished and cleaned up, fixer still working), and gives no rationale or bound for discarding the live fixer.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:83
    summary: D5's Bound says only lyx code writes the marker and then admits the fixer can create it, and the verb returns ready for any existing path it is handed, a second early-release route that neither D5 nor Open risks names.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:165
    summary: The Constraints entry for the marker location restates D5 loosely, dropping the .lyx segment inside the told state directory and the target-relative path.
  - id: F6
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:75
    summary: The fact that the review-parse gate lets a run through after its budget is stated in D5, D6 and D7 instead of once.
---

### [MEDIUM] Resume branch is specified for the happy path only

**Location:** `_lyx/discussion/decision-record.md:96-100` (D6 resume), `:111-113` (D7), `:211` (resume acceptance criterion)
**Issue:** D6 says what the resume branch does when both halves are live and the attached reviewer completes: remove any marker, then run the review-ready function.
The round-level rules around it are written for `Engine.Run`'s goroutines and are not carried to the resume branch.
- An attached fixer that completes as done before the marker exists (the case the first focus question asks about) is settled for a fresh round, in D6 and in the acceptance criterion: stop the reviewer, remove its strand, return the skipped-handoff error.
  For the resume branch nothing says whether the same rule applies, and the resume acceptance criterion lists only the reattach and the one-half-live restart.
- An attached half that dies or times out while the other half is live is not covered.
  Today's attach path respawns on a died or timed-out attached run, and that is wrong here, because the other half is still live.
- D7's comparison needs the bytes the reviewer's review-ready step parsed, and those bytes live in `Engine.Run`.
  The resume branch runs the same step, but today's attach path performs no strict parse and no comparison, so a fixer write to the review file after a resumed handoff reaches the judge and the bouncer facts unbounded.
  The acceptance criterion "a fixer that changes the review file after the handoff ends the round" is worded for every round.
  The only way to meet it on resume is not stated.
- The classification names the cases "both live", "exactly one live" and "one gone and the other done", and omits "neither live", for example both halves finished and cleaned up before the crash.
  The natural reading is archive and spawn as attempt 1, which D6 should say.

On the second focus question: D6 does name the three things `BurlerProducer` needs (the strand guid carried by the probe result, the same strand remover, and the one shared review-ready function in D5), so that part is settled.
What is missing is what it does with them outside the happy path.

**Fix:** In D6's resume paragraph, state that an attached half's outcome is judged by the same rules as a fresh round.
An attached fixer's done before the marker, and an attached half's died or timeout while the other lives, stop the other half and return as in the fresh round.
The review-ready function returns the parsed bytes, and the resume branch runs D7's comparison once the attached fixer is done.
Add "neither live: archive and spawn as attempt 1".
Extend the resume acceptance criterion to the failure branches and the comparison.

### [MEDIUM] Failed removal of a stopped half is not addressed

**Location:** `_lyx/discussion/decision-record.md:37-39` (D1), `:90-94` and `:100` (D6)
**Issue:** D6 stops a half by removing its strand, and D6's Bound is "a round never ends with one half still live, and a half already live is never spawned a second time".
Both bounds hold only if the removal succeeds.
The decision does not say what the round returns when the removal fails (or when the half's liveness cannot be established).
It also does not say whether `BurlerProducer`'s retry may then spawn a second pair of halves beside the leftover one, which is exactly what attempt 2 after a returned died outcome would do.
`websterengine`'s remover, which D1 names as the shape to follow, propagates both a failed removal and a failed liveness probe as an error so that no replacement is spawned beside a leftover agent, and shuttle's attach does the same.
The record's bound is silent where its model states it.
D1 also says "removes its strand" without saying the removal is a no-op for a half that already finished: shuttle removes the strand of a run that reaches done during finalize, so a stop applied to such a half must not fail.

**Fix:** Add to D6 that the stop is remove-if-live, that a removal or liveness error is returned as the round's error in place of the failing half's outcome, and that no retry or respawn happens beside a strand that could not be removed.

### [LOW] Resume restarts a live fixer in the usual resume state

**Location:** `_lyx/discussion/decision-record.md:99`
**Issue:** The reviewer is cheap and short, and when its run reaches done shuttle removes its strand and run directory.
The state a crash most often leaves is therefore reviewer done, fixer live, with the fixer already released and part-way through its fixes.
D6 classifies this as "exactly one half live" and stops the fixer, the expensive half, archives the review and re-runs both from attempt 1.
The fixer's committed and uncommitted edits to the target stay in place while its session dies, and the new reviewer reviews that half-fixed target.
Today a resume continues the live agent.
The Rationale ("a half-finished round has no usable result") does not fit here, since the review is complete and a fixer is live.
The record does not say that restarting is chosen because the tamper comparison cannot run without the parsed bytes (see F1), or that the loss of the fixer's work is accepted.

**Fix:** State the reason for restarting in D6 and what bounds it, or attach to the live fixer after re-running the review-ready step on the finished review.
If restart stays, say so in the Decision, because the record currently presents the most common case as one among several.

### [LOW] D5's bound on releasing the fixer names one route and misses another

**Location:** `_lyx/discussion/decision-record.md:80`, `:83-84`; Open risks `:190`
**Issue:** The Bound says "only lyx code writes the marker" and the following sentence says the fixer can create it, so the first claim is not a bound and the second is the real one.
The verb is also described as reading "one caller-named path" and returning ready as soon as it exists.
The fixer therefore gets released early by naming any existing file (the review file itself exists from the reviewer's first write), with no marker forged.
Open risks names only the forged marker, and the acceptance criteria cover only stale and late markers.
This answers the fourth focus question for the stated bound: the real bound is the fixer's prompt, for two routes, and only one is named.

**Fix:** Reword the Bound to say what the verb and the lyx writer guarantee, and say that early release by creating the marker or by naming another existing path is prompt-enforced.
Add the second route to the Open risks entry.

### [LOW] Marker location differs between D5 and the Constraints

**Location:** `_lyx/discussion/decision-record.md:76` against `:165`
**Issue:** D5 puts the standalone marker "under `.lyx` in the told state directory at the review file's path relative to the target".
The Constraints entry says it "sits under `.lyx` at the mirrored subpath of the review file (under the told state directory in standalone geometry)".
That drops the `.lyx` segment inside the state directory and the target-relative path, so a plan writer reading only the Constraints can place the marker at `<stateDir>/<relative path>`.

**Fix:** Make the Constraints entry say "under `.lyx` in the told state directory, at the review file's path relative to the target in standalone geometry" so the two places state it identically, or refer to D5 once.

### [NIT] The gate's pass-on-cap behavior is restated three times

**Location:** `_lyx/discussion/decision-record.md:75`, `:91`, `:108`
**Issue:** D5, D6 and D7 each explain that the review-parse gate lets a run through once its re-prompt budget is spent.
Each use needs the consequence for its own Decision, but the mechanism itself is explained three times.

**Fix:** State the mechanism in D5 and let D6 and D7 state only their own consequence (the strict-parse error, and `Gate` being the fixer's).

## Focus verdicts

1. Fixer done before the marker while the reviewer is live (D1, D6): for a fresh round the record settles it.
   Go stops the reviewer, removes its strand and returns an error naming the skipped handoff, and the acceptance criterion on a round never ending with a live half names this case.
   Not a defect for a fresh round.
   The resume branch and the failing stop are not covered; see F1 and F2.
2. `BurlerProducer` in D6's resume branch: the strand guid, the remover and the shared review-ready function are all named, so that part is not a defect.
   The branches beyond the happy path are open; see F1.
3. Bound on a fixer write to the review file (D4, D7, Open risks): D7's byte comparison is the bound for a fresh round and is stated consistently with Open risks.
   It is absent on resume (F1).
4. D5 and the Constraints: the marker location is not stated identically (F5), and the stated bound on early release names one of two routes (F4).

## Focus departures

None.
