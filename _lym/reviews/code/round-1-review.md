---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/landing/summary.md#Merge danger
    summary: The Merge danger section calls the door two-way, but the diff renames a state.json key (masterNotePath to masterNotePaths) that a binary on either side of the change reads, and the blast radius does not name it
    fix: "In `_lym/landing/summary.md` under `## Merge danger`, set `**Door:** one-way` and widen `**Blast radius:**` to name the state.json key rename. A run whose state.json was written before the change and is resumed after it loses its earlier note path and raises the parent-write finding once. A state.json written after the change and read by an older binary has no `masterNotePath`, so that binary flags even the current Master's note. Either case is cleared through `lym webster accept-audit`."
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-scope
    location: internal/websterengine/state.go#LoadState
    summary: A run whose state.json predates the change, which is the in-flight resumed case the board entry reports, still has its earlier note flagged, because the old masterNotePath key is never mapped into MasterNotePaths
    fix: "In `internal/websterengine/state.go`, add a `State` field tagged `json:\"masterNotePath,omitempty\"` and document it as the single note path a state written before `MasterNotePaths` existed holds. In `LoadState`, when that field is non-empty and `MasterNotePaths` does not contain it, prepend it to `MasterNotePaths`, then clear the field so the next save drops the key. Add a `TestRun_DoneOutcome` row that seeds state.json with the legacy key on the earlier note and asserts the run ends done with `MasterNotePaths` equal to `[earlier, latest]`. Rewrite the summary's Design decisions bullet on the old key to match."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/websterengine/audit.go:56
    summary: The ClassParentWrite comment still describes the allowance as one note, the friction note its prompt names, though CheckParent now allows every note a Master prompt of the run named
    fix: "In `internal/websterengine/audit.go` lines 56-57, change `its friction note` to `the friction notes the run's Master prompts named`, and change `the friction note its prompt names` to `any friction note a Master prompt of the run named`."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/websterengine/state.go:99
    summary: "The MasterNotePaths doc says the list is empty when friction notes are off, but a spawn with notes off keeps the paths earlier spawns recorded"
    fix: "In `internal/websterengine/state.go` line 99, replace `Empty when friction notes are off, and for a state written before the field existed.` with `Empty when friction notes were off at every Master spawn, and for a state written before the field existed.`"
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/websterengine/runlevel_test.go:2039
    summary: "The two guards on the spawn-time append at runlevel.go:810 have no covering row: skipping a path already listed, and skipping an empty path when friction notes are off while keeping the earlier ones"
    fix: "Add two `TestRun_DoneOutcome` rows next to the resumed-note row. In the first, the seeded state lists `<FrictionDir>/webster-master.md` and no file exists at that path, and the row asserts `MasterNotePaths` equals that single path. In the second, the seeded state lists one earlier note, `prepare` sets `fx.Deps.FrictionDir` to `\"\"`, the audit writes that earlier note, and the row asserts the run ends done and `MasterNotePaths` still equals the one earlier path."
  - id: F6
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/websterengine/audit.go:291
    summary: The parent-write violation Detail still says Master may write its friction note, singular, so a finding on a resumed run does not say that earlier Masters' notes are allowed too
    fix: "In `internal/websterengine/audit.go` line 291, change `and its friction note;` in the Detail format string to `and the friction notes the run's Master prompts named;`."
---

## Ground truths

The board entry asks that the parent audit treat every friction note path a Master of this run was given as Master's own.
The diff builds this for every state written by the new binary.
`Run` appends each spawn's note path and skips duplicates and empty paths (`runlevel.go:810-812`).
`CheckParent` allows a write to any listed path (`audit.go:283`).
Both callers, the run-exit audit (`runlevel.go:1093`) and record-batch (`recordbatch.go:413`), pass the list.
accept-audit does not call `CheckParent`, and it loads and saves `State` whole, so the list survives it.
`--fresh` and a missing state build a new `State`, so the list starts empty for a new run, which matches "this run".
The `Root-Cause:` trailer on 39d30e171 names the defect the diff removes, the overwrite of `State.MasterNotePath` at each spawn.
No PATTERN change is declared, and no `Pattern-Departure:` or `Loosened-Check:` trailer is needed.
PATTERN-tolerant-records holds: no decode refuses an unknown key, and `internal/state` keeps the stale `masterNotePath` key on rewrite.

### [MEDIUM] Merge danger calls a state.json key rename two-way

**Location:** `_lym/landing/summary.md`, `## Merge danger`
**Issue:** The rubric makes a `two-way` call a finding when the diff changes a record format another binary reads.
state.json's `masterNotePath` key becomes `masterNotePaths`.
A newer binary ignores the old key, and an older binary ignores the new one, so a run that crosses the binary change re-raises the parent-write finding, in the second direction on Master's current note as well.
The summary's Design decisions section describes the first direction, but the door and blast radius do not.

### [LOW] Pre-change in-flight runs still hit the reported bug once

**Location:** `internal/websterengine/state.go#LoadState`
**Issue:** The run reported in the board entry is a resumed run whose state.json holds `masterNotePath`.
After this change, that run's earlier note is dropped from the allowance and is flagged again.
The summary records the omission as a choice and points to the accept route, which exists (`AcceptPendingAudit`, `auditledger.go:108`).
Mapping the legacy key in `LoadState` would close the case with one small change.

### [LOW] ClassParentWrite comment still names one note

**Location:** `internal/websterengine/audit.go:56`
**Issue:** The comment reads "the friction note its prompt names", while the file header and `CheckParent`'s doc now describe every note the run's Master prompts named.

### [LOW] MasterNotePaths doc overstates when it is empty

**Location:** `internal/websterengine/state.go:99`
**Issue:** A resumed spawn with friction notes off skips the append, and the list keeps the earlier paths, so "Empty when friction notes are off" is false for that state.

### [LOW] Append guards are untested

**Location:** `internal/websterengine/runlevel_test.go:2039`
**Issue:** No row reaches the `slices.Contains` skip, which happens when an earlier Master never wrote its note and `friction.NotePath` returns the same path (`friction.go:146-148`).
No row reaches the empty-path skip either.
Dropping either guard leaves every current test green.

### [NIT] Violation Detail says "its friction note"

**Location:** `internal/websterengine/audit.go:291`
**Issue:** The operator-facing Detail names a single note, which is imprecise for a resumed run where earlier notes are allowed too.

## Rejected

- Failure paths around the spawn record a gap: the note path is saved in the same `SaveState` as `MasterStrand`, so a failed save loses both together, and a failed `StartMaster` returns before any Master ran on that prompt. This is not a defect.
- A resumed run with friction notes off still allows the earlier notes: this follows the board entry's "every friction note path a Master of this run was given". This is not a defect.
- The shared `resumedNoteState` pointer in `TestRun_DoneOutcome` is set in `prepare`, before `seedMatchingState` reads it (`runlevel_test.go:2272-2275`), and only one row uses it. The row fails without the fix.
- The new `TestCheckParent` rows duplicate nothing: no earlier row asserted that an empty list refuses a note write, or that a write to a non-last listed note is allowed.
- The master stencil's friction-note lines stay accurate, since each prompt names one note.
