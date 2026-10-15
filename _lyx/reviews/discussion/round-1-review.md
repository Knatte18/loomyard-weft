---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:114
    summary: Settler rule 2 matches any go-rule settle record, but the Go driver writes go-rule records for moved, busy-exit, running-exit and hand-over, so an orch-settled stop reads as shed and rules 3 to 6 are shadowed
    fix: In Stops by settler item 2, replace "a Go driver settle record shows a rule acted on it" with "a Go driver settle record with settler go-rule and action restep or verify-retry names the stop"; add one sentence that go-rule records with action moved, busy-exit, running-exit or hand-over settle nothing and leave the stop to the later items; add an acceptance criterion that a stop whose only go-rule record is moved, followed by a goto with by orch, reads orch.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:151
    summary: The Constraint says the reportengine allowlist gains internal/durfmt only, yet the settler Decision reads the Go driver settle records, whose type and tolerant decoder live in internal/sheddriver, which is not on the allowlist
    fix: Add a sentence to the Stops by settler Decision naming how reportengine decodes `go-settle.jsonl` (either the settle record type and a byte-level tolerant decoder move to a leaf the engine may import, or reportengine imports `internal/sheddriver`), and update the PATTERN-told-geometry Constraint to list that package beside `internal/durfmt`, so the Constraint and the Decision name the same allowlist growth.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:96
    summary: Sleep is subtracted from every interval it overlaps, which removes it from row and phase time, yet the first acceptance criterion requires phase seconds to sum to the span, which only holds if sleep stays inside phase time
    fix: In the Sleep Decision, replace "Sleep is subtracted from every interval it overlaps" with "Sleep stays inside its phase's time, as waits do, and is subtracted only from active time, wait time and attempt time"; keep the acceptance criterion that phase seconds sum to the span.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:138
    summary: The Seconds Decision promises every markdown duration with seconds through the status pane formatter, but FormatSeconds renders a day or more as FormatMinutes does (2d03h, no seconds), and week buckets and long spans routinely pass a day
    fix: In the Seconds Decision, state the day-and-over case explicitly, either accepting the pane format (durations of a day or more render as 2d03h, without seconds) and rewording the acceptance criterion to the status pane's format only, or giving durfmt a seconds form for a day and over that FormatSeconds does not change; pick one and make the acceptance criterion at line 197 match it.
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:44
    summary: The window rules leave two choices open, whether a partial run adds to the day and week buckets, and whether a run overlapping --since contributes buckets dated before --since
    fix: In Run selection and windows, add that a partial run adds to no bucket (or that it does, stating which), and that bucket totals clip intervals to the window as well as to the bucket so no bucket before --since is printed.
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:128
    summary: The Go driver resumes halted runs through Deps.Step as well as Deps.Goto, but only its goto is given the shed label, so a driver resume writes no by and its label is left undefined
    fix: In The caller label, extend the sentence on the Go driver to say it passes shed through both its Deps.Goto and Deps.Step seams, and extend the acceptance criterion at line 195 to say a Go driver resume also writes by shed.
  - id: F7
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:122
    summary: The Stops by settler rationale argues against a rejected alternative (folding tackler into shed or driver), which the compaction rules keep out of Decisions
    fix: Shorten the rationale to "the board's settlers, plus tackler, which settles stops without an LLM parent", and add a Rejected alternatives bullet in support-log.md for folding tackler into shed or driver with the existing reason.
---

The pair is close to plannable: the verb, run selection, span, phases and caller label are bounded and consistent with the code read.
The findings concern how the records distinguish settlers, one constraint the settler Decision outgrows, and two identities the acceptance criteria state that the Decisions do not deliver.

### [MEDIUM] Settler rule 2 shadows orch settles and the later rules

**Location:** `_lyx/discussion/decision-record.md:114`
**Issue:** `internal/sheddriver/drive.go` writes `SettlerGoRule` records for `SettleMoved` ("an orch goto moved the run: re-step", line 778), `SettleBusyExit` (241), `SettleRunningExit` (768) and `SettleHandOver` (345), none of which settle the stop.
Read literally, "a Go driver settle record shows a rule acted on it" holds for each of them, and since item 2 precedes items 3 to 6 in "the first that holds", a stop the orch moved on classifies as `shed`.
The records can distinguish the cases by `action`, but the Decision does not say so.

### [MEDIUM] Settle-record reading is outside the stated allowlist growth

**Location:** `_lyx/discussion/decision-record.md:151`
**Issue:** `SettleRecord` and `ReadSettleRecords` live in `internal/sheddriver/settle.go`, and `ReadSettleRecords` reads a file from disk while `reportengine` reads git objects.
The `reportengine` allowlist in `seam_enforcement_test.go` has no `sheddriver`, and the Constraint says it gains `internal/durfmt` only.
Its package doc says each record is read "through a tolerant parse in the package that owns its format", so the plan writer must either breach that or breach the Constraint.
The `durfmt` import itself is justified: `reportengine` owns `markdown.go`.

### [MEDIUM] Phase sum and sleep subtraction disagree

**Location:** `_lyx/discussion/decision-record.md:96`
**Issue:** Waits "stay inside their phase's time", but Sleep "is subtracted from every interval it overlaps", and row intervals are the intervals phases sum.
With sleep removed from rows, phase seconds sum to span minus sleep, contradicting the acceptance criterion at line 181.
The active/waits/sleep identity holds as written: sleep first, waits net of sleep and unioned, active the remainder, which partitions the span even when a wait, a sleep and an attempt all overlap.

### [MEDIUM] FormatSeconds drops seconds from one day on

**Location:** `_lyx/discussion/decision-record.md:138`
**Issue:** `internal/statuspane/pane.go:129` documents FormatSeconds as rendering "from one day on as FormatMinutes does", which prints `2d03h`.
A week bucket summing parallel run-time passes a day often, so "every duration in markdown renders with seconds" and the acceptance criterion at line 197 cannot both hold with the named formatter.

### [LOW] Partial runs and pre-window buckets left open

**Location:** `_lyx/discussion/decision-record.md:44`
**Issue:** A partial run is kept out of medians and the run table, but its bucket contribution is unstated.
A run whose span overlaps `--since` enters the report, and day totals clip only to the bucket, so whether buckets before `--since` appear is unstated.

### [LOW] The Go driver's resume gets no label

**Location:** `_lyx/discussion/decision-record.md:128`
**Issue:** `internal/sheddriver/drive.go:51-53` exposes both `Deps.Step` and `Deps.Goto`, and the driver's tackler and arbiter resumes may continue without a goto target.
The label is classified only in the CLI layer, so a resume through `Deps.Step` writes no `by`.
Settler items 2 to 4 still take precedence for such stops, so the effect is on the record, not the settler count.

### [NIT] Rejected alternative in the settler rationale

**Location:** `_lyx/discussion/decision-record.md:122`
**Issue:** "folding it into `shed` or `driver` would hide which agent carries the load" argues against an unchosen option; the discussion spec keeps rejected alternatives in the support log.

## Rejected

- In-flight definition and reused slug: consistent with each other and with the acceptance criteria; an in-flight records branch's untagged tip and the after-newest-tag snapshot rule match `convergence`'s fresh-seed reading.
- Caller label bound: the status field and the goto `by` field are report-only, tolerant and read by no Go decision; the `shed` label from the driver and batten adds no routing edge or guard change, and the Constraints name the `shedverbs`/`shedengine` import limits the Decision relies on.
- `sleepjournal` as a stdlib-only leaf: consistent, since only parsing moves and `reportcli` runs `journalctl`.
- Relocation of the 2026-10-12 references in Goal and the span rationale: short, and they tell the plan writer what the interval rule and the setup difference are, so they stay.
