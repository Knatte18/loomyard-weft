---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:203
    summary: D10 gives the ask-the-operator phrase list only as examples and does not say how the scan tells an imperative from a forbidding sentence
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:202
    summary: D10 does not say which stencils the orch exemption covers, so the start, resume, adopt and note stencils are unsettled
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:46
    summary: D1 says a spawning module that sets no skill-load timeout gets the engine's default but names neither the owning engine nor how the watcher obtains the timeout
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:202
    summary: D10's load-a-skill scan has no stated match, and the orch stencil names ly:board as a pointer, which a naive scan would flag
---

### [MEDIUM] Ask-the-operator scan match is not stated

**Location:** `decision-record.md` D10, third paragraph.
**Issue:** The Decision says the scan matches "a closed list of imperative phrases the registry declares" and gives two phrases "for example".
A closed list given as examples is open for the plan writer.
The exemption for "a stencil sentence that forbids or bounds asking" names no mechanism, though the discussion stencil's own wording needs one: Step 2 has "before asking the operator anything", Step 3 has "you MAY ask which module boundary" and "you MUST NOT ask the operator to enumerate".
Without a stated rule the test either flags the discussion stencil or exempts by sentence, which is the judgment a mechanical scan cannot make.
**Fix:** State the phrase list in D10 and the exemption rule (a match is skipped when the same sentence carries a negation or a bounding word such as "do not", "never", "must not", "before"), and state that the discussion stencil's Step 2 and 3 wording passes it.

### [MEDIUM] Orch exemption does not name its stencils

**Location:** `decision-record.md` D10, `PATTERN-parent-directive` entry and third paragraph.
**Issue:** "The orch stencil is outside the rule" and "the orch stencil is not scanned" name one stencil, but the orch family also has the start, resume, adopt, note and handoff-request stencils (D3, D5).
The orch role stencil of D3 says to ask the operator about what to build, and the family files may too.
The scan's scope is therefore unsettled.
**Fix:** Exempt every stencil under `contracts/stencils/orch/` by family, and say the scan covers every other registered stencil.

### [MEDIUM] Skill-load timeout owner and delivery are unsettled

**Location:** `decision-record.md` D1, last paragraph; D7, last line of the Decision.
**Issue:** D1 makes the timeout "one provider-neutral spec field" and says a module that sets none gets "the engine's default", without naming whether `shuttleengine` or `claudeengine` owns that default.
The watcher has no spec at a reload, and `orchengine` carries skill names only, so D1's "single bound for every skill load, launch or reload" does not say how the watcher is handed the timeout.
D7 says the watcher's timeout is the handoff timeout "as D1 sets", which only works if the watcher is told it.
**Fix:** State in D1 that `claudeengine` owns the default as a constant beside the skill-load sequence, that a spawning module may override it on the spec, and that the orch passes its handoff timeout both on its launch spec and to the watcher, which hands it to the skill-load sequence at each reload.
D7 then refers to that sentence.

### [LOW] Load-a-skill scan match is unstated

**Location:** `decision-record.md` D10, second paragraph.
**Issue:** The test "fails on a load-a-skill instruction" with no match rule.
The orch stencil names `ly:board` as the place for board mechanics (D3), which a scan for skill names would flag.
**Fix:** State that the scan matches an imperative load verb ("load", "invoke") followed by a skill name or the `Skill` tool, and that naming a skill as a pointer is not a match.
