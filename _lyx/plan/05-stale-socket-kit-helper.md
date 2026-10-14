# Card 5 — One stale-socket helper, in tmuxkit

**Create:**
- `plan:internal/testkit/tmuxkit#BindStaleUnixSocket` -> `func BindStaleUnixSocket(t *testing.T, dir, key string)`

**Delete:**
- `internal/reedengine#bindStaleUnixSocket`

**Edit:**
- `internal/reedengine#TestRemoveSocketFileOnceGone`
- `internal/reedengine#TestRemoveStaleSocket`
- `internal/testkit/tmuxkit#TestRemoveDeadSocket`

**Intent:**
Decision D5, per `PATTERN-testkit`.
`reedengine`'s private `bindStaleUnixSocket` (in `socketfile_test.go`) moves to `tmuxkit.go` as the exported `BindStaleUnixSocket`, body and doc comment unchanged in substance.
It binds a datagram socket at `dir/key` and closes it, leaving a socket file that refuses every stream connection.
A child forked by a parallel test cannot make that path answer a `Dial`, as a closed listener's inherited copy can.
`tmuxkit` is the kit because `reedengine`'s tests already import it for `TestMain`, and it does not import `reedengine`.
The function sits in the kit's non-test source and is imported only from tests, as every kit is.
`tmuxkit`'s package doc gains one sentence naming it.

Callers:
- `TestRemoveSocketFileOnceGone` and `TestRemoveStaleSocket` set `setup: tmuxkit.BindStaleUnixSocket` on their stale-socket rows.
- `TestRemoveDeadSocket` builds its dead socket with `BindStaleUnixSocket(t, dir, "dead")` in place of listening and closing a listener.
  It calls `removeDeadSocket(dead, 0)` like the other paths, dropping the `deadSocketWait` fork-inheritance wait and its comment.
  `deadSocketWait` itself stays, because `killServer`'s own call still uses it.
- The `listen` helper stays for the live socket.

The existing assertions are the coverage: the dead socket is removed and the live one and the regular file stay.
They now pass with no wait.

**ImpactSummary:** Test-only; three socket tests switch to the shared kit helper, and tmuxkit's dead-socket test no longer waits.

**Commit:** 5: tmuxkit: share the datagram stale-socket helper and drop the fork wait
