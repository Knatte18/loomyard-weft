---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:95
    summary: D5's status-unreadable way forward waits for the file to read again, while D5's own No re-step bullet says a file that fails to decode stays that way, so at a decode error the way forward names no step that reaches anything
    fix: In D5's way-forward bullet (decision-record.md:95), split status-unreadable by its two variants. For a read-lock or raw read error that failed twice, keep re-running `lyx shed step <run> --until-stop`. For a decode error, replace "once the file reads" with the condition under which it reads, re-stepping with a binary that decodes the file as after a version skew, and otherwise reporting the stop to the run's parent with the status path and loop log, as refusal-spec's `arbiter - status or envelope unreadable` row does. Reword the No re-step bullet (line 96) so it says the file stays undecodable to the binary that failed, not for good. Carry the same split into the refusal-spec bullet at decision-record.md:97 and into the D5 acceptance row.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:97
    summary: D5 extends refusal-spec's shed step interrupted row, classed transient, with causes the spec's own class table makes correctness halts, and states no class for them
    fix: In D5's refusal-spec bullet (decision-record.md:97), state each new cause's class. status-missing and a decode-error status-unreadable are correctness halts, matching the shed `status file missing` row, because neither clears by itself or by a retry. A read error that fails twice stays transient. Where the step interrupted row's single class cannot carry both, the bullet names a separate shed row for the correctness-halt causes.
  - id: F3
    severity: NIT
    class: scope
    location: _lyx/discussion/decision-record.md:150
    summary: D3 counts a Rename destination as a created directory, but no acceptance criterion pins that case
    fix: In the verify-package-missing acceptance row (decision-record.md:150), add to the reports-nothing list a Rename card whose pair's New side lies in a new directory, followed by a Verify on that directory.
---

## Focus verdicts

D3's created-directory set: not a defect.
D3 reads `CardTargetDirs`, not `createTargetsUnion` or `diskPathForRef`.
`refUnit` in `internal/planparser/targetdirs.go` maps every glyph through `UnitPath`, member glyphs and `plan:` handles included, and `CardTargetDirs` walks every `TargetGroup` whatever its type, so Create, Edit, Prosa and Custom groups all count, along with a Rename's New side.
Move pairs are the one gap, and D3 bounds them by skipping any Verify on or after a Move card.
The ancestor rule covers a Verify on a parent of a created file's directory.
No valid plan outside the stated bounds draws a false positive from what I read.
The acceptance row pins member-glyph Create and Custom groups but not Rename (F3).

D5 and refusal-spec: Scope In, D5's last bullet and the Constraints line each name the shed `step interrupted` row.
D5's bullet also states that the status file is left as found.
The row's class is not settled (F2).

D5's way forward and Bound: the status-missing way forward names the recipe's reseed, matching refusal-spec's `status file missing` row, so a re-step that cannot reach anything is not offered there.
The Bound says a later-`running` file is resumed by the re-step, not `lyx loom resume` or `lyx loom arbiter`, which settles the question.
The decode-error half of status-unreadable has no reachable way forward (F1).

D5's coverage of transient causes: settled.
`state.ReadJSONNoMkdir` wraps a lock failure as `acquire read lock: …`, a non-ENOENT read failure with `ErrRead` and a decode failure with `ErrDecode`, so the split D5 states can be implemented.
D5 covers read-lock and raw read errors with one re-read under the existing re-step, and decode errors as a stop.

Acceptance criteria: D2's criterion fails if the PATTERN.md line or its background lacks the mapping.
D7's fails if the cobra bullet, the overview pointer or the rubric check is missing.
Each criterion tests behavior a Decision states.
D6 has none, which fits a no-change Decision.

### [MEDIUM] D5's decode-error way forward waits on a file that never heals

**Location:** `_lyx/discussion/decision-record.md:95`
**Issue:** The `status-unreadable` way forward is to re-run `lyx shed step <run> --until-stop` "once the file reads".
The next bullet (line 96) rules out a re-step on a decode error because `internal/state` writes atomically, "so a file that fails to decode stays that way".
Read together, the two bullets say that for a decode error the condition the way forward waits on never comes about.
The one route by which it does is version skew, the round-1 framing: a loop on an older binary fails to decode a file that a current binary reads.
The record states neither that route nor what to do when the file is corrupt rather than skewed.
`PATTERN-refusal-way-forward` needs a way forward that an agent can act on.
The read-error variant, which re-reads once and then stops, can clear, so a re-step fits it.

### [LOW] New loop-stop causes keep the step interrupted row's transient class

**Location:** `_lyx/discussion/decision-record.md:97`
**Issue:** refusal-spec's class table defines transient as "a condition that clears by itself or by a named action", and the shed `step interrupted` row is transient.
D5 adds `status-missing`, whose way forward is a reseed, matching the `status file missing` row's correctness-halt class.
It also adds a decode-error `status-unreadable`, which D5 itself says does not clear.
D5 states no class for either, and the Constraints line notes that `convergence-plan` classes halt reasons by their leading clause.

### [NIT] No acceptance criterion for a Rename destination as a created directory

**Location:** `_lyx/discussion/decision-record.md:150`
**Issue:** D3 says a Rename destination counts as a created directory.
The acceptance row's reports-nothing list covers path targets, a member-glyph Create and a Custom group, but has no Rename case.
