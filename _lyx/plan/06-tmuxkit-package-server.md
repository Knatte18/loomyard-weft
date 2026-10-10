# Card 6 — tmuxkit-package-server

**Create:**
- `plan:internal/testkit/tmuxkit#PackageServer` -> `func PackageServer(t *testing.T, tmux string) string`
- `plan:internal/testkit/tmuxkit#packageServerOnce` -> `var packageServerOnce sync.Once`
- `plan:internal/testkit/tmuxkit#packageServerKey` -> `var packageServerKey string`

**Edit:**
- `internal/reedengine/contract_integration_test.go`
- `internal/testkit/tmuxkit/tmuxkit_integration_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/testkit/tmuxkit#startServer`
- `plan:internal/testkit/tmuxkit#registerKey`

**Prosa:**
- `pattern/PATTERN-test-isolation.md`

**Intent:** D5: one server per package for the tests that name their own sessions.
`PackageServer` mints one key per test binary under `packageServerOnce`, starts it through `startServer` and registers it; it registers no cleanup, because `Main`'s sweep kills every server under the kit's directory.
A test qualifies when it names its own sessions, kills no server and asserts nothing about the server's whole session set; a test that drives reed through a hub keeps the hub's `reedengine.ServerName` key, and a test that runs `Down` or `kill-server`, or compares `list-sessions` to an exact set, keeps `Socket`.
Classified onto `PackageServer`, all in `contract_integration_test.go`: `TestExactSessionTargetsNeverPrefixMatchSiblings`, `TestDisplayMessageDoesNotErrorForAnAbsentSession` and `TestSessionNameRewriteIsSilentAndExactTargetsMissIt`; each kills at most its own sessions and checks its own names by containment.
Every other tmux-tier test keeps its key: `TestMultiplexerContract` and `TestSelvageNeverGetsZeroHeightLayoutCell` compare the session set or run `Down`, and the reedcli smoke tests bring the server down.
`tmuxkit_integration_test.go` gains `TestPackageServer_OneKeyPerBinary`: two calls return the same key, the server answers `list-sessions` on it, and the key is registered.
`tmuxkit`'s budget row rises by one.
`PATTERN-test-isolation.md`'s tmux socket files section gains the `PackageServer` sentence and the classification criterion.

**ImpactSummary:** Three reedengine tests change their key source; no production code changes.

**Verify:** lyx gate test --tags tmux ./internal/testkit/tmuxkit ./internal/reedengine
