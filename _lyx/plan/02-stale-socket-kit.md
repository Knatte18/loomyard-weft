# Card 2 — stale-socket-kit

**Create:**
- `plan:internal/testkit/tmuxkit#BindStaleUnixSocket` -> `func BindStaleUnixSocket(t *testing.T, dir, key string)`
- `internal/testkit/tmuxkit/stalesocket.go`

**Delete:**
- `internal/reedengine#bindStaleUnixSocket`

**Edit:**
- `internal/reedengine#TestRemoveSocketFileOnceGone`
- `internal/reedengine#TestRemoveStaleSocket`
- `internal/testkit/tmuxkit#TestRemoveDeadSocket`
- `internal/testkit/tmuxkit/tmuxkit.go`

**Intent:**
D5: the datagram-built stale socket moves from `reedengine`'s tests into the `tmuxkit` kit, and `tmuxkit`'s dead-socket test uses it in place of a closed listener.

`BindStaleUnixSocket` goes into a new non-test file `internal/testkit/tmuxkit/stalesocket.go` with the body and doc of today's `bindStaleUnixSocket`: it binds a `unixgram` socket at `dir/key` and closes it, leaving a socket file that refuses every stream connection, failing the test through `t.Fatalf` on its own setup only.
A datagram socket never listens, so a child a parallel test forks before its exec cannot make the path answer a `Dial`, which a closed listener's inherited copy can.
The kit already imports `net`, `path/filepath` and `testing`; the file adds no import the Testkit Invariant bans.
`reedengine`'s tests already import `tmuxkit` through their `TestMain`, and `tmuxkit`'s non-test files import nothing from `reedengine`, so no cycle forms.

`reedengine`'s private `bindStaleUnixSocket` is deleted; the "stale socket is removed" rows of `TestRemoveSocketFileOnceGone` and `TestRemoveStaleSocket`, its only references, take `tmuxkit.BindStaleUnixSocket` as their `setup`.

`TestRemoveDeadSocket` builds its dead socket with `BindStaleUnixSocket(t, dir, "dead")` instead of listening and closing, and calls `removeDeadSocket` on it with a zero wait like the other paths, dropping the `deadSocketWait` call and the comment about a parallel test's fork; the closed-listener form was the one remaining instance.
Its three assertions stay: the live socket and the plain file survive and the dead socket is removed.

The package doc in `tmuxkit.go` gains one sentence naming `BindStaleUnixSocket` as the stale socket a test of socket removal builds.

**ImpactSummary:** Test-only: one helper moves from `reedengine`'s tests into the `tmuxkit` kit, and three tests' setups switch to it.
