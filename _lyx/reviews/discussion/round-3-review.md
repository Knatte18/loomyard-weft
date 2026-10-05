---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:38
    summary: D1 says the skip bound is one the engine owns and also that it is the orch's handoff timeout from D7; a spawned role's launch has no handoff timeout to use
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:239
    summary: Acceptance says only a skill load that exceeds the timeout is skipped, but D1 also skips a skill the provider reports unknown, and D7 persists only the timeout skip
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:194
    summary: D10 never states what text the ask-the-operator scan matches, and the discussion stencil's own Steps 2 and 3 and the orch stencil of D3 contain ask-the-operator wording the scan would hit
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:7
    summary: Goal names the discussion role as the only role that talks to the operator, but D3 has the orch ask the operator about what to build
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:169
    summary: D9 defines the parent as the session that spawned the worktree's run, so a role spawned inside the run (burler, conflict, bouncer) escalates past its real spawner; the record does not say whether that is intended
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:176
    summary: D9 lists the D2 rows with several opening stencils but omits the burler row, whose step files it names in the next sentence
---

### [MEDIUM] Skip bound has two owners

**Location:** `decision-record.md` D1 (lines 38 and 43), D7 (line 148).
**Issue:** D1 says a skill is skipped after "a bound the engine owns", then says the bound is "the single timeout D7 names for every skill load, launch or reload", which D7 identifies as the existing orch handoff timeout.
A launch of a plan, burler or webster role goes through `claudeengine` with a spec and no orch watcher, so it has no handoff timeout.
The plan writer cannot tell whether the engine owns the value, whether the spawning module passes it on the spec, or whether one orch config value governs every spawned role.
**Fix:** State once, in D1, who owns the timeout and how a launch gets it (a spec field or an engine constant); D7 then refers to D1 for it and keeps only the persistence rule.

### [MEDIUM] Acceptance disagrees with the skip rule

**Location:** `decision-record.md` D1 (line 38), D7 (line 147), acceptance criterion at line 239.
**Issue:** D1 skips a skill on a timeout or on the provider reporting it unknown.
The acceptance criterion says "only a skill load that exceeds the timeout is skipped and logged", and D7 persists only the timeout case.
An unknown-skill report that arrives as a local error with no turn (the Open risk) is a skip the criterion forbids and D7 does not persist, so a restarted watcher would re-type it.
**Fix:** Make D1, D7 and the criterion name the same two skip causes, and say in D7 that either cause is persisted as a skip.

### [MEDIUM] D10's scan has no matchable text and collides with two stencils

**Location:** `decision-record.md` D10 (lines 194-197), D3.
**Issue:** `PATTERN-parent-directive` fails a stencil that "tells an agent to ask the operator in its pane", but the record gives no pattern a scan can match.
The discussion stencil carries such wording itself, outside `{{.mode_rules}}`: Step 2 "before asking the operator anything" and Step 3's "ask the operator" lines.
The orch stencil of D3 says to ask the operator about what to build, and the orch is outside D9's mapping.
As written, the exemption covers only the `{{.mode_rules}}` text, so the scan flags the discussion stencil in both modes and the orch stencil.
**Fix:** In D10, state the matched text (a closed phrase list or a marker the writer uses), name the discussion stencil's Step 2 and 3 wording as exempt through the stencil's own registration, and exclude the orch stencil, saying which of the two the test does.

### [LOW] Goal's one exception contradicts D3

**Location:** `decision-record.md` Goal (line 7), D3 (line 81).
**Issue:** Goal says every spawned role escalates to its parent and "the one exception is the interactive discussion role"; D3 has the orch ask the operator about what to build.
The orch is in D2's table but has no parent, so it is a second exception.
**Fix:** Name the orch in Goal as the role that has no parent and answers to the operator.

### [LOW] Parent identity for roles spawned inside a run

**Location:** `decision-record.md` D9 (lines 169 and 181-182).
**Issue:** The parent is "the session that spawned this worktree's run", taken from the origin record that fills `LYX_PARENT`.
That is one name per worktree, so a burler, conflict or bouncer role spawned by webster or the driver escalates to the orch and not to the session that spawned it.
Whether that is intended is not stated, and "escalates to its parent" in Goal reads as the direct spawner.
**Fix:** Add one sentence to D9 stating that the parent is the worktree-level origin for every role, or naming the direct spawner for the in-run roles.

### [LOW] Multi-stencil enumeration omits burler

**Location:** `decision-record.md` D9 (lines 175-177).
**Issue:** The list of rows with several opening stencils names treadle judge and triage, bouncer judge and seed, and webster master and recovery.
The next sentence names `burler-step-*` as step files, so the burler row is built from a prefix and steps but is missing from the list the mapping must cover.
**Fix:** Add the burler row to the list, naming its top-level file as the opening stencil.

## Focus departures

None.
