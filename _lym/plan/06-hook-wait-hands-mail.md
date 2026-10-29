# Card 6 — hook-wait-hands-mail

**Edit:**
- `internal/creelcli#hookWait`
- `internal/creelcli#hookWaitCmd`
- `internal/creelcli#TestCreelCLI`
- `internal/creelcli#Command`
- `internal/creelcli/doc.go`
- `docs/overview.md`

**Create:**
- `plan:internal/creelcli#hookWaitExit` -> `func hookWaitExit(wakeCode int, wroteLines bool, err error) int`
- `plan:internal/creelcli#TestHookWaitExit` -> `func TestHookWaitExit(t *testing.T)`

**Uses:**
- `plan:internal/creelengine#ErrAckUnsent`
- `internal/creelcli#newHookRig`
- `internal/creelcli#runCreelHook`
- `internal/creelcli#hookLine`

**Intent:**
Decision D3, hook side: `hook-wait` writes the ring's mail as the wake and acknowledges it, and `creelengine.BellLine` is no longer its output.

`hookWait` sends `Request{Verb: VerbHookWait, Wait: true, TakesMail: true}` with a handler: the handler joins the response's messages' `creelengine.MessageLine`s with newlines, takes `renderer.RewakeOutput(text)`, writes the stderr bytes and sets `wroteLines` only once the write succeeded, returning the write error otherwise, so the client sends no acknowledgement.
`hookWait` returns `hookWaitExit(code, wroteLines, err)`, where `code` is `RewakeOutput`'s exit code, with every early return (unknown engine, unreadable input, subagent, no hub client) still 0.
`hookWaitExit` returns 0 when no lines were written (a superseded wait, a refused or failed call before the ring, a ring with no messages from an older daemon whose response asks no acknowledgement, a failed write of the lines), 0 when `err` matches `creelengine.ErrAckUnsent`, and the wake code otherwise: the acknowledgement was written, whether the daemon confirmed it, refused it on a failed journal write or the confirmation was lost.
An exit 0 after the lines' write fails drops the stderr Claude Code would show, so the mail stays unread and the ring unanswered, and the bell's window applies; an exit with the wake code after a refused or lost confirmation shows mail the daemon may not have journaled, the duplicate D3 accepts.
`hookWaitCmd`'s Long and `internal/creelcli/doc.go`'s hook-wait sentences say that a ring writes the claimed lines as the engine's wake and acknowledges them, with these exit rules, and that the wait is never resumed once a ring reached it.
The `lym creel` group help in `Command`'s Long says an idle strand is woken by its waiter with the mail itself, and `docs/overview.md`'s creel paragraph says the same where it says the woken turn's hooks hand it the mail.

Tests:
- `TestHookWaitExit` (tier0), a table in `internal/creelcli/hookread_test.go`: lines written and confirmed (2), written and the confirmation refused or lost, an error other than `ErrAckUnsent` (2), written and the ack unsent (0), lines not written (0), nothing written and no error, the no-lines ring or supersede (0).
- (tier1) `TestCreelCLI`'s subtest `hook-wait rings a strand as exit 2 with the bell line, …` becomes `hook-wait rings a strand as exit 2 with its mail, …`: stderr equals `hookLine(1, alpha, "mail")` and a newline, and `lym creel log` holds a `read` event for #1; its subagent and refusal rows stay.
- (tier1) A new `TestCreelCLI` subtest `hook-wait whose stderr refuses the lines exits 0 and leaves the mail unread`: with `failingWriter` as stderr the exit is 0 and stdout empty; the daemon releases the ring's claim on its own goroutine once it reads the closed connection, so the row runs the tool-boundary `hook-read` again until it prints, a run before the release printing nothing and claiming nothing, the real-time deadline only bounding a failing test, and then requires the mail's hook output and a `read` event for #1 in `lym creel log`.

**ImpactSummary:** hookWait keeps its signature and its one caller; only its ring branch and exit decision change, pinned by one new table test and two CLI subtests.
