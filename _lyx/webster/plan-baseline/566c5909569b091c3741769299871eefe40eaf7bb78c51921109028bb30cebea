# Card 6 — creelengine-journal

**Create:**
- `internal/creelengine/doc.go`
- `internal/creelengine/geometry.go`
- `internal/creelengine/journal.go`
- `internal/creelengine/fold.go`
- `internal/creelengine/journal_test.go`
- `internal/creelengine/testmain_test.go`
- `plan:internal/creelengine#Geometry` -> `type Geometry struct`
- `plan:internal/creelengine#SocketName` -> `func SocketName(shortname, hubPath string) string`
- `plan:internal/creelengine#Clock` -> `type Clock interface`
- `plan:internal/creelengine#Message` -> `type Message struct`

**Edit:**
- `//cmd/lyx/testdata/test-budget.yaml`

**Intent:** Start the engine package with its told geometry and its storage: one append-only journal folded into the pool (D6).
`Geometry` carries the absolute paths the engine is told and derives nothing from (`PATTERN-told-geometry`): `HubPath`, `Shortname`, `StateDir` (`<hub>/_board/.lyx/creel`), `JournalDir` (`StateDir/journal`), `LockPath` (the daemon lock in `StateDir`) and `SocketPath`.
`SocketName` returns `creel-<shortname>-<h>.sock`, `<h>` the first 8 hex digits of the SHA-256 of `hubPath` (D3); `internal/hubgeom` joins it to the lyxtemp root (card 12).
`Clock` is the time seam (`Now`, `After`), so no test waits out an interval.
`journal.go` holds the unexported journal writer and reader: one JSONL file per UTC day (`YYYY-MM-DD.jsonl`) under `JournalDir`; an append writes one line and `fsync`s before returning, and a day file opens with a high-water record of the largest id issued.
Events are `posted`, `reminded`, `read`, `forwarded`, `returned`, `undelivered`, `delivered`, `sending`, `injected`, `unconfirmed`, `expired`, `aged`, `interrupted`, `held`, `released`, `withheld` (an orch receiver's reminder held back until `creel-notices`, D16), plus the `carried` record retention writes and the `high-water` record.
`fold.go` folds the retained files into the pool state: `Message` (id, `From`, `Holder`, `Path`, `Kind` mail or reply, `Subject` = first line, `Body`, `ReplyTo`, times, `Status` unread/read/undelivered/delivered, `EndedUnreadBy`, the receiver that ended unread, with its time, for a returned message), queued injects (id, from, receiver, text, TTL, consumed TTL from the last `aged` event, head-of-queue flag, outcome), holds (receiver, owner, expiry), the withheld reminders per orch receiver and the known program-sender mailboxes.
The fold is idempotent by id: a `carried` record and the original line of one message fold to one entry.
An inject with a `sending` event and no `injected`, `unconfirmed` or `expired` outcome folds as unconfirmed, so one in flight at a crash is never retyped (D9).
A new id exceeds both every high-water record and the fold's largest id, so ids never repeat, also after retention deleted the files holding the largest ids.
A reader ignores unknown keys (`PATTERN-tolerant-records`), ignores a line whose event name it does not know (a newer binary's event) with one Debug line, and skips a torn last line from a crash with one Warn; a malformed line that is not the last fails the load naming the file and line and the way forward, to repair or remove that line by hand, since skipping it could drop a message silently.
A missing or empty journal directory folds to an empty pool, the fresh hub.
Retention: at a UTC day change the writer appends to today's file a `carried` record for each unread message, queued inject (with its consumed TTL) and live hold held by files older than 7 days, then deletes those files.
The writer and its append are unexported, so the daemon (card 10) is their only caller (`PATTERN-creel-store`, card 11).
`doc.go` opens the package header: creel is the per-hub message daemon, `creelengine` is told its geometry and a delivery seam, and this card's storage section; card 16 writes the full design.
`testmain_test.go` runs the package under `tmuxkit.Main`, so every temp file and socket lives in the lyxtemp run directory.
`TestJournal_Fold` table rows: a missing and an empty journal directory fold to an empty pool; posted, forwarded, read, returned and withheld events fold to the expected holder, path, status and ended-unread receiver; an unknown key and an unknown event name are ignored; a torn last line is skipped with one Warn; a malformed middle line fails naming the file; a `sending` without outcome folds unconfirmed; the last `aged` event sets the consumed TTL; ids continue past a high-water record whose file has the largest id deleted.
`TestJournal_Retention` with an injected clock: files older than 7 days are deleted after their unread messages, queued injects (consumed TTL kept) and live holds are carried; read messages, finished injects and expired holds are not carried; a crash between the carry and the deletion (both the carried records and the old files present) folds to no duplicate.
`SocketName` is pinned once, by card 12's `TestCreelGeometry`.
`cmd/lyx/testdata/test-budget.yaml` gains an `internal/creelengine: 2` row.

**ImpactSummary:** New package, no existing caller; one new budget row.
