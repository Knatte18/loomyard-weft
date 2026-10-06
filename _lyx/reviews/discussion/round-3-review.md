---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    location: _lyx/discussion/decision-record.md:112-116
    summary: D12 adds a mechanical gate but no self-check verb, so the record knowingly breaches PATTERN-gate-self-check-parity and leaves the plan writer a pattern violation to ship
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:34-36
    summary: D1 Bound claims an extra Enter never lands on a draft of other text, but the contains-match on a 40 character prefix, or on the whole text when shorter, matches a draft that merely contains it
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:31-34
    summary: D1 says the needle is normalized as sendVerified does but cuts at 40 characters while sendVerified cuts at 48, and the settle has no stated value or injection point for the clock-injected test
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:82-86
    summary: D8 has no acceptance line for a boundary with no later turn end, and the one the watcher is reacting to is not defined
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:88-93
    summary: D9 says a compaction reload starts at the pointer step while D10 and the acceptance criteria type /reload-plugins before it, and no acceptance line covers the doc.go update both decisions require
---

### [BLOCKING] D12 knowingly breaches the gate self-check parity pattern

**Location:** `_lyx/discussion/decision-record.md:112-116` and the Constraints line at `:145`.

**Issue:**
`PATTERN-gate-self-check-parity` requires that a mechanical gate's closure and its CLI self-check verb call the same package function, and `pattern/PATTERN-gate-self-check-parity.md` states "Adding a mechanical gate means adding its verb and its parity check in the same task".
D12 adds a gate entry that parses the review file with `ParseReview`, which is a mechanical gate, and adds no verb.
The Constraints line says outright that "the pattern's verb half is not delivered", so the record does not state how the pattern is met; it states that it is not.
The pairing D12 does give, gate and post-gate parse calling one function, is a different pair from the one the pattern binds.
The repo rule is that a constraint wins over a conflicting scope line, and a violation of a listed constraint is BLOCKING.
The record picks neither resolution and hands the plan writer a breach.

**Fix:**
Decide it in the record, in one of two ways.
Either name the verb as the pattern's own requirement, carved out of the "no new verbs" scope line with a reason, with the verb and the gate calling `ParseReview` and its parity check in the same task.
Or state that `PATTERN.md` and `pattern/PATTERN-gate-self-check-parity.md` are amended in the same commit to exempt a review-parse gate whose parse error is delivered in-session, and add that amendment to Scope and the acceptance criteria.
Either way, replace the "not delivered" sentence in Constraints with how the pattern is met.

### [MEDIUM] D1's Bound is false for a draft that contains the sent text

**Location:** `_lyx/discussion/decision-record.md:34-36`.

**Issue:**
The matching rule counts the box as holding the sent text when its normalized content contains the first 40 normalized characters of the text, or all of it when the text is shorter.
A box holding a draft of other text that contains a short sent text, for example a sent `yes` inside a draft `yes please change the plan`, matches.
The same holds for a draft that shares the first 40 characters, which spawned prompts from lyx commonly do.
Both cases get an extra Enter that submits the draft, so the Bound "it never lands on ... a draft of other text" does not follow from the rule.
The acceptance criteria test only a draft that shares nothing with the sent text.

**Fix:**
State what bounds the match for a short text and for a shared prefix: for example match only when the box content, normalized, starts with the needle, or require the whole normalized text for a text below the cut, and say which.
Add a row for a draft that contains a short sent text to the existing "holds a draft of other text" acceptance line.

### [LOW] D1's cut length disagrees with sendVerified, and the settle is not testable as written

**Location:** `_lyx/discussion/decision-record.md:31-34`.

**Issue:**
D1 says the text is "normalized as `sendVerified` normalizes its needle" and then cuts at 40 characters, while `sendVerified` in `internal/shuttleengine/run.go` cuts its needle at 48 runes.
The two needles of one send then differ without a reason.
The record gives the length, so the length is testable, but the settle is described only as "the one a read needs to see the box redrawn", which has no value, and no line says it is the existing injectable `inputSleep` clock that a test shortens.
The order, Enter then settle then read, is stated in the Decision and in no acceptance line, so a test with an injected clock has nothing in the record to assert.

**Fix:**
Use the one needle length for both, or say why the box read uses 40.
Name the settle as a provider-owned constant, injected the way `sendVerified`'s interval is, and add to the acceptance criteria that the box is read only after the settle that follows an Enter.

### [LOW] D8 leaves the no-turn-end case and "the one being read" undefined

**Location:** `_lyx/discussion/decision-record.md:82-86` and `:182-183`.

**Issue:**
D8 states a behavior for each of the four combinations: more than one later turn end moves the baseline, one that is the one being read reloads, and one that is not or none types nothing and leaves the baseline.
The acceptance criteria cover several, exactly one that is read, and exactly one not yet read, but have no line for a boundary with no later turn end.
"The turn end the watcher is reacting to" is not defined; the watcher takes the newest turn end of the events it read on a tick, and a test that injects a clock needs that stated.
It is also unstated whether two turn ends read in one tick count as several.

**Fix:**
Define the turn end being read as the newest turn end read on the tick, say that two turn ends read in one tick are several, and add an acceptance line that a boundary with no later turn end types nothing and leaves the baseline.

### [LOW] D9 and D10 disagree on where a compaction reload starts, and the doc update has no acceptance line

**Location:** `_lyx/discussion/decision-record.md:88-93`, `:97-99`, `:184`.

**Issue:**
The focus question is settled in D9's favour on one point.
D9's premise A2, its Bound, its acceptance criterion and R4 all treat the skills-step removal as final, with the premise's failure accepted as a risk, and D9 names the `internal/orchengine/doc.go` statement it supersedes.
The wording differs, though: D9 says the compaction sequence "starts at the pointer step", while D10 and the acceptance line type `/reload-plugins` first, and `reload_step` has no value for that step in the persisted encoding `doc.go` describes.
The plan writer has to guess whether `/reload-plugins` is a step before the pointer step or part of it.
Both decisions also require the "The reload sequence" section of `doc.go` to change, but no acceptance line checks it, unlike D13's doc line.

**Fix:**
Write D9 as "the compaction entries skip the skills step and the retry step" and leave the position of `/reload-plugins` to D10, saying whether it is a persisted step.
Add an acceptance line that the "The reload sequence" section of `doc.go` describes `/reload-plugins` and no longer says a compaction loses the session's skills.
