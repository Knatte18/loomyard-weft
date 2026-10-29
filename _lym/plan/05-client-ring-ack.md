# Card 5 — client-ring-ack

**Edit:**
- `internal/creelengine#Client.Call`
- `internal/creelengine#Client.CallWatching`
- `internal/creelengine#Client.resume`
- `internal/creelengine#Client.exchange`
- `internal/creelengine#waitState`
- `internal/creelengine_test#TestDaemon_Lifecycle`
- `internal/creelengine/doc.go`

**Create:**
- `plan:internal/creelengine#ErrAckUnsent` -> `var ErrAckUnsent = errors.New("creel: the acknowledgement could not be sent, so the daemon journaled nothing")`

**Uses:**
- `internal/creelengine_test#rig.client`
- `internal/creelengine_test#result`

**Intent:**
Decision D3, client side: a hook-wait whose ring reached its handler never resumes the wait, and its caller can tell an acknowledgement that was never written from one that was.

`Client.exchange`: a failed write of the ack line returns a `*connLostError` wrapping `ErrAckUnsent` beside the write error, so `errors.Is(err, ErrAckUnsent)` holds and a read's reconnect is unchanged; a lost or refused confirmation keeps its error, which never matches `ErrAckUnsent`.
Before calling `handle`, exchange sets the struct `waitState`'s new field `handled bool`.
`Client.CallWatching` returns the error at once, with no resume and no reconnect, for a `VerbHookWait` request whose state is `handled`; every other verb and a hook-wait whose handler never ran resume as today.
What this lets through: nothing new; it removes the resume of a hook-wait whose mail the daemon may already have journaled, bounded to the hook-wait verb, so `read --wait` keeps its resume.
The doc comments of `Client.Call` and `Client.resume` say a hook-wait resumes only until a ring reaches the handler.

Tests (tier0), two rows in `TestDaemon_Lifecycle`, each with no daemon: the test listens on the rig's socket path itself, counts the connections it accepts, answers the hook-wait request line with a pending line and a ring line carrying one message and `ack_required` (both with the rig's hub path), and runs `rig.client(...).Call` with a `VerbHookWait`, `Wait: true` request and a handler in a goroutine of its own that sends the call's outcome into a `callResult` channel, taken with `result`, whose real-time deadline makes a client that resumes fail rather than hang on its fake clock (`callAsync` passes no handler):
- `AHookWaitWhoseConfirmationIsLostAfterItsRingReachedTheHandlerIsNotResumed`: the fake reads the ack line and closes without confirming; the call returns an error that is not `ErrAckUnsent`, and the fake accepted one connection.
- `AHookWaitWhoseAckCannotBeSentReturnsErrAckUnsent`: the fake closes right after the ring line and the handler returns only once the fake has closed; the error matches `ErrAckUnsent`, and the fake accepted one connection.

`internal/creelengine/doc.go`'s sentence "A dropped hook-wait is resumed by sending the same request again, up to MaxReconnects" adds: until a ring reaches the caller's handler, after which a lost connection ends the call; and it names `ErrAckUnsent`.

**ImpactSummary:** One new error sentinel and a no-resume branch for one verb in the Go client; no exported signature changes, and every other verb's reconnect path is untouched.
