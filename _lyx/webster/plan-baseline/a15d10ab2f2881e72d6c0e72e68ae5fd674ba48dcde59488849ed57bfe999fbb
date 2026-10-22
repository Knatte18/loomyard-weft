# Card 19 — reed-client-attached

**Create:**
- `plan:internal/reedengine#Engine.ClientAttached` -> `func (e *Engine) ClientAttached() (bool, error)`
- `plan:internal/reedengine#TestClientAttached` -> `func TestClientAttached(t *testing.T)`

**Uses:**
- `internal/reedengine#newAttachTestEngine`
- `internal/reedengine#parseClientList`
- `internal/reedengine#ErrNoSession`

**Intent:**
The live fact a per-machine Open-Window reads (D8).
`Engine.ClientAttached` lists the session's clients (`list-clients -t <session>`) and reports whether any is attached; a missing session reads false with no error, and any other tmux failure is returned.
`TestClientAttached` drives a fake tmux: one client attached reads true, no clients reads false, a missing session reads false, and a failed listing returns the error.

**ImpactSummary:** reed gains one client read; no existing caller changes.

**Commit:** 19: reed: report whether a session has a client attached
