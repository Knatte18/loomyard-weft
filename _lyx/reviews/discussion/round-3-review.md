---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:127
    summary: "The settle-record match rule cannot be computed from what convergence holds for a stop: the stop's change time is batten's observation time, a note-only stop has no change, and the producer may be empty"
    fix: "In the `Stops by settler` Decision, replace the sentence at line 127 with a rule built on data a stop carries. State that the history length compared is the count of entries of the driven history stamped strictly before the next entry of either history after the stop's change, or give another rule that survives batten observing the change after the driver has already restepped. State that a stop without a change, a note-only stop, is matched on producer and state alone against the records of its run's loom file and that its settler is read from the later items when no record names it. State that an empty producer on a change matches no record by producer and falls to the later items. Add one Acceptance criteria line for a restepped stop whose batten observation lags the loom's next entry."
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:111
    summary: "Batten's settle file is read, but a stop is defined only as a convergence stop, which holds no batten-row stop, so every batten-file record matches no stop and adds a finding"
    fix: "In the `Stops by settler` Decision, say what a batten-file record can settle. Either define a batten-row halt, a halted or failed batten history row, as a stop the report counts apart from `convergence`'s halt count with its own settler, or state that the batten file is read only to find the `shed`-class records of the batten driver for the child's stops and that a batten-file record naming a batten row is not a finding. Align the Acceptance criteria line at 202 to whichever is chosen, and say in the line at 128 that a record naming a batten row is exempt from the matches-no-stop finding if the second option is taken."
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:54
    summary: "Each interval runs from the previous merged stamp, so an out-of-order stamp makes the next interval re-cover time and breaks the sum-to-span acceptance"
    fix: "In the `Run span and intervals` Decision, change each row's interval to run from the latest merged stamp seen so far, not the previous one, which keeps an out-of-order stamp at a zero interval and stops the next row from re-covering time. Adjust the criteria line at 191 to say a stamp earlier than the latest stamp so far gives a zero interval and a finding, and the next interval starts from that latest stamp."
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:211
    summary: "The Notes say the tokencount interval rule is the one this report uses, but the Decisions differ in the history merged, the first row, the start of the span and the latest-stamp rule"
    fix: "Reword the first Notes bullet to say `tools/tokencount/shed.go` splits a run's span by one shed history with sleep left out of each interval and is the model for the per-row split, and that this report merges the loom and batten histories and opens the span at batten's first row as the Run span and intervals Decision states. Also drop 'interval rule' from the Rationale at line 56, which cites the 2026-10-12 computation's rule while stating where the report departs from it."
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:126
    summary: "The settle-record files are read at the tag, but a run in flight has no archive tag, so where its stops get records is left open"
    fix: "In the `Stops by settler` Decision, change 'at the tag' at line 126 to 'at the tag, or at the records branch tip for a run in flight', and state that a record not yet committed at that tip leaves its stop `open` rather than adding a finding."
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:119
    summary: "Item 6 lists the labels orch, driver and operator, but the Go driver and batten write the label `shed`, so which class a `shed`-labelled move reads as is left to the plan writer"
    fix: "In the `Stops by settler` list, change item 6 to say the `by` label `shed` reads as class `shed` and every other label reads as its own value, so a move by the Go driver or batten that no settle record covers still reads `shed`, not an unlisted role."
---

### [MEDIUM] Settle-record match rule is not computable from a stop's data

**Location:** `_lyx/discussion/decision-record.md:127`
**Issue:** The Decision matches a record to a stop when the record's history length equals the count of entries stamped at or before "the stop's change" in the driven history.
A stop is built from `battenshed.StateChange`, whose `At` is the time of batten's history entry for the change, which batten writes when it observes the child's state, later than the child's own halt.
A restep the Go driver takes between the halt and that observation adds loom entries stamped before `At`, so the count overshoots the record's `len(st.History)` and the record never matches.
Convergence also lists stops with no change (`Stop.Change` is false for a note left unmatched in `internal/reportengine/stops.go`), which have no time to count against.
`StateChange.Producer` is empty when the recorded text names none, and the rule compares producers by equality.
The Decision's own wording, "history length equals the count of entries stamped at or before the stop's change", leaves a plan writer no way to get the match right for these three cases.

### [MEDIUM] A batten-file record matches no stop under the stop definition

**Location:** `_lyx/discussion/decision-record.md:111`
**Issue:** The Decision defines a stop as a halt or awaiting stop as `convergence` lists it, and `convergence` builds stops from the child's state changes batten recorded and from halt notes.
The Decision then reads a second settle file, batten's, which `internal/battencli/drive.go` writes for batten's own status, so its records name batten rows.
The Acceptance criteria line at 202 says "a batten-file record names only a batten-row stop", but no batten-row stop is in the stop list the report builds.
Under line 128, each such record "matching no stop is a finding", so every batten-driven stop adds a finding to the report.
The plan writer has to pick what a batten-file record settles, and the Decision does not.

### [MEDIUM] "Previous merged stamp" re-covers time after an out-of-order stamp

**Location:** `_lyx/discussion/decision-record.md:54`
**Issue:** The Decision gives a stamp earlier than the previous one a zero interval and then starts the next row's interval from the previous stamp, which is that earlier stamp.
The next interval then covers time the earlier rows already own, so phase seconds sum past the span and the first acceptance criterion (line 188) fails on exactly the records line 191 is written for.
`tools/tokencount/shed.go` starts each interval at the latest end among the entries before it, which avoids this.

### [LOW] The Notes' claim that tokencount's interval rule is the one used does not hold

**Location:** `_lyx/discussion/decision-record.md:211`
**Issue:** `tools/tokencount/shed.go` splits one shed history, starts the first entry's interval at the span start raised to the older tag's date, uses the latest earlier end, and takes sleep out of each interval.
The report merges two histories, opens at batten's first row and, as written, starts from the previous stamp.
The Rationale at line 56 already says the span opening differs, so the Notes contradict the Decision they head.
A plan writer who reuses `Producers()` for the report gets the latest-stamp rule and the shed-only split, not the merged one.
The sleep part of the claim holds: both leave machine sleep out of every row, as the Sleep, Waits, Output and Acceptance criteria Decisions state alike.

### [LOW] Settle records "at the tag" for a run in flight

**Location:** `_lyx/discussion/decision-record.md:126`
**Issue:** A run in flight has a records branch and no archive tag, yet it has stops and settle records, and item 8 (`open`) exists for it.
The Decision reads the files "at the tag" and does not say where a run in flight's records come from.

### [LOW] Item 6 does not map the `shed` label

**Location:** `_lyx/discussion/decision-record.md:119`
**Issue:** Item 6 names `orch`, `driver`, `operator` or another role.
The caller-label Decision has the Go driver and batten pass `shed`, which is also a settler class.
Whether `shed` falls under "another role" or reads as the `shed` class depends on the plan writer.

## Rejected

- Classification coverage of the Go driver's settle record pairs: every `go-rule`, `tackler`, `arbiter` and `parent` pair `internal/sheddriver/drive.go` writes has a stated class or an explicit settles-nothing line (items 2 to 4 and the two paragraphs after the list), so two plan writers classify one stop alike; not a defect.
- Whether the Sleep, Waits, Output and Acceptance criteria Decisions agree on sleep: they agree that sleep lies outside every row interval, phase, wait, attempt and bucket figure and is reported as its own figure; not a defect.
- A missing list of the functions and signatures the report calls, such as `shedrun.SettleRecords(location, slug)` against `(location, runID)`: the writer is told not to gather exact signatures, so absence is by design.
