# Card 2 — One stale-socket helper in a test kit

**Create:**
- `internal/testkit/socketkit/doc.go`
- `plan:internal/testkit/socketkit#BindStaleUnixSocket` -> `func BindStaleUnixSocket(t *testing.T, dir, key string)`

**Delete:**
- `internal/reedengine#bindStaleUnixSocket`

**Edit:**
- `internal/reedengine#TestRemoveSocketFileOnceGone`
- `internal/reedengine#TestRemoveStaleSocket`
- `internal/testkit/tmuxkit#TestRemoveDeadSocket`
- `docs/overview.md`

**Intent:** D5, under `PATTERN-testkit`.
Create the kit package `internal/testkit/socketkit`: `doc.go` holds the package doc (shared unix-socket test fixtures, imports the standard library only, asserts nothing beyond `t.Fatalf` on its own setup), and `socketkit.go` holds `BindStaleUnixSocket`, moved verbatim from `reedengine`'s `bindStaleUnixSocket` with its doc comment: it binds a `unixgram` socket at `dir/key` and closes it, leaving a socket file that refuses every stream connection, so a child forked by a parallel test cannot make the path answer a `Dial` the way an inherited closed listener can.
The kit imports only `net`, `path/filepath` and `testing`.
It cannot live in `tmuxkit`: every `tmuxkit` export but `Main` counts as a tmux spawn (`cmd/lyx/tmuxkitspawn_test.go`), and `reedengine`'s `socketfile_test.go` is untagged.

Delete `bindStaleUnixSocket` from `internal/reedengine/socketfile_test.go`; its only callers are the `stale socket is removed` rows of `TestRemoveSocketFileOnceGone` and `TestRemoveStaleSocket`, whose `setup` becomes `socketkit.BindStaleUnixSocket`.

In `TestRemoveDeadSocket` (`internal/testkit/tmuxkit/tmuxkit_test.go`, package `tmuxkit`, untagged) the dead socket is built with `socketkit.BindStaleUnixSocket(t, dir, "dead")` instead of listening and closing a listener, and the separate `removeDeadSocket(dead, deadSocketWait)` call and its fork-inheritance comment go: the dead path joins the loop that calls `removeDeadSocket(path, 0)`.
The test's assertions stay: the live socket and the plain file survive, the dead socket is removed, so the test fails if the helper's socket accepted connections.
`deadSocketWait` stays, since `killServer` still uses it.

In `docs/overview.md`, beside the `internal/testkit/tmuxkit` line of the kit list, add one line naming `internal/testkit/socketkit` as the stale unix-socket fixture `reedengine`'s and `tmuxkit`'s socket tests share.

No new top-level test, so no test-budget row changes; the new kit has no tests of its own.

**ImpactSummary:** Test-only: one helper moves from reedengine's tests into a new stdlib-only kit, three existing tests switch to it, and no production code changes.

**Commit:** 2: testkit: share the datagram stale-socket helper through socketkit
