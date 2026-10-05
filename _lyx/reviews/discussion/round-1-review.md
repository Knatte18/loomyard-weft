---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md, D9 and D10
    summary: D9 and the D10 scan forbid asking the operator in the pane, but the discussion role in interactive mode is built to do exactly that, so the decision contradicts a role it names
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md, D7 and Acceptance criteria
    summary: D7 skips a skill that never ends a turn while the criteria say a restarted watcher never skips a step; the two cover different cases but the criteria do not say so
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md, D6
    summary: D6 adds the distill verb and a note gate to compact mode but states no bound for a note never written, a watcher not live or an idle probe that never passes
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md, D1 and D7
    summary: A skipped skill lets a role run without its scribe rules, and the Decisions state neither that nor what bounds it
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md, D2 and D9
    summary: D9 and D10 speak of every role's top-level stencil, but D2's table mixes roles with a top-level stencil and roles that have only a prefix or step files, and no Decision defines the set the directive and the scan cover
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md, D9
    summary: D9 trusts only the parent's message but does not say how a role recognises that a message comes from its parent
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md, D5, D7 and D9 rationales
    summary: Dated history and board-naming narrative in rationales belongs in the support log, which already holds it
---

### [BLOCKING] D9 and D10 contradict the interactive discussion role

**Location:** `_lyx/discussion/decision-record.md`, D9 and D10; the discussion role's `modeRules` in `internal/loomengine/prompt.go`.
**Issue:** D2 lists `discussion`, and D9 renders into every listed role a directive saying "a question to the operator in your pane is never the way forward".
The discussion role in interactive mode is told to ask the operator numbered questions in the pane and wait for the typed reply; that interview is its purpose.
D10's second test fails on any stencil that tells an agent to ask the operator in its pane, so it would fail on the discussion stencil's own mode rules.
The Goal's "never to the operator in its own pane" has the same conflict.
**Fix:** Name the discussion role's interactive mode as the one exception in D9, D10, Goal and Acceptance criteria: its interview questions go to the operator by design, and the directive's operator-ban line is not rendered there.
State the bound: the exception covers interview questions only, and a blocked step still goes to the parent.

### [MEDIUM] D7 against the acceptance criteria

**Location:** `_lyx/discussion/decision-record.md`, D7 and Acceptance criteria.
**Issue:** D7 logs and skips a skill load that never ends a turn within the handoff timeout; the criteria say a restarted watcher "never skips one".
They cover different cases: the criteria are about a restart losing a step, D7 about a load that never confirms.
Read alone, the criteria contradict D7.
**Fix:** Word the criterion so a restart never loses an unconfirmed step, and only a load that exceeds the timeout is skipped, as D7 states.

### [MEDIUM] D6 states no bound for a stuck or absent cycle

**Location:** `_lyx/discussion/decision-record.md`, D6.
**Issue:** `distill` is a new verb, and compact mode gains the note gate and `DEFER`.
The Decision does not say what happens to the request marker when the note is never written, when no watcher is live, or when the idle probe never passes, nor what a stale marker can later do.
**Fix:** State that a request not completed within the existing handoff timeout is abandoned with a log entry, that nothing is cleared or compacted without a passed note gate, and that with no live watcher the verb reports `watcher_live` false and leaves no marker that a later watcher acts on.

### [MEDIUM] A skipped skill is unbounded in the Decisions

**Location:** `_lyx/discussion/decision-record.md`, D1 and D7.
**Issue:** A role whose skill load is skipped runs without `scribe:*` rules.
D1 and D7 say only that the skip is logged.
The attack-surface question needs the Decision to say what this lets through and what bounds it.
**Fix:** Add to D1 that a skipped skill lets the role run on its stencil alone, that the stencil carries the whole procedure so the loss is limited to style and quality guidance, and that the skip is visible only in the log.

### [MEDIUM] The directive's role set is not defined

**Location:** `_lyx/discussion/decision-record.md`, D2, D9 and D10.
**Issue:** D2's table names `recovery`, `bouncer-seed`, `treadle targeting`, `describe` and `burler`.
Some have a template, `recovery` has a prefix stencil, and `burler` has step files and a round-orchestrator template.
D9 says "the top-level stencil of every role in D2's table" and D10's test scans "top-level role stencils", but nothing defines which stencil that is for each role.
**Fix:** Add a Decision line stating that the directive renders into the one stencil that opens each role's session, and that the D10 test identifies it from the registry by a role-to-stencil mapping declared beside the roles.

### [LOW] How a role recognises its parent's message

**Location:** `_lyx/discussion/decision-record.md`, D9.
**Issue:** The directive trusts only the parent, but does not say how the role tells a parent's message from another session's.
**Fix:** State that the sender is matched against the parent name the directive was rendered with.

### [LOW] Narrative in rationales

**Location:** `_lyx/discussion/decision-record.md`, D5, D7 and D9 rationales.
**Issue:** "on 2026-10-05 the orch ignored the `SessionStart` hook's request", "the operator's decision of 2026-10-05" and "The board named the template `orch-note.md`" are history; no requirement depends on them.
The support log's Interview and Question ledger already hold the board-naming point.
**Fix:** Cut the dates and the board-naming sentence from the record; the requirement each rationale carries stays.
