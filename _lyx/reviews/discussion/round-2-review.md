---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:122
    summary: Open risks names the Plan-Review rubric as the backstop for an unmarked signature change, but D6 adds no such rubric finding and the current rubric has none
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:57
    summary: D3 and two acceptance criteria rest on a begin-batch canonicalization that keeps a handle a handle; the transition to a member glyph is the record-batch bind
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:74
    summary: D5 does not say which tier runs the arrow's head-to-member match, whether it runs at dispatch, or what the arrow does on a plan handle Edit target
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:7
    summary: The Goal states a universal no-verdict-change rule that ValidateDispatch's existing delete-target-gone downgrade contradicts
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:65
    summary: D4 removes the stencil sentence with the check, but D6 and D7 put every stencil edit in the last card
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:40
    summary: Restatements and process facts the plan writer cannot act on; relocate or delete them after extracting nothing, since each requirement is already stated elsewhere
---

### [MEDIUM] Open risks promises a rubric backstop no Decision builds

**Location:** `_lyx/discussion/decision-record.md:122` (Open risks), against D6 at line 91.
**Issue:** Open risks says "Planners may skip the re-sign arrow; the Plan-Review rubric is the only backstop for an unmarked signature change."
D6 gives the rubric exactly one new finding: an `Edit` card holding only Go file paths of non-test files while its Intent names specific symbols.
That finding does not catch an `Edit` card that lists a member glyph, changes its signature, and omits the arrow.
`contracts/stencils/loom/loom-rubric-plan-review.md` has no signature or caller finding today (grep for signature, caller, re-sign finds only an unrelated blast-radius line).
So the record names a mitigation that no Decision creates, and the plan writer will not write one.
The risk is then unmitigated and mislabelled as mitigated.
**Fix:** Either add to D6 a rubric finding for an `Edit` card whose Intent states a signature or return-type change on a member that carries no arrow, or reword the risk to say no mechanical or rubric backstop exists.
Add a matching acceptance criterion if the rubric finding is added.

### [MEDIUM] D3's stability claim and its criteria name the wrong transition

**Location:** `_lyx/discussion/decision-record.md:57`, `:127`, `:130`.
**Issue:** D3 says the pure pairings give the same verdict "before and after `begin-batch` canonicalizes the handle", and D3's bound says a pairing "only appears after `begin-batch` canonicalizes a handle into a glyph".
In `internal/planglyph/handle.go` `CanonicalizeHandles` rewrites a draft handle to `plan:<expected-glyph>`, which `planparser.IsHandleRef` still reports as a handle.
It runs inside `resolvePass` at every gate, the plan gates included, so the plan gate has already canonicalized the handle before any later begin-batch.
A handle becomes a plain member glyph only when `BindHandles` runs at record-batch, after the declaring card's batch.
Consequences:

- The criterion at line 130 ("before and after the handle is canonicalized") compares two states that are both handles, so it passes trivially and leaves the real change (handle to bound glyph) unpinned.
- The criterion at line 127 and the file-versus-member bound at line 59 describe the same event with the wrong mechanism, so a test written to them exercises `CanonicalizeHandles` and never `BindHandles`.
- For a `Rename` to-side handle, canonicalization replaces the draft unit with the old side's resolved unit, so "counts as a member of its declared unit" is not stable across canonicalization for a draft that misspells the unit.
`handle-malformed` pins a `Create` handle's unit to a package directory, so that case is stable.

**Fix:** Say that the handle becomes a member glyph when record-batch binds it.
Restate the line 130 criterion as: package self glyph plus a handle of that package, and the same card after the handle is bound to its glyph, both give the redundancy finding from `ValidateDispatch`; file plus handle gives none at the plan gate, and file plus the bound glyph gives none from `ValidateDispatch`.
State which unit a `Rename` to-side handle is counted under (the resolved old side's), or exclude `Rename` to-side handles from the pure pairing.

### [LOW] D5 leaves the arrow check's placement and handle target open

**Location:** `_lyx/discussion/decision-record.md:74-77`.
**Issue:** D5 says the head "must name the same member as the glyph" and that `PATTERN-sole-parsers` confines arrow parsing to `planparser`.
Deriving a member from a declaration head is what `quarry.Name` does, and in the current code only `planglyph`'s `CanonicalizeHandles` calls it.
The record does not say whether the match is a pure `planparser` check, a resolve-backed `planglyph` check, or where it sits relative to the dispatch split D3 draws.
The Goal makes the tier of every new check a stated matter, and this check is tree-independent, so it may run at dispatch, but the record does not decide it.
It also names file paths and self glyphs as the non-member targets that make the arrow blocking, and is silent on a `plan:` handle `Edit` target, which is neither.
**Fix:** State that `planparser` parses the arrow into glyph and head and the member match runs in the named tier, whether `ValidateDispatch` runs it, and that an arrow on any target that is not a member glyph, a handle included, is the blocking finding (or name what a handle target does).

### [LOW] The Goal's no-verdict-change sentence over-claims

**Location:** `_lyx/discussion/decision-record.md:7`.
**Issue:** "No check gives a different verdict at dispatch than it gave at the plan gate because of state the run itself changes" is universal.
`ValidateDispatch`'s existing `downgradeGoneDeleteTargets` exists to give a different verdict at dispatch (blocking `path-missing` or `glyph-not-found` becomes informational `delete-target-gone`) because a run landed earlier cards.
A plan writer reading the Goal literally would audit every existing check, which is outside the Scope.
**Fix:** Scope the sentence to the checks this task adds or changes.

### [LOW] D4 and D6/D7 disagree on which card edits the stencil

**Location:** `_lyx/discussion/decision-record.md:65`, `:90-91`, `:96`.
**Issue:** D4 says the check is removed "with its spec row and the stencil sentence requiring every handle to be used", and D4's card comes before D6's.
D6 says the stencil "drops the 'used by at least one later card' rule (D4)", and D7 puts D6's stencil last.
The plan writer has two cards that each claim the stencil sentence, or a window in which the stencil demands a rule the validator no longer enforces.
**Fix:** Take the stencil sentence out of D4's list and leave it to D6, or say D4's card removes it and D6 does not.

### [LOW] Restatements and non-actionable process facts

**Location:** `_lyx/discussion/decision-record.md:40`, `:50`, `:97`, `:136`, `:139`.
**Issue:** Applying the relocation test; each requirement below is already stated elsewhere, so nothing needs extracting first.

- Line 40: the failed-card recovery path (`rebaseline --card NN`, then `recover-batch`) is an execution-time operator fact the plan writer cannot act on, and Q13 in the support log already holds it.
  The part the plan writer acts on, one fix card per probed defect, is stated at line 39.
- Line 50: the `webster-parallel-execution` paragraph is deliberation about a parked note, already named under Scope Out, and states no requirement.
- Line 97: D7's second sentence restates D1's lines 36-37 (rows pass on the chain as it stands; later cards add or flip rows).
- Lines 136 and 139: "every card leaves `go test ./...` and `go test -tags integration ./...` green" and the final suite criterion overlap, and D1 line 36 states the per-card rule a third time.

**Fix:** Move the line 40 recovery sentence and the line 50 paragraph to the support log, keep the per-card green rule once (D1), cut D7's duplicate sentence down to the flip assignments, and keep one of the two green-suite criteria.

## Focus departures

None.
Each focus entry was reached: dispatch split (F2, F3, F4, and the D3 and D5 text is otherwise consistent with `planglyph.go`), test-row timing (D1 and D7 settle it; the D4 flip is implied, not named, and not raised as a finding), defect-to-card path (D1 settles it; `rebaseline --card` exists in `internal/websterengine/doc.go`), caller-coverage file set and under-reporting bound (stated and consistent with `deleteorder.go`; not a defect), re-sign arrow (F3), and relocation (F6).
