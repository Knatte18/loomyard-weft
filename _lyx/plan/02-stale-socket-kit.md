# Card 2 — stale-socket-kit

**Create:**
- `plan:internal/testkit/tmuxkit#BindStaleSocket` -> `func BindStaleSocket(t *testing.T, path string)`

**Edit:**
- `internal/testkit/tmuxkit#TestRemoveDeadSocket`
- `internal/reedengine#TestRemoveSocketFileOnceGone`
- `internal/reedengine#TestRemoveStaleSocket`

**Delete:**
- `internal/reedengine#bindStaleUnixSocket`

**Intent:** Move the datagram-built stale socket into a test kit per D5 and `PATTERN-testkit`, so it exists once.
`BindStaleSocket` goes in `internal/testkit/tmuxkit/tmuxkit.go` (a non-test file of the kit, which already imports `net`): it binds a `unixgram` socket at `path` and closes it, leaving a socket file that refuses every stream connection, and fails the test through `t.Fatalf` on a bind or close error.
Its doc comment carries the reason `bindStaleUnixSocket` gave: a datagram socket never listens, so a child a parallel test forks before its exec cannot make the path answer a `Dial` the way an inherited copy of a closed listener can.
It asserts nothing beyond its own setup, per the kit rules; callers skip on Windows before calling it, as they already do.

Delete `bindStaleUnixSocket` from `internal/reedengine/socketfile_test.go`.
Its two callers, the `stale socket is removed` rows of `TestRemoveSocketFileOnceGone` and `TestRemoveStaleSocket`, set their `setup` to a closure calling `tmuxkit.BindStaleSocket(t, filepath.Join(dir, key))`; the reedengine tests already import `tmuxkit` through `testmain_test.go`.

`TestRemoveDeadSocket` builds its `dead` socket with `BindStaleSocket(t, filepath.Join(dir, "dead"))` in place of listening and closing a listener, removes the `listen` closure's dead-socket use, and calls `removeDeadSocket(dead, 0)` with no wait, dropping the fork-inheritance comment and the `deadSocketWait` argument there.
Its assertion that the dead socket is gone stays, so the test still fails if `removeDeadSocket` keeps a refusing socket; `removeDeadSocket` removes on any `net.Dial` error, which a datagram socket gives at once.
The `// Package tmuxkit` comment at the top of `tmuxkit.go`, the file `BindStaleSocket` lands in, gains one sentence naming `BindStaleSocket` as the shared stale-socket fixture.

**ImpactSummary:** Test-only: one helper moves from reedengine's tests into tmuxkit, its two reedengine callers and tmuxkit's own dead-socket test switch to it; no production code changes.

**Commit:** 2: Share the datagram stale-socket helper through tmuxkit
