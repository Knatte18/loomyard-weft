# Card 6 — hook-wait-hands-mail

**Edit:**
- `internal/creelcli#hookWait`
- `internal/creelcli#hookWaitCmd`
- `internal/creelcli#TestCreelCLI`
- `internal/creelcli#Command`
- `internal/creelcli#postCmd`
- `internal/creelcli/doc.go`
- `internal/creelengine/doc.go`
- `docs/overview.md`
- `contracts/stencils/orch/orch-template-role.md`

**Create:**
- `internal/creelcli#hookWaitExit`
- `internal/creelcli#TestHookWaitExit`

**Uses:**
- `internal/creelengine#ErrAckUnsent`
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
The `lym creel` group help in `Command`'s Long says an idle strand is woken by its waiter with the mail itself, `postCmd`'s Long says the same where it says "an idle one woken first", and `docs/overview.md`'s creel paragraph says the same where it says the woken turn's hooks hand it the mail.
The orch stencil `contracts/stencils/orch/orch-template-role.md` follows in step 4: the sentence on where a `[creel] #<id> from "<sender>": <text>` line arrives adds the wake beside a tool result and the turn end, and the sentence after it says an idle orch is woken with its mail under `[creel] Mail that arrived while you were idle:` and acts on it in that turn, and that only an orch no waiter reaches gets the fixed line `[creel] Mail is waiting for you.`, after which its hooks hand it the mail as that turn ends.
`internal/creelengine/doc.go`'s Idle wake section gains D3's Bounds that rest on the hook showing the mail: the mail is journaled read before Claude Code shows it, so a session that dies, or a hook Claude Code ends at its timeout or at session end, between the journaled read and Claude Code reading the exit loses it as read; an acknowledgement whose confirmation is refused or lost shows the mail the daemon did not journal twice, once in the wake and once at a later claim, and the bell may follow the woken turn; Claude Code escapes `system-reminder` tags and replaces its own marker patterns in the wake body, so such mail reaches the model altered.
Its claim paragraph's delivered-once sentence names the Constraint's two exceptions: `hook-read`, and `hook-wait` once its acknowledgement is written, show the mail whatever the acknowledgement's outcome, so mail the daemon did not journal is handed out once more; and the ring's mail is journaled read before it is shown, so the loss window loses it as read.

Tests:
- `TestHookWaitExit` (tier0), a table in `internal/creelcli/hookread_test.go`: lines written and confirmed (2), written and the confirmation refused or lost, an error other than `ErrAckUnsent` (2), written and the ack unsent (0), lines not written (0), nothing written and no error, the no-lines ring or supersede (0).
- (tier1) `TestCreelCLI`'s subtest `hook-wait rings a strand as exit 2 with the bell line, …` becomes `hook-wait rings a strand as exit 2 with its mail, …`: stderr equals `hookLine(1, alpha, "mail")` and a newline, and `lym creel log` holds a `read` event for #1; its subagent and refusal rows stay.
- (tier1) `TestCreelCLI`'s subtest `a hook-wait superseded by a newer one of the strand exits 0 printing nothing` changes its newer waiter's stderr assertion from `creelengine.BellLine` and a newline to `hookLine(1, alpha, "mail")`, a newline, `hookLine(2, alpha, "more mail")` and a newline, its exit 2 unchanged.
- (tier1) A new `TestCreelCLI` subtest `hook-wait whose stderr refuses the lines exits 0 and leaves the mail unread`: with `failingWriter` as stderr the exit is 0 and stdout empty; the daemon releases the ring's claim on its own goroutine once it reads the closed connection, so the row runs the tool-boundary `hook-read` again until it prints, a run before the release printing nothing and claiming nothing, the real-time deadline only bounding a failing test, and then requires the mail's hook output and a `read` event for #1 in `lym creel log`.

**ImpactSummary:** hookWait keeps its signature and its one caller; only its ring branch and exit decision change, pinned by one new table test and two CLI subtests.
