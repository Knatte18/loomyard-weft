---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:106-107
    summary: D6 states write, overwrite and clear for the pre-fix head but not whether reset consumes it or whether run --fresh clears it, so a stale value from an earlier gate stays usable
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:64-67
    summary: D3 bound says the hook changes nothing but the stdin, yet a prefixed command is what the permission rules then match, and the Decision states no effect on allow and deny matching
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:211
    summary: R4 says the operator's guard accepts the by-path residual, but no operator answered in this session and Q13 is an auto-pick; D6 Bound omits the residual
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:92
    summary: D5 names lyx webster run as the step that has Master rewrite absent contract files without stating the condition under which run re-enters Master for a run that reached exit
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:214-231
    summary: Acceptance criteria lack the failed-tool-result case for D5 and D6, the pre-fix head lifecycle, and the way forward after accepting on absent files
---

### [MEDIUM] Pre-fix head lifecycle leaves reset and --fresh open

**Location:** `decision-record.md` D6, A7.
**Issue:** D6 fixes when the value is written (first failed evaluation per shuttle run), overwritten and cleared (passing gate).
It does not say whether `reset --to pre-fix` consumes the value, and "changes no Webster state besides the reset itself" implies it stays.
A value left by an earlier gate then survives the reset and, until the next failed evaluation, a later `--to pre-fix` can still pass the ancestor check and discard later commits.
`run --fresh` is not stated to clear it either.
**Fix:** State that a successful `reset --to pre-fix`, `reset --to start` and `run --fresh` clear the persisted value, and that a value left by an earlier gate is otherwise used as recorded.

### [MEDIUM] D3 prefix versus permission-rule matching

**Location:** `decision-record.md` D3.
**Issue:** The attack-surface question asks what a changed guard lets through or blocks.
D3 bounds the rewrite as touching only stdin, but the rewritten command is what allow and deny rules then see, so a prefix can stop an allow rule from matching or change how a deny rule matches a compound command.
**Fix:** Add to D3's bound that the rewrite must leave allow and deny matching of the original command unchanged, with a test; if Claude Code matches on the rewritten command in a way that breaks this, the fallback condition applies.

### [MEDIUM] R4 attributes acceptance to an operator who did not answer

**Location:** `decision-record.md` R4, D6 Bound.
**Issue:** "the operator's guard accepts this residual" contradicts the support log: no operator answered, and Q13 is an auto-pick.
D6's Bound lists what the verb can discard but omits that a path the run wrote and another party edited afterwards is discarded.
**Fix:** Reword R4 as an auto-pick, and state the by-path residual in D6's Bound.

### [LOW] D5 re-entry condition

**Location:** `decision-record.md` D5.
**Issue:** The Decision asserts `lyx webster run` has Master write the files again, without stating that `run` re-enters Master for a run that reached its exit audit.
**Fix:** State the condition and have the Plan producer verify it in code, with the way forward naming whichever step does re-enter.

### [LOW] Acceptance criteria gaps

**Location:** `decision-record.md` Acceptance criteria.
**Issue:** No criterion covers a Write or Edit whose tool result failed (D5, D6), the pre-fix head's write, overwrite and clear, or the post-accept way forward on absent files.
**Fix:** Add one criterion for each.
