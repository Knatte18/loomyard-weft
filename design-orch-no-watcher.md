# The orch relies on driver messages, not a scratch watcher

`orch-no-watcher` · Notes · design

Seen 2026-10-03: the hub orch watches each driven run with a hand-written `.scratch/hub/watch.sh`, a background shell loop that polls the run's `status.json` every 30 s and exits on the first non-running state change.
It lives outside lyx, untested and uncommitted, and has to be re-armed by hand after every context cycle.
The driver already messages its parent (`ly:orch`) at every stop it cannot handle, which is the path the orch should rely on.
Idea (operator): question whether the orch needs a watcher at all; if a gap remains that the driver's message does not cover (a driver that dies without messaging, a stop the driver does not report), close it in lyx's own code, never with a scratch script.
Likely home: `lyx batten run`, whose Run-Shed row already polls the same `status.json` every 30 s.
Batten could message its parent on a non-running state change, and check the driver strand's liveness through reed to cover the case `internal/battenshed/doc.go` leaves out by design (a driver that dies, parks on a prompt, or stops with the run non-terminal).
Batten can also be started by hand from a terminal with no orch above it, so the parent is optional: message it when one is recorded, otherwise only log.
