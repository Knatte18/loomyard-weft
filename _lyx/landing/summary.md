# Report timing falls back to hub pauses when the journal is unreadable

`lyx report timing` subtracts machine sleep it reads from the systemd journal.
On a machine whose user cannot read the journal, it subtracted nothing, so sleep and held time counted as active run time.

## What changes

- **Fallback idle source.**
  When journalctl is absent from PATH, fails, or prints output that does not parse, the verb reads the hub's pause record and hands the engine every hub-scope entry's interval as the run's sleep.
  A resumed entry runs from its pause to its resume, and a live one to the time the verb runs.
  A hub pause holds every session of the hub, and `lyx hibernate` pauses the hub before the machine sleeps, so every sleep lyx itself takes is covered.
- **The note says what happened.**
  `sleep_note` now names why the journal was not read and either says the hub's pauses stand in, or that none are recorded or the record could not be read.
  Each of these says the figures may include sleep.
- **A readable journal behaves as before.**
  A journal that records no sleep is a true reading: nothing is subtracted, and the pause record is not consulted.
- **`sleepjournal.Read` gains a third result** reporting whether the journal was read, so callers tell a machine that did not sleep from one whose sleep is unknown.
  `tools/tokencount` ignores it and keeps its output.

## Design decisions

- **Hub-scope entries only.**
  The engine's sleep is machine-wide, while a pair-scope pause idles one run, and batten already counts a pair's held time as the paused phase.
- **The engine is untouched.**
  It takes sleep out of every row interval and wait first, so a hub pause that batten also records as the paused phase counts once, as sleep.
- **Fallback only on an unreadable journal.**
  A hub pause is not machine sleep, so with a journal that answers, the journal stays the only source.
- **Gaps in the runs' own timestamps were not used.**
  A long gap is as often a long agent step as an idle machine, so any threshold would subtract real work.
- Bound: the record keeps resumed entries for `pauserecord.Retention`, so a sleep with no hub pause, or older than that, still counts as run time; the note says the figures may include sleep.

## Docs

`internal/reportcli/doc.go`'s sleep section, the verb's help text and `internal/sleepjournal/doc.go` describe the fallback and the new result.

## Tests

`TestTimingSleepNote` in `internal/reportcli` now drives a real pause record file: hub pauses, live and resumed, stand in for a missing journalctl while a pair pause does not; a failing journalctl with no record; an undecodable journal and record named together; and a readable journal, with or without sleep, that leaves the record unread.
The `Read` rows in `internal/sleepjournal` assert the new read result.
