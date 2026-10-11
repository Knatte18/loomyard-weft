---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md, Decisions, D9 first bullet
    summary: D9 drops the fasit and Do-not-flag dispute grounds with the judge as the only stated bound, but under D11 auto-accept at a spent budget the judge's ruling settles nothing and the applied change lands unbounded and unlisted in Open risks
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md, Acceptance criteria against D11 Bound
    summary: Three of D11's stated bounds have no acceptance criterion that fails when the bound is removed - a recorded decision settling as recorded whatever the flag, a recorded escalation standing over auto-accept, and a restart replay escalating
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md, D3 last paragraph, D4 fifth bullet, Acceptance criteria restart line
    summary: The retry exits read as died twice or timed out once, while today's rule they claim to keep is two consecutive died-or-timed-out attempts, and D3 states the reviewer's died exit only
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md, D3 and D4 exit bullets
    summary: Neither half's exit list states the seat-could-not-be-stopped exit, which today returns its error bare with nothing archived and no retry
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md, D6 first paragraph against D4 zero-findings bullet
    summary: D4 spawns the fixer on a zero-findings round whose gates fail with the gate findings as its work, but D6's prompt contents hold only the review and the targets its findings name, so that fixer's prompt has neither its work nor a target
---

### [MEDIUM] D9's dispute-ground removal is unbounded under auto-accept

**Location:** `_lyx/discussion/decision-record.md`, Decisions, D9 first bullet; D11; Open risks.

**Issue:** D9 removes a guard: the fixer can no longer dispute a finding as `premise false` against the fasit or the rubric's `Do not flag` list, so a reviewer finding the rubric excludes, or one contradicting the fasit, is applied to the target.
The rubric's attack-surface rule requires the Decision to state what this lets through and what bounds it.
D9 names the bound only as "the Bouncer's judge, which reads the rubric, rules on a finding that breaks them".
The judge rules after the fixer has already changed the target, and its only lever is a CONTINUE that buys another round.
Under D11 with a spent budget, and always under `<perch>_max_bounces: 1` with auto-accept (the hub target D10 names), that CONTINUE is settled as an accept: the rubric-breaking change is committed and the segment goes Done, with no round left to revert it.
D11's carry-over moves the open findings into `## Open risks`, but the applied change is not an open finding, so nothing records it.
Neither D9 nor `## Open risks` states this; the Open risks lines cover vague `fix` text and the missing PATTERN overview only.

**Fix:** In D9's first bullet, add a Bound sentence: what the change lets through is a fasit-contradicting or rubric-excluded finding applied to the target, bounded by the judge's CONTINUE and a later round reverting it, and not bounded at all when the round's CONTINUE is auto-accepted at a spent budget.
Add a matching `## Open risks` line naming that residue under auto-accept, so the plan writer and operator see it.

### [MEDIUM] D11's recorded-decision and recorded-escalation bounds have no criterion

**Location:** `_lyx/discussion/decision-record.md`, Acceptance criteria, held against D11's Bound paragraph and its last sentence.

**Issue:** D11 states these bounds, each of which keeps auto-accept from overriding a state a human or an earlier call already set:

- "A recorded decision settles as recorded whatever the flag's current value";
- "an escalation already recorded for the round stands: auto-accept never overrides an Awaiting the parent was told about";
- "a replay of an earlier round's recorded CONTINUE escalates as today, since a replay follows ... a restart between judging and settling".

The auto-accept criteria cover the CIRCLING cause, the failing bounce seam, the fix-gate-exhausted replay and the flag-false default, but none would fail if an implementation auto-accepted over an existing escalation record (the parent was told and is now overridden), re-settled a parent's recorded decision as an automatic accept after the flag was flipped on, or auto-accepted the replay that follows a restart between the judge's write and the settle.
The fix-gate-exhausted criterion does not cover the restart replay: an implementation can tell the two apart by whether the round's fixer report is present, and pass one while failing the other.
D11 also says "No escalation record or parent notice is written", and the matching criterion checks the notice only.

**Fix:** Add to `## Acceptance criteria`:

- With auto-accept true and the budget spent, a round that already carries an escalation record returns the same Awaiting and writes no automatic accept.
- A round with a recorded parent decision settles as that decision whether `<perch>_auto_accept` is true or false at settle time.
- With auto-accept true, a run killed after the judge wrote a CONTINUE at a spent budget and before settling escalates on resume rather than accepting.

Amend the existing auto-accept criterion to read "and neither an escalation record nor a parent notice is written".

### [LOW] Retry exits misstate today's two-attempt rule

**Location:** `_lyx/discussion/decision-record.md`, D3 last paragraph, D4 fifth bullet, Acceptance criteria restart line.

**Issue:** D4's bullet reads "fixer died twice, timed out, or ended with an unrecognized outcome: hard error, as today", and the restart criterion repeats "whose fixer died twice or timed out".
Today's rule, which "as today" points at, is the attempt-1/attempt-2 retry in `BurlerProducer.Call` (`internal/shedadapters/burler.go`): a died or timed-out first attempt retries, and only two consecutive died-or-timed-out attempts are a hard error.
Read literally, D4 makes a single timeout a hard error.
D3 states the reviewer's exit as "dies twice" only, leaving the reviewer's timed-out and unrecognized-outcome exits unstated, though D2 says the bounded retry on died or timed-out stays per half.

**Fix:** In D4's bullet and the restart criterion, write "two consecutive died or timed-out attempts, or an unrecognized outcome".
In D3's last paragraph, write "or that ends two consecutive attempts died or timed out, or with an unrecognized outcome" in place of "or that dies twice".

### [LOW] The seat-not-stopped exit is unstated for both halves

**Location:** `_lyx/discussion/decision-record.md`, D3 last paragraph and D4 exit bullets.

**Issue:** The focus directive asks whether every exit of the fix row is stated, with what it archives or keeps.
Today a half that could not be stopped (`burlerengine.ErrHalfNotStopped`) is returned bare, with no archive and no retry, because the session may still be writing the round's files.
D4's exit list omits it, and so does D3's reviewer exit; the split adds a second place it occurs, and a plan writer following D4's archive rules (archive the fixer report on a fixer failure) would archive under a live writer.

**Fix:** Add a D4 bullet: "fixer that could not be stopped: error returned with nothing archived and no retry, as today".
Add the same clause for the reviewer to D3's last paragraph.

### [LOW] D6's prompt contents omit the gate-findings fixer's work

**Location:** `_lyx/discussion/decision-record.md`, D6 first paragraph, held against D4's zero-findings bullet.

**Issue:** D4 says a zero-findings round whose gates fail "spawns the fixer with the gate's findings as its work".
D6 lists the fixer prompt's contents as the review file, "the target paths the review's findings name", the write surface and the rules.
On that path the review has no findings, so D6's prompt names no target and carries no work; the gate findings D4 relies on are not among D6's inputs.

**Fix:** In D6's first paragraph, add to the prompt's contents: "on a round whose review has no findings and whose gates failed, the gate's findings and the target paths they name".

## Focus verdicts

- Focus 1 (gates after a fix gate is exhausted): settled, not a defect.
  D4's zero-findings bullet runs the fix row's gates over the unchanged target before any Go-authored report, its Bound states what the skip lets through, and the fourth acceptance criterion pins the failing-gate case against CONVERGED and auto-accept.
  At budget 1, the exhausted round's counted Stuck reaches the Bouncer's re-bounce, which `episodeStuckCount` blocks at the generic budget as D10 states.
- Focus 2 (every fix-row exit): F3 and F4.
- Focus 3 (zero-findings trust): settled, not a defect.
  An APPROVED review with findings under a misnamed top-level key is refused by D7's unknown-key check whether or not it has prose blocks, and the matching acceptance criterion names that case.
- Focus 4 (every D10 and D11 bound has a criterion): D10's bounds, including the budget-1 gate-exhaustion halt under auto-accept, each have one; D11's do not, F2.
