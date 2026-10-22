# Decision record — report timing finds idle time when the journal is unreadable

## Goal

When `lyx report timing` cannot read the systemd journal, it takes idle time from another source instead of counting sleep and held time as active, and its `sleep_note` says the figures may still include sleep.

## Scope

- `internal/sleepjournal`: `Read` also reports whether the journal was read, so a caller tells an unreadable journal from one that records no sleep.
- `internal/reportcli`: on an unreadable journal the timing verb reads the hub's pause record and hands the hub-wide pause intervals to the engine as the run's sleep, with a note naming the stand-in.
- Docs: `internal/reportcli/doc.go`'s sleep section and `internal/sleepjournal/doc.go`.
- Out of scope: the report engine, which already subtracts whatever intervals it is told; per-pair pause entries; inferring idle time from gaps in the runs' own timestamps.

## Decisions

- **The fallback source is the hub-scope entries of the pause record, live and resumed.**
  Rationale: a hub pause holds every session of the hub, so its window is idle for every run, and `lyx hibernate` always pauses the hub before it hibernates, so every sleep lyx itself takes is a hub pause.
  The record is already on the prime's disk and read through `pauserecord`; `CoverageOf` with an empty slug yields exactly the hub entries' intervals, a live entry's running to now.
- **The fallback applies only when the journal could not be read**: journalctl absent from PATH, failing, or printing output that does not parse.
  Rationale: a readable journal with no sleep is a true reading, and a hub pause is not machine sleep, so the source the board entry asks for stays the journal whenever it answers.
- **The stand-in intervals go into `TimingOptions.Sleeps` unchanged, and the engine is not touched.**
  Rationale: the engine takes sleep out of every row interval and wait before anything else, so a pause that batten also records as the paused phase is counted once, as sleep.
- **The note always says the figures may include sleep when the journal was unreadable**, whether pauses stood in, none were recorded, or the pause record could not be read, and names the journal's reason and, for an unreadable record, its error.
  Bound: the stand-in covers only hub pauses the record still keeps, which `pauserecord.Retention` bounds; a sleep taken without a hub pause, or older than the retention, stays counted as run time, and the note says so.

## Constraints

- `internal/sleepjournal` stays standard-library only (`PATTERN-leaf-packages`).
- The pause record path comes from `hubgeom.PauseGeometry`; reportcli derives no path (`PATTERN-cwd-resolution`, `PATTERN-lyxdirs-single-declarer`).
- The verb stays read-only: reading the record takes no lock and writes nothing.
- Go comments one sentence per line (`PATTERN-comment-line-breaks`).

## Auto-mode assumptions

- Hub pauses are the "hub-wide idle windows" and "recorded tender pauses" the board entry names; pair-scope pauses are left out because `TimingOptions.Sleeps` is machine-wide and a pair pause idles one run only.
- Labeling the stand-in time as sleep in `sleep_seconds` is acceptable since the note states where it came from.

## Open risks

- A machine that sleeps without `lyx hibernate` and without a hub pause still overstates active time; the note says the figures may include sleep.

## Acceptance criteria

- With journalctl absent or failing and the record holding a resumed hub pause, the sleeps are that pause's interval and the note names the journal's reason and the stand-in.
- A live hub pause runs to the time the verb is told.
- Pair-scope pause entries contribute nothing.
- With an unreadable journal and no hub pause recorded, no sleep is subtracted and the note says the figures may include sleep.
- With an unreadable journal and an undecodable pause record, no sleep is subtracted and the note names both reasons.
- A readable journal with no sleep, and one with sleep, behave as before and never consult the pause record.
- `sleepjournal.Read` reports the journal read for parsed output and unread for an absent, failing or undecodable run.
