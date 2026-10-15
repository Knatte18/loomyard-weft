---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md, D4, third rule (the phase bullet)
    summary: D4 says the tokens stay the last reading until the next turn end during a cycle, but the watcher re-takes the reading at a compaction boundary with no turn end and clears it to unknown on a resume timeout
    fix: In D4's phase bullet, replace the clause "during a cycle the tokens stay the last reading until the next turn end, and the phase cell says so" with "the tokens are the persisted reading as it is, which during a non-idle phase may predate the cycle; the phase cell is what tells the operator a cycle is in progress". Add no new rule; the line keeps drawing the persisted fields only.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md, D2 and the sixth acceptance criterion
    summary: D2 names an absent file (not found) and a failed read, but never says which outcome a present file that fails to decode gets, and its rationale only covers a stat or read failure during the rename
    fix: In D2's Decision paragraph, after "an absent file reads as not found", add the sentence "A present file that fails to decode is a failed read, counted by the same tolerance as a failed stat or read." In the acceptance criterion beginning "One failed state read after a good one", change "One failed state read" to "One failed state read, a decode failure of a present file included,".
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md, D3, Bound line
    summary: D3's bound says lyx orch start replaces the runs strand only on a build change or a dead strand, but it also replaces a live strand whose runs sidecar is absent or records another guid
    fix: In D3's Bound line, change "which `lyx orch start` does on a build change or a dead strand" to "which `lyx orch start` does when the strand is dead or its sidecar does not record it as launched by the running build".
---

### [LOW] D4's mid-cycle reading clause rests on a false premise

**Location:** `_lyx/discussion/decision-record.md`, D4, the phase bullet.
**Issue:** The clause claims that during a cycle the tokens stay the last reading until the next turn end.
`internal/orchengine/watcher.go`'s `tickCompacting` calls `storeCurrentReading` at a qualifying compaction boundary and at the compaction timeout, both without a turn end, and the resuming phase's timeout path sets `LastContextTokens, LastContextKnown = 0, false`.
The overview only draws persisted fields, so the code the plan writer builds is unaffected, but a test written from the clause would pin behavior the watcher does not have.
"The phase cell says so" also reads as if the phase cell carries extra text; D4's own rule is that the phase renders as it is.

### [LOW] A decode failure of a present state file has no stated outcome

**Location:** `_lyx/discussion/decision-record.md`, D2 and the acceptance criteria.
**Issue:** D2 assigns an absent file to "not found" and a failed read to the two-strike tolerance.
A present file that fails to decode is neither named, and the rationale ("covers a transient stat or read failure during the rename") suggests only stat and read count.
`statuspane.StatusCache`, which D2 names as the model, counts a decode failure as a failure, but the orch reader is a new `orchengine` function whose shape is the plan writer's to choose, and an existing sibling (`CycleRequested`) maps an unparsable file to a non-error result.
The plan writer can therefore build a reader that returns not found on a corrupt file, which would hide the orch line instead of drawing `orch  state unreadable`.

### [NIT] D3's bound under-states when the runs strand is replaced

**Location:** `_lyx/discussion/decision-record.md`, D3, Bound.
**Issue:** `ensureRunsStrand` in `internal/orchcli/runs.go` keeps a strand only when `statuspane.StrandCurrent` holds, which requires it live and the sidecar to record its guid and the running build.
An absent or mismatched sidecar also replaces a live strand of the same build.
The bound's direction stays right, since a threshold edit alone still does not show, so this is wording only.

## Focus directive verdicts

- D2 atomic writes: holds.
  Every write of `Paths.StatePath` goes through `internal/state` (`SaveState`, `saveStateForStrand`, `updateState`), whose `writeJSONUnlocked` ends in `fsx.AtomicWriteBytes`; no other production code writes the file.
  The two-strike tolerance is stated the same way in D2, the acceptance criterion and the Turn 3 ledger line; the criterion omits D2's "same failed modification time" clause, but that case is always a failure in a row, so the criterion subsumes it.
- D2 and D4 absent against unreadable: the decode case is unassigned (F2).
- D3 bound: consistent in direction, wording under-states the replacement cases (F3).
- D4 rules against criteria: every rule has a matching criterion except the phase-during-cycle clause, whose premise is false (F1) and which needs no criterion once restated, and the no-padding rule, which the criterion "the run lines follow as the parent renders them" together with the stated line form already pins.
  The stopped-session rule is backed: `lyx orch stop` removes the strand, and the watcher's `tick` records `WatcherExit = "strand gone"`.
- Decisions against Constraints: no contradiction.
  `PATTERN-config-strictness` governs which `configengine` loader a module adopts; D3 keeping `LoadConfig` and degrading its error at the call site does not change orch's policy.
  D3's Warn is a one-shot at verb start on a fallback path, which `internal/logger`'s level policy names as Warn; the per-frame Debug constraint matches the overview's existing per-frame failures in `overviewSource.frame` and `poll`.
- Support log against record: every Turn 3 "fixed" line is reflected in the record, and no requirement sits only in the support log; the rejected per-frame config load and per-frame lock probe are carried by D3's "once at start" and the criterion "takes no lock".
