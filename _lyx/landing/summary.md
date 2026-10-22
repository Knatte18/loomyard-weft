# Report timing falls back to hub pauses when the journal is unreadable

`lyx report timing` took machine sleep only from `journalctl`.
When the user may not read the system journal, `journalctl` exits 0 with empty output and a permission hint on standard error, and the report read that as "no sleep".
Sleep and held time were counted as active run time, and the `sleep_note` did not say the journal had been unreadable.

Now an unreadable journal is told apart from one that records no sleep.
When the journal could not be read, the verb reads the hub's pause record and hands the engine every hub-scope pause interval as the run's sleep, a live pause running to now.
The `sleep_note` names why the journal was not read, says whether pauses stood in, and says the figures may include sleep no hub pause covers.

Closes #803

## Design decisions

- **Hub pauses are the stand-in.**
  A hub pause holds every session of the hub, and `lyx hibernate` pauses the hub before the machine sleeps, so its window is idle for every run.
  Pair-scope entries are left out, because the engine's sleep is machine-wide and a pair pause idles one run only.
- **The fallback applies only when the journal was not read.**
  That covers `journalctl` absent, failing, a permission-denied journal, or output that does not parse.
  A readable journal with no sleep is a true reading and never consults the pause record.
- **The engine is untouched.**
  It takes sleep out of every interval before anything else, so a pause that is also recorded as the paused phase is counted once, as sleep.
- **`sleepjournal.Read` gains a third result, whether the journal was read, and a `stderr` parameter.**
  Access denial is detected from the two permission messages `journalctl` prints.
  `tools/tokencount` is updated for the new signature and captures stderr too, so it also names an unreadable journal instead of reporting no sleep.
- **An undecodable pause record** leaves nothing out, and the note names both reasons.
- **Bound:** the stand-in covers only the hub pauses `pauserecord.Retention` keeps.
  A sleep taken without a hub pause, or older than the retention, still counts as run time, and the note says so.
- The verb stays read-only: reading the record takes no lock and writes nothing, and its path comes from `hubgeom.PauseGeometry`.
- `sleep_seconds` labels the stand-in time as sleep; the note states where it came from.

`internal/reportcli/doc.go`, `internal/sleepjournal/doc.go` and the verb's help text describe the fallback.

## Not covered

- Idle time inferred from gaps in the runs' own timestamps is not attempted.
- Nothing was run against a machine whose journal is genuinely unreadable; the tests feed `journalctl` output, standard error and errors directly, and the integration check seeds a hub pause.
