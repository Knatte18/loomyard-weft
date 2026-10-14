---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:141
    summary: D9 dates the hand-on end by the verdict file, but the verb's own hand-ons (timeout, died, missing or unparseable verdict, failed resume precondition) leave no verdict.yaml or one that says resume; only envelope.json records every hand-on
    fix: In D9's end-condition bullet (decision-record.md line 135) say an attempt hands on when its saved envelope (`envelope.json`, written by the verb's `finish` for every outcome) carries verdict `hand-on`, and in line 141 replace "the hand-on attempt's verdict file time" with "the write time of that attempt's `envelope.json`"; make the same replacement in the restart acceptance criterion (line 195, "the hand-on verdict file's time") and in Q14 of support-log.md.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:135
    summary: The hand-on end condition and its time are not bounded to the episode, so a re-halt at the same stop key ends the new hold at once with an end time before the episode's since, and the notice goes earlier than today's rules allow
    fix: In D9's end-condition bullet (line 135) qualify the hand-on as "an attempt for the stop key whose envelope was written in this episode hands on", as the park marker already is; and in line 141 state that the hold's end time is never earlier than the episode's since.
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:140
    summary: The reason for measuring the report wait from the hold's end restates the decision instead of giving the cause
    fix: Replace "since the report wait must start no earlier than the hold's end" in line 140 with "since a hold outlasting `noticeReportWait` would otherwise find the wait already spent at its end and send the notice with `report none yet` before the driver's park marker".
  - id: F4
    severity: NIT
    class: design
    location: _lyx/discussion/decision-record.md:142
    summary: D9 says an applied resume leaves no record batten can read a time from, but the attempt's `applied` mark is a file with a write time, like the verdict file D9 does read
    fix: In line 142 either date an applied resume by its attempt's `applied` mark write time, removing it from the batten-first-observed case and from the restart criterion's own-start-time list (line 195) and Q14, or reword the premise to say the design chooses batten's first observation for it, without claiming no record exists.
---

### [MEDIUM] The hand-on end time names a file most verb-made hand-ons lack

**Location:** `_lyx/discussion/decision-record.md:141`, also line 195 and support-log Q14.

**Issue:** D9 ends the hold when "an attempt for the stop key hands on" and dates that end by "the hand-on attempt's verdict file time".
`verdict.yaml` is the agent's own word, not the verb's outcome.
In `internal/loomcli/arbiter.go`, `conclude` and `stopRunAndHandOn` route through `handOn`, which makes a hand-on with cause timeout, died or missing when no `verdict.yaml` exists, cause unparseable when it does not decode, and cause precondition when `verdict.yaml` says `resume` but `resumePrecondition` rejects it.
Only `finish` writes the outcome for every case, to `envelope.json` in the attempt directory.
D9's first line counts a timeout and a malformed verdict among the outcomes that notify the parent, and those are exactly the cases with no verdict file time.
A plan writer following the record reads `verdict.yaml`, so the hold misses a precondition hand-on, which reads as a resume, and has no end time for a timeout.

### [LOW] A same-key re-halt ends the hold with a stale time

**Location:** `_lyx/discussion/decision-record.md:135`, line 141.

**Issue:** The park marker end is limited to "this episode", but the hand-on end is not.
A stop key is producer, history length and state (`loomengine.StopKeyOf`), and batten's own `warnHaltEpisode` doc notes that a re-halt after an observed resume can recur at the same history length, since a hard producer error or a mid-call pause appends no history entry.
At such a re-halt the old attempt still exists, so the hold starts (the start condition admits "an attempt at this stop key"), then ends at once on that attempt's old hand-on, with an end time from the earlier episode.
`stateChangeReady` then finds `noticeReportWait` already spent and sends `report none yet` at once, before the driver's park marker, which today's rules would wait three minutes for.
Bounding the hand-on to the episode, and the end time to no earlier than since, keeps D9's own premise that the report wait starts no earlier than the hold's end.

### [NIT] The report wait's reason is circular

**Location:** `_lyx/discussion/decision-record.md:140`

**Issue:** "measures its report wait from the hold's end … since the report wait must start no earlier than the hold's end" gives the decision as its own reason.
The cause is `stateChangeReady`'s `waited := now.Sub(p.episode.since) >= noticeReportWait`: any hold past three minutes leaves that true at its end.
Line 143 states the effect, but not the cause.

### [NIT] An applied resume does leave a dated record

**Location:** `_lyx/discussion/decision-record.md:142`

**Issue:** `internal/loomengine/arbiter.go` lists the `applied` mark among an attempt directory's files, and `ParseArbiterAttemptDir` reads it.
Its write time dates the applied resume as a verdict file's dates a hand-on, so "leaves no record batten reads a time from" is a false premise, even if batten does not read that time today.
The impact is small, since an applied resume moves the run and usually ends the episode.

## Verdicts on the focus questions

- **Hand-over exemption.**
  It separates the two markers.
  Every Go-driver hand-back but the hand-over's own goes through `handBackAs` (`internal/loomcli/drive.go`), which appends a `hand-back` settle record before `handBack` writes the marker.
  `handOver` alone appends `hand-over` and calls `handBack` directly.
  The successor's `driveVerb` removes the stale park marker before its first action, so the hand-over's marker never sits beside a newer settle record.
  A settle append that fails is only logged, which leaves the successor's marker exempt until the backstop: bounded, and not a finding.
  The acceptance criterion at line 198 covers a successor that hands back without an arbiter attempt.
- **Signal batten reads.**
  The wording is exact: `readStopReport` (`internal/battencli/wire.go`) reads only the park marker's content and mtime, so the `startReport` files cannot end the hold.
  The stated reason for the report wait is circular (F3), not false.
- **End-time sources and restart.**
  Park marker and backstop are named and survive a restart.
  The hand-on source is wrong (F1) and unbounded to the episode (F2).
  An applied resume and the arbiter switched off fall back to batten's first observation, and a restart's own start time, as stated; F4 notes the premise for the applied resume.
