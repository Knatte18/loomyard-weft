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

**Intent:**
Decision D5: the datagram-built stale socket becomes one shared helper in the `tmuxkit` test kit, per `PATTERN-testkit`.

`BindStaleUnixSocket` moves the body and doc comment of `reedengine`'s private `bindStaleUnixSocket` unchanged into the new non-test file `internal/testkit/tmuxkit/stalesocket.go`.
It binds a `unixgram` socket at `dir/key` and closes it, leaving a socket file that refuses every stream connection, so a child forked by a parallel test cannot make the path answer a `Dial`.
It calls `t.Fatalf` on its own setup failure only, as the kit rules allow.
`tmuxkit` imports no `reedengine` code, and `reedengine`'s untagged `testmain_test.go` already imports `tmuxkit`, so the import adds no cycle and no new tier.

`reedengine`'s socket tests import the helper: the `setup` rows "stale socket is removed" in `TestRemoveSocketFileOnceGone` and `TestRemoveStaleSocket` name `tmuxkit.BindStaleUnixSocket`, and the private copy in `internal/reedengine/socketfile_test.go` is deleted.

`TestRemoveDeadSocket` builds its dead socket with `BindStaleUnixSocket(t, dir, "dead")` in place of listening and closing a listener.
It then removes the dead socket with `removeDeadSocket(dead, 0)` like the live, plain and absent paths, and drops the comment and the `deadSocketWait` argument it carried for fork inheritance.
`removeDeadSocket` removes any socket whose `net.Dial("unix", …)` fails, and a stream dial to a datagram socket fails, so the test still asserts the dead socket is gone, the live socket and the plain file stay.
`deadSocketWait` itself stays: the kit's production cleanup still passes it for a killed server.

**ImpactSummary:** Test-only: one helper moves from `reedengine`'s tests into the `tmuxkit` kit, two `reedengine` test rows and one `tmuxkit` test switch to it; no production code changes and no new top-level test.
