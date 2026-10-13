# Card 5 — One stale-socket helper, in tmuxkit

**Create:**
- `internal/testkit/tmuxkit/stalesocket.go`
- `plan:internal/testkit/tmuxkit#BindStaleSocket` -> `func BindStaleSocket(t *testing.T, path string)`

**Edit:**
- `internal/testkit/tmuxkit/tmuxkit.go`
- `internal/testkit/tmuxkit#TestRemoveDeadSocket`
- `internal/reedengine#TestRemoveSocketFileOnceGone`
- `internal/reedengine#TestRemoveStaleSocket`

**Delete:**
- `internal/reedengine#bindStaleUnixSocket`

**Intent:**
D5: the datagram-built stale socket moves from `reedengine`'s tests into the `tmuxkit` test kit, per `PATTERN-testkit`, and the one remaining test that builds a dead socket by closing a listener uses it.
A closed listener is the form a parallel test's forked child can keep answering until its exec; a datagram socket never listens, so no inherited fd can make its path answer a `Dial`.

`BindStaleSocket`, in the new file `stalesocket.go`, binds a `unixgram` socket at `path` and closes it, leaving a socket file that refuses every stream connection, and fails the test with `t.Fatalf` on its own setup only, as `bindStaleUnixSocket` does today.
It imports the standard library only, inside what `tmuxkit` may import.
`tmuxkit.go`'s package doc names `BindStaleSocket` beside the other helpers it lists.
`reedengine`'s tests already import `tmuxkit` through their `TestMain`, so no new import edge forms.

`TestRemoveSocketFileOnceGone` and `TestRemoveStaleSocket`, the two callers of `bindStaleUnixSocket`, set their stale-socket rows' `setup` to a closure calling `tmuxkit.BindStaleSocket(t, filepath.Join(dir, key))`, and `bindStaleUnixSocket` is deleted.
`TestRemoveDeadSocket` builds its dead socket with `BindStaleSocket` in place of a listener it closes, and calls `removeDeadSocket(dead, 0)` with no wait, dropping the fork-inheritance comment the wait carried; its assertions are unchanged, so it still fails if `removeDeadSocket` kept a refusing socket or removed the live one or the plain file.
`deadSocketWait` keeps its production use in `tmuxkit.go`.

**ImpactSummary:** a test-only helper moves from `reedengine`'s tests to `tmuxkit`; its two callers and `TestRemoveDeadSocket` are on this card, and no production code changes.

**Commit:** 5: Move the stale-socket helper into tmuxkit
