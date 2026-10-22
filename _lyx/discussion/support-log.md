# Support log — report timing finds idle time when the journal is unreadable

## Interview

A darn run holds no interview: the writer worked from the board entry and a reading of `internal/reportcli`, `internal/sleepjournal`, `internal/reportengine` and `internal/pauserecord`.

## Rejected alternatives

- **Inferring idle time from gaps in the runs' own timestamps.**
  A long gap between rows is also a long agent step, such as a Webster batch, so any threshold would subtract real work; the fallback would swap one silent error for another.
- **Reading sleeps from an in-process clock watch (`pauserecord.SleepWatch`).**
  It is in-memory per process and records nothing a later report can read.
- **Using hub pauses even when the journal reads.**
  A hub pause is not machine sleep, and the batten record already shows a run's held time as the paused phase; mixing them in would relabel paused time as sleep on machines where the journal answers.
- **Including pair-scope pause entries.**
  They idle one run, and `TimingOptions.Sleeps` is machine-wide; carrying them per run would need a new engine option for a case batten's paused phase already counts.
- **A new engine option for the stand-in, apart from sleep.**
  The engine's arithmetic already treats any told interval as idle and nets it out first, so a second field would duplicate that path for no different figure.

## Review rounds

_No rounds yet._

## Question ledger

- Which recorded source stands in for sleep? Self-picked: hub-scope pause entries, since every lyx hibernate starts with one and each holds every run.
- Does the fallback also cover a missing journalctl? Self-picked: yes; a missing journalctl is as unreadable as a failing one, and a non-systemd machine gains the same stand-in.
- Does a journal that reads but records no sleep fall back? Self-picked: no; that is a true reading.
