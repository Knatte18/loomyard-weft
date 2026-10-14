# Card 1 — stale-socket-kit

**Create:**
- `internal/testkit/socketkit/socketkit.go`
- `plan:internal/testkit/socketkit#BindStaleUnixSocket` -> `func BindStaleUnixSocket(t *testing.T, dir, key string)`

**Edit:**
- `internal/reedengine#TestRemoveSocketFileOnceGone`
- `internal/reedengine#TestRemoveStaleSocket`
- `internal/testkit/tmuxkit#TestRemoveDeadSocket`

**Delete:**
- `internal/reedengine#bindStaleUnixSocket`

**Intent:** D5: the datagram-built stale socket moves from `reedengine`'s tests into a new test kit, per `PATTERN-testkit`, and `tmuxkit`'s dead-socket test uses it in place of a closed listener.

Create `internal/testkit/socketkit/socketkit.go`, package `socketkit`, whose package doc comment says it holds the stale unix socket file the reed and tmux kit tests share, imports only the standard library (`net`, `path/filepath`, `testing`), and is imported only from tests.
`BindStaleUnixSocket` is the body of today's `bindStaleUnixSocket` in `internal/reedengine/socketfile_test.go`, moved verbatim with its doc comment: it binds a `unixgram` socket at `dir/key` and closes it, leaving a socket file that refuses every stream connection, and a child a parallel test forks before its exec cannot make it answer a `Dial`, as a closed listener's inherited copy would.
It fails the test only through `t.Fatalf` on its own bind or close, as `PATTERN-testkit` allows a kit.

Delete `bindStaleUnixSocket` from `internal/reedengine/socketfile_test.go`, and point the `setup` field of the "stale socket is removed" rows of `TestRemoveSocketFileOnceGone` and `TestRemoveStaleSocket` at `socketkit.BindStaleUnixSocket`, importing the kit.
Their row tables and assertions are otherwise unchanged.

In `TestRemoveDeadSocket` (`internal/testkit/tmuxkit/tmuxkit_test.go`), build the dead socket with `socketkit.BindStaleUnixSocket(t, dir, "dead")` instead of listening and closing `deadListener`, and remove it with `removeDeadSocket(dead, 0)` alongside the live, plain and absent paths, dropping the `deadSocketWait` argument and the comment about a parallel test's fork holding a copy of the closed listener.
The `listen` helper keeps building the live socket.
The test still asserts the live socket and the plain file survive and the dead socket is removed, so it fails if `removeDeadSocket` stops removing a socket that refuses connections with no wait.
`deadSocketWait` stays in `tmuxkit.go`, where `killServer` still waits out a killed server.

`socketkit` has no test of its own, so it needs no row in `cmd/lyx/testdata/test-budget.yaml`; neither package's top-level test count changes.

**ImpactSummary:** test-only: two reedengine socket tests and one tmuxkit test switch to the shared kit helper, and the reedengine-private copy goes.

**Commit:** 1: Move the stale-socket test helper into socketkit and use it in tmuxkit
