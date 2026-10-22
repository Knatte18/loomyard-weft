---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/reportcli/timing.go:158-163 and internal/sleepjournal/sleepjournal.go:Read
    summary: The fallback does not fire for a user who lacks journal permission, because journalctl exits 0 with empty stdout and Read reports that as a readable journal with no sleep
    origin: handler
    fix: In machineSleeps, capture journalctl's stderr (set cmd.Stderr to a bytes.Buffer on the exec.Cmd instead of calling Output) and pass it to sleepjournal.Read. In Read, when the run succeeded with empty stdout and stderr names missing access (the "Hint ... not seeing messages from other users and the system" line or "No journal files were opened due to insufficient permissions"), return no sleeps, a clause saying the journal is not readable by this user, and read=false. Empty stdout with no such stderr stays read=true. Add Read rows and a TestTimingSleepNote row for the permission case, and update the Read doc comment and reportcli/doc.go to name it.
  - id: F2
    severity: LOW
    class: design
    location: internal/reportcli/timing.go:82-86
    summary: Nothing tests the verb's wiring of the pause record path and now into machineSleeps, so a wrong or empty RecordPath passes every unit row
    origin: lens:code-tests
    fix: Extend the existing sleep_note check in internal/reportcli/convergence_integration_test.go:224 so that, with journalctl unavailable or failing in that fixture, the note must name the stand-in or report no recorded pause, not merely be non-empty. Add no new test function.
  - id: F3
    severity: NIT
    class: consistency
    location: internal/sleepjournal/sleepjournal.go:1
    summary: The file header still says Read maps a run to the sleeps and a reason clause, leaving out the new read result that doc.go now names
    origin: lens:code-affected-set
    fix: In the header comment on line 1, change "to the sleeps and a reason clause" to "to the sleeps, a reason clause and whether the journal was read".
---

### [BLOCKING] Fallback misses the permission-denied case the entry reports

**Location:** `internal/reportcli/timing.go:158-163`, `internal/sleepjournal/sleepjournal.go` `Read`.
**Issue:** The entry's machine cannot read the system journal because the user is in neither `adm` nor `systemd-journal`, and the report "says nothing about it".
A failing journalctl already produced a `sleep_note` (the triage note says so), so the silent case is a run that exits 0.
Without journal access, journalctl normally exits 0, prints only a permission hint on stderr, and writes nothing to stdout.
`Read` maps that to "the journal records no systemd-sleep operation" with `read=true`, so `sleepsFromJournal` takes the journal-read branch and never consults the pause record.
The new fallback therefore does not fire on the machine that motivated the change; it fires only for an absent or failing journalctl.
`machineSleeps` calls `.Output()`, which discards stderr, so the signal is not visible to `Read` today.
I could not reproduce the permission case: the reviewing user is in `adm`, and journalctl exited 0 there.
The finding rests on that exit-0 behaviour and on the entry's wording.

### [LOW] The verb's pause-record wiring is untested

**Location:** `internal/reportcli/timing.go:82-86`.
**Issue:** `TestTimingSleepNote` hands `sleepsFromJournal` a hand-made path, and `pauserecord.Read` treats a missing file as an empty record.
An empty or wrong `RecordPath`, or a wrong `now`, would pass every row.
The only integration check asserts a non-empty note.

### [NIT] Stale file header

**Location:** `internal/sleepjournal/sleepjournal.go:1`.
**Issue:** The header omits the read result that the package doc and the `Read` doc comment now state.

## Rejected

- A new failure mode from the eager `hubgeom.PauseGeometry` call (code-correctness, LOW): `resolvePrime` has already confirmed the prime worktree at `timing.go:66`, and `primeGeometry` follows it, so the call adds no realistic failure.
- A hub-pause window reversed by clock skew (code-correctness, noted and not filed): the fork did not trace the engine and raised no defect; the engine unions and clips the intervals.
- `PATTERN-leaf-packages.md` wording (code-affected-set): it names no signature and stays true.
