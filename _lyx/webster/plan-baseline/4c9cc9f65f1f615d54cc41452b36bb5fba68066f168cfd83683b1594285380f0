# Card 10 — read-checked-write

**Create:**
- `internal/creelcli#writeOk`
- `internal/creelcli#failingWriter`
- `internal/creelcli#failingWriter.Write`

**Edit:**
- `internal/creelcli#readCmd`
- `internal/creelcli#messageFields`
- `internal/creelcli#TestCreelCLI`
- `internal/creelcli/doc.go`

**Uses:**
- `internal/creelcli#rig.startDaemon`
- `internal/creelcli#rig.sweepStep`
- `internal/creelcli#rig.sweepLast`
- `internal/creelcli#runCreel`
- `internal/creelengine#Client.Call`
- `internal/creelengine#Message`

**Intent:** Decisions D4 and D7: a `read` whose output cannot be written acknowledges nothing, and `read` shows a message the removed-holder sweep returned.

D4: `writeOk` renders the fields as `output.Ok` does, `ok` set to true and one JSON line, and returns the write's error; `output.Ok` keeps its signature.
`readCmd`'s handler writes its envelope with `writeOk` and returns the write error, so `Client.Call` sends no acknowledgement, the daemon releases the reservation, the messages stay unread and the verb exits non-zero through `refuse`.
The nothing-unread path, which prints after `Call` returns, checks its write the same way and returns the error.
A partial write followed by an error leaves the messages unread, so a consumer may hold a truncated envelope of messages it reads again later; the handler's doc says so.

D7: `messageFields` adds `ended_unread_by` and `ended_unread_at` when the message's `EndedUnreadBy` is set; the fold already holds both.

Tests, rows of `TestCreelCLI`:
as a strand with a message, `read` into a `failingWriter` (whose `Write` always fails) exits non-zero, and a `read` into a buffer then prints that message, retried under a real-time deadline while it answers nothing unread, since the daemon releases the reservation only once it sees the dropped connection;
with nothing unread, `read` into a `failingWriter` exits non-zero;
and a message the strand posted to a second strand, which the fake delivery then drops so two sweeps return the message to its sender, reads back with `ended_unread_by` naming the second strand and an `ended_unread_at`.

`internal/creelcli/doc.go` says `read` acknowledges its messages only once its envelope was written.

**ImpactSummary:** Changes the read verb's write and the message rendering only; every other verb keeps output.Ok.
