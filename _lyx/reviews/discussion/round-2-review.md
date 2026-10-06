---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:41
    summary: D1 hands the git-hook question to D4's hand check, but D4 states no such question
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:65
    summary: D3 leaves open what a run writes when a package fails to list or build, and where a newly measured package section goes
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:46
    summary: D2 does not say what counts as an empty reason or which functions count as a test function
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:146
    summary: Acceptance criteria lack scenarios for behaviors D2 and D3 state
---

### [MEDIUM] D1 delegates the hook question to a D4 that does not ask it

**Location:** `_lyx/discussion/decision-record.md:41`, `_lyx/discussion/decision-record.md:70`
**Issue:** D1's Bound says the scan cannot see a hook, alias or helper that makes git run module code, and that "D4's hand check therefore also asks whether the candidate makes git run module code".
D4 states only "a read of its assertions finds nothing its covering tests do not also assert" and never mentions the question.
The plan writer reads D4 for the deletion procedure, so the guard that bounds D1 would be lost.
The case is real in these packages: `internal/fabricengine/hook.go` installs a git hook into the repo's hooks directory.
**Fix:** State the question in D4: a candidate that makes git run module code, through a hook, alias or helper it installs, is kept under D2.

### [MEDIUM] D3 leaves the failure write and the new-section position open

**Location:** `_lyx/discussion/decision-record.md:65`
**Issue:** For a tag mismatch D3 says the tool writes nothing.
For a package that fails to list or build it says the old section is kept and the run exits non-zero, but not whether the other measured sections are still written.
A narrowed run that measures a package with no section in the existing report also has no stated position for the new section, and the report's section order decides whether two parallel regenerations merge cleanly.
**Fix:** State that a run with any failed package still writes the sections it measured, keeps the failed package's old section, and exits non-zero.
State that a new section goes in the report's existing package order (sorted by import path).

### [LOW] D2 leaves "empty reason" and "test function" undefined

**Location:** `_lyx/discussion/decision-record.md:46`
**Issue:** A whitespace-only reason is neither named as empty nor as valid.
"A `func Test…` line" excludes `Benchmark`, `Fuzz` and helper functions, but the refusal for "no test function directly below" does not say so.
**Fix:** Say that a reason with only whitespace is empty, and that the line below must start a top-level `func Test`.

### [LOW] Acceptance criteria miss stated behaviors

**Location:** `_lyx/discussion/decision-record.md:146`
**Issue:** No scenario covers a directive on a non-candidate test listed under "Kept", a kept test staying in the covering pool, or the non-zero exit when a package fails to list or build.
The failure bullet says "fails to build" where D3 says "fails to list or build".
**Fix:** Add the three scenarios and align the wording.
