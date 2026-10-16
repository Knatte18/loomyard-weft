# Card 5 — carried-log-entries

**Edit:**
- `internal/creelengine#LogEntry`
- `internal/creelengine#pool.apply`
- `internal/creelengine#pool.logEntryFor`
- `internal/creelengine#journalWriter.retain`
- `internal/creelengine#TestJournal_Retention`
- `internal/creelengine#TestJournal_Fold`
- `internal/creelengine_test#TestDaemon_Lifecycle`
- `internal/creelcli#logEntryFields`
- `internal/creelcli#TestCreelCLI`
- `internal/creelengine/doc.go`

**Uses:**
- `internal/creelengine#pool.applyCarried`
- `internal/creelengine#pool.carryRecords`
- `internal/creelengine#Store.Log`
- `internal/creelcli#newRig`
- `internal/creelcli#rig.startDaemon`

**Intent:** Fix #684 (D5): `log` shows a carried entry as a `carried` event.

`LogEntry` gains `Kind`, the kind of the message or inject a carried entry snapshots, empty on every other entry.
`pool.apply` adds one event-list entry for a carried record after `applyCarried` folds it, where today it returns before the append; a carried record whose `of` this binary does not know adds none, as an unknown event adds none.
`pool.logEntryFor` builds that entry with event `carried`, the carried record's own time, its id, its sender, its receiver, its kind (`mail` when the record names none), and the subject of the message the pool holds under that id; a carried hold's entry names the held receiver in `To` and the owner in `By`.
`journalWriter.retain` folds each carried record it writes into the pool through `pool.apply`, as `append` does for every other record, so a running daemon's `log` lists the carry at once; the fold is idempotent by id, so the entries the pool already holds are unchanged, and `prune` keeps these entries since they are of the new day.
A message carried again at a later retention gets one `carried` entry per carry, and `--from`, `--to` and `--since` match the entry like any other event.
The journal gains no new record shape (`PATTERN-tolerant-records`).
In `internal/creelcli`, `logEntryFields` prints `kind` when the entry carries one.

Tests:
- `TestJournal_Retention`: for both the writer's pool and the refolded journal, the events with id 1 end in one `carried` entry from alpha to gamma of kind mail at the day-eleven time, inject 3's carry has kind inject, and the live hold's carried entry names beta in `To` and alpha in `By`; this covers the carry written into a running writer and the fold of a journal holding the carried record.
  A second rotation seven more days on carries message 1, still unread, again, and its events then hold two `carried` entries, one per carry.
- `TestJournal_Fold`'s row "an unknown key and an unknown event name are ignored" also holds a carried record whose `of` this binary does not know, which adds no event-list entry, so the row's one-event count stands.
- `TestDaemon_Lifecycle`'s row "DayChangeRunsRetentionFromThePoll": a `log --id 1` request to the running daemon after the poll lists a `carried` entry for the old mail.
- `TestCreelCLI` gains the row "log --id prints a carried entry's kind", which pins only what `logEntryFields` adds: a journal day file planted before the daemon starts holds a carried record of an unread message, and `lyx creel log --id <id>` prints that entry's event `carried` with kind `mail`.

Docs in the same commit: the Storage and Log paragraphs of `internal/creelengine/doc.go` say each carried record is a `carried` log event with the snapshot's id, parties and kind at the carry's time, both after a fold and in the running daemon.

**ImpactSummary:** One field added to the exported `LogEntry` and one new event kind in its list; `Store.Log`'s filters and every other event are unchanged, and the CLI prints the new field only on carried entries.
