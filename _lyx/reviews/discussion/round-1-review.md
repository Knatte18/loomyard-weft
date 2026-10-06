---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md Decisions 4 and 6
    summary: A readable transcript with no skill listing, or a truncated one, classifies every skill unknown and silently skips skills that exist; no Decision says whether that degrades instead
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md Goal and Decisions 4 and 7
    summary: On a load-turn timeout there is no turn end and no transcript read, so nothing confirms a skill; the Goal's "skipped at the end of that turn" and Decision 7's "not confirmed" do not hold on that path
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md Decisions 11 and 12
    summary: The persisted step encoding for the retry step is unstated, so an old reload_step 1 can collide with it, and the skills-to-retry list is not tied to the step change that makes it durable
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md Decision 13
    summary: Decision 13 states the post-compaction gap as closed while its premise is unverified; the bound lives only in Open risks, not in the Decision
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md Decisions 5 and 6
    summary: The retry turn's own outcomes are unstated for an unreadable transcript, an error on a retried skill (cause unknown vs not loaded), and a skill the first turn already loaded
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Decision 8
    summary: Decision 8 removes the pane-text classifier but does not state what the transcript check lets through that it did not, namely an unknown skill when the transcript is unreadable
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Decision 14
    summary: The rationale of Decision 14 re-argues a rejected alternative (a short SkillLoadTimeout) that the support log already records
---

### [MEDIUM] Missing or truncated skill listing classifies every skill unknown

**Location:** `decision-record.md` Decisions 4 and 6, Acceptance criteria.
**Issue:** Decision 4 defines `unknown` as "absent from the session's skill listing in the transcript".
Decision 6 degrades to unverified only for no `transcript_path`, an unreadable file, or no load message found.
A transcript that is readable and has a load message, but carries no `skill_listing` attachment (format change, listing not in the part read) or a truncated listing, is covered by neither.
Read literally, Decision 4 then marks every requested skill `unknown`, and Decision 5 skips them all at once with no retry.
That is a silent skip of skills that exist, the opposite of the Decision 6 bound that a format change costs only an unverified load.
The Auto-mode assumption that the listing exists was verified against one session only.
**Fix:** State in Decision 4 or 6 that a transcript with no skill listing is treated as unreadable (unverified degrade), and that a skill is `unknown` by absence only when a listing was found.
State what a truncated listing can let through (a skill past the cut is classified by its Skill call result alone) or that absence from the listing alone never classifies a skill `unknown` without a failed Skill call.
Add the readable-but-no-listing case to the unverified acceptance criterion.

### [MEDIUM] Load-turn timeout has no confirmation source

**Location:** `decision-record.md` Goal, Decision 4, Decision 7.
**Issue:** The Goal says an unknown skill "is skipped at the end of that turn".
Decision 7 says a load turn that does not end in time skips "every skill it asked for that is not confirmed".
On that path there is no turn end, hence no Stop payload and no transcript path, so Decision 4's classification never runs and nothing is ever "confirmed".
The Decision thus skips every skill, including ones the model may have loaded before hanging, while its wording suggests a per-skill filter.
The Goal's wording is also false for the timeout path.
**Fix:** Say in Decision 7 that a timeout skips every skill the turn asked for, with cause `timeout`, and that no classification runs.
State the bound: a skill that did load before the hang is logged as skipped but stays loaded in the session; the prompt pointer is typed afterwards, and no retry follows a timed-out first turn.
Narrow the Goal to "skipped when that turn ends, or when it times out".

### [MEDIUM] Reload step encoding and restart durability are unstated

**Location:** `decision-record.md` Decisions 11 and 12, Acceptance criteria on the orch reload.
**Issue:** The persisted `reload_step` is today an index into the skill list, with the skill count as the pointer step (`internal/orchengine/state.go:90`).
Decision 11 adds a retry step and a persisted list of skills to retry but never says how the retry step is encoded.
If it is step index 1 after the skills step 0, an old `reload_step` of 1 (the second skill in flight) reads as the retry step, not as "past the skills step" which Decision 12 maps to the pointer step.
Decision 12's "falls past the new skills step" is ambiguous for exactly that value.
Decision 11 also does not say that the skills-to-retry list and the step change are persisted together, so a watcher restarted between classification and that write can classify again, which the acceptance criterion forbids as a second retry.
**Fix:** State the step encoding (for example: the retry step is a distinct state, or the retry is identified by a non-empty persisted retry list) and that an old-style value is read as the pointer step whatever the new encoding.
State that the step change and the retry list are written as one state update, so the restarted watcher either re-classifies the same transcript to the same result, or resumes the retry step without re-sending.

### [MEDIUM] Decision 13 presents an unverified premise as closing the gap

**Location:** `decision-record.md` Decision 13, Auto-mode assumptions, Open risks.
**Issue:** Decision 13 says the Skill-tool load "closes the gap" from 70bf30ae8.
The Auto-mode assumptions and Open risks say re-injection after compaction is "not re-verified" and, if wrong, a compacted Webster master runs without its skills.
The rubric's bound rule puts the statement of what a wrong premise lets through in the Decision that introduces it, and Plan-Write reads only the record, but the Decision itself reads as settled.
**Fix:** Rewrite Decision 13's rationale so it states the premise as an assumption and the bound: if re-injection does not occur, a compacted Webster master or other spawned role runs without its skills, nothing detects it, and no fallback exists, as before this change.
Reference the Open risks bullet rather than repeating it.

### [MEDIUM] Retry-turn outcomes are only partly stated

**Location:** `decision-record.md` Decisions 5 and 6.
**Issue:** Decision 5 bounds the retry to one turn and says whatever is still not loaded is logged `not loaded`.
Open cases without a stated outcome: (a) the transcript becomes unreadable between the two turns, where Decision 6 is worded for the load turn and does not say whether the retried skills are then confirmed unverified or skipped; (b) a retried skill whose Skill call errors, which Decision 4 calls `unknown` but Decision 5 logs as `not loaded`; (c) a retry message that would name a skill the first turn already loaded, which cannot arise only if the retry list is derived strictly from the first classification, a fact the Decision does not state.
A pane that dies during either turn is covered by the acceptance criterion "as today".
**Fix:** State that Decision 6 applies to every load turn including the retry (retried skills are confirmed unverified, with the one Warn), that a retried skill is classified with Decision 4's three classes and logged with its class's cause, and that the retry names only skills classified `missing` by the first turn.

### [LOW] Decision 8 removes a guard without stating what passes

**Location:** `decision-record.md` Decision 8.
**Issue:** The rationale argues the classifier's notice can no longer occur, which is the stall cause; it does not state what the replacement lets through.
With an unreadable transcript, an unknown skill passes as part of an unverified load; Decision 6 states that for the load generally but Decision 8 does not tie it to the removed guard.
**Fix:** Add one bound line: an unknown skill on a turn with an unreadable transcript is no longer detected and appears only in the `skill load unverified` Warn's list, with no stall.

### [LOW] Decision 14 re-argues a rejected alternative

**Location:** `decision-record.md` Decision 14 rationale.
**Issue:** The rationale clause "where a short `SkillLoadTimeout` would only wait out a timeout, and the driver spec is built in production code, which a test-only timeout would have to reach" argues against a path not chosen.
The compaction rule for the record keeps rejected alternatives out of `## Decisions`, and `support-log.md` already records this alternative under Rejected alternatives.
The completeness-before-leanness test: the requirement the clause carries, that the stubs exercise the turn-end path and not the timeout, is already in Decision 14's body and in the last acceptance criterion ("no wait on the skill-load timeout"), so nothing is lost by cutting.
The `test-suite-prune-core` disjoint-hunks Bound is a constraint the plan writer needs and stays.
**Fix:** Reduce the rationale to "it exercises the real protocol end to end" and drop the timeout argument from the record.

### Checked, not a finding

- The Open risks hand-check bullet is an operator verification step with matching acceptance criteria (unknown skill skipped without retry, one skills turn per reload); no requirement is missing from the Decisions or Acceptance criteria, and it is not deliberation narrative.
- Decision 12's bounded worst case (one reload skips remaining skills) is stated and legitimate; F3 concerns only the encoding ambiguity.
