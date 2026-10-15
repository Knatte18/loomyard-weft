---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:114
    summary: Stops by settler never says which settle-record files the report reads or how a record's stop key is matched to a stop as convergence lists it, so items 2 and 3 have no join rule
    fix: In the Stops by settler Decision, after the sentence that moves the settle record type to shedrun, add - The report reads two settle-record files at the tag, the loom run's and batten's, each under its own run directory. A record names a stop when its stop key's producer and state equal the stop's and its history length equals the count of entries, in the history of the status that record's driver drives (the loom history for the loom file, batten's for batten's), stamped at or before the stop's change; a record matching no stop is a finding. Add an acceptance criterion - A loom-file go-rule restep record whose key matches a batten child change reads shed, and a batten-file record names only a batten-row stop.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:96
    summary: Sleep stays inside phase and bucket time, so an overnight hibernate inflates phase medians and day totals, against the Notes claim that tokencount's interval rule is the one used, since tokencount leaves sleep out of every row
    fix: In the Sleep Decision, replace the sentence beginning Sleep stays inside its phase's time with - Sleep, clipped to the run's span, is left out of every row interval, and so out of phase, wait and attempt time; it is reported per run and per bucket as its own figure, clipped to the bucket as other intervals are. In Waits, keep active time as the span minus sleep minus waits. In Output, name sleep as a column of the run table and the bucket table. In Acceptance criteria, change the first bullet to - For a landed run holding both histories, its phase seconds, other included, plus its sleep sum to its span, and active plus waits plus sleep equals the span. In Notes for the plan writer, leave the tokencount bullet as written, since it then holds.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:115
    summary: Stops by settler classifies every go-rule action but leaves the tackler, arbiter and parent records unclassified, and item 3 names no settler and action
    fix: In the Stops by settler Decision, change item 3 to - tackler - a settle record with settler tackler and action restep names the stop; and after the go-rule sentence add - A spawn record of any settler, a parent hand-back record and every arbiter record settle nothing by themselves; an arbiter settle is read from its applied attempt in item 4.
---

### [MEDIUM] No join from settle records to stops

**Location:** `_lyx/discussion/decision-record.md:114`

**Issue:** Item 2 counts a stop as `shed` when a `go-rule` `restep` or `verify-retry` record "names the stop", and item 3 reads a tackler resume from "a settle record", but the Decision never says how a record names a stop.
A settle record's key is `sheddriver.StopKey` (`internal/sheddriver/stopkey.go`): the current producer, the history length and the state of the status its driver drives.
The stops the report counts are convergence's, built from `battenshed.StateChange` (`internal/battenshed/classify.go:37`), which carries a producer, a state and a time and no history length.
There are also two files: batten's Go driver appends to `shedrun.SettleRecords(location, slug)` keyed on batten's status (`internal/battencli/drive.go:167`), and the loom driver to `shedrun.SettleRecords(location, c.runID)` keyed on the loom status (`internal/loomcli/drivewiring.go:142`).
Without a stated join, two plan writers would match differently, and a match on producer and state alone conflates repeated stops at one row, since a re-stepped interrupt at `Webster` gives one record per history length.

### [MEDIUM] Sleep counted inside phase and bucket time

**Location:** `_lyx/discussion/decision-record.md:96`

**Issue:** The Sleep Decision keeps sleep inside its phase's time and subtracts it only from active, wait and attempt time, and the Output names no per-bucket sleep figure.
A run that hibernates overnight in `webster` therefore reads ten hours of `webster` in its row, in the phase median and in that day's bucket, which is the figure the operator reads to see where time went.
The Notes for the plan writer say `tools/tokencount/shed.go`'s interval rule is the one this report uses, but that rule leaves sleep out of every row (`ProducerRow`'s doc, "machine sleep left out", and `Producers`, "An interval's elapsed and idle time leave out the machine's sleeps"), so the note and the Decision disagree.
On the focus question: under the Decision as written the identities hold when all four overlap, since waits are net of sleep, attempts lie inside waits and active is the remainder, provided sleep is clipped to the span, which the Decision does not state either; the fix states it.
Round 1's F3 put sleep inside phase time to make phase seconds sum to the span; the fix keeps the two consistent the other way, by moving sleep out of rows and rewording that criterion to phase seconds plus sleep, so it reopens no contradiction.

### [LOW] Non-go-rule settle records left unclassified

**Location:** `_lyx/discussion/decision-record.md:115`

**Issue:** `internal/sheddriver/drive.go` writes, besides the `go-rule` records the Decision classifies, `tackler`/`spawn` (line 452), `tackler`/`restep` (494), `arbiter`/`spawn` (540), `arbiter`/`restep` (552) and `parent`/`hand-back` (866).
The Decision names none of these pairs, so "a settle record shows an applied tackler resume" leaves the plan writer to infer that `tackler`/`restep` is the one, and whether an `arbiter`/`restep` record settles under item 4 or only the applied attempt does.

## Rejected

- Focus 1, reachability: with the `go-rule` `moved`, `busy-exit`, `running-exit` and `hand-over` records settling nothing, every later item stays reachable; a `moved` record followed by an orch goto reaches item 6, which the acceptance criteria pin. The Go driver's own steps and gotos carry `shed`, which coincides with item 2's class, so no stop is misread.
- Focus 2, imports: the Constraints and the Decision name the same set, `internal/durfmt` added and the settle format moved into the already-allowed `internal/shedrun`; live pairs and sleep intervals arrive told, and the `by` label is read off `shedengine` history, already allowed. Moving the format into `shedrun` makes `shedrun` its owner, so the package doc's tolerant-parse-in-the-owner rule holds. Not a defect.
- Focus 4, seconds: `statuspane.FormatSeconds` renders below a day with seconds and from a day on through `FormatMinutes` as `%dd%02dh`, so a week bucket or a long span prints as `7d00h` or `15d04h`, as the Decision and its criterion say. Not a defect.
