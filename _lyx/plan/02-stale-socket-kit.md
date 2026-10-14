# Card 2 — stale-socket-kit

**Create:**
- `internal/testkit/tmuxkit/stalesocket.go`
- `plan:internal/testkit/tmuxkit#BindStaleUnixSocket` -> `func BindStaleUnixSocket(t *testing.T, dir, key string)`

**Delete:**
- `internal/reedengine#bindStaleUnixSocket`

**Edit:**
- `internal/reedengine#TestRemoveSocketFileOnceGone`
- `internal/reedengine#TestRemoveStaleSocket`
- `internal/testkit/tmuxkit#TestRemoveDeadSocket`

**Intent:** Move the datagram-built stale socket helper into the `tmuxkit` test kit (D5, `PATTERN-testkit`), so one copy exists and both packages' tests share it.
`BindStaleUnixSocket` lives in a new non-test file of `tmuxkit`, `stalesocket.go`, with the body and doc comment of today's `reedengine` `bindStaleUnixSocket`: it binds a `unixgram` socket at `dir/key` and closes it, failing the test with `t.Fatalf` on its own setup only.
It imports `net`, `path/filepath` and `testing` only, so it stays inside the kit rules `internal/testkit/enforcement_test.go` scans; `reedengine`'s tests already import `tmuxkit`, so no new edge appears.
Delete the private `reedengine` copy and point the stale-socket rows of `TestRemoveSocketFileOnceGone` and `TestRemoveStaleSocket` at `tmuxkit.BindStaleUnixSocket`.
In `TestRemoveDeadSocket`, build the dead socket with `BindStaleUnixSocket(t, dir, "dead")` instead of listening and closing a listener, drop the `SetUnlinkOnClose(false)` close dance for it, and call `removeDeadSocket(dead, 0)` like the other paths: a datagram socket never listens, so no parallel test's forked child can make it answer, and the fork-inheritance wait and its comment go.
The live socket keeps its listener.
`deadSocketWait` itself stays, since `tmuxkit.go` still uses it for killed servers.
The changed rows are the coverage: the stale socket must still be removed with no wait.

**ImpactSummary:** Test-only: one helper moves from a reedengine test file into tmuxkit, two reedengine tests and one tmuxkit test switch to it; no production code changes.
