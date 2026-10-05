# Card 17 — tmuxkit removes the socket files of the servers it kills

**Create:**
- `plan:internal/testkit/tmuxkit#KillOnCleanup` -> `func KillOnCleanup(t *testing.T, tmux, key string)`

**Edit:**
- `internal/testkit/tmuxkit#Socket`
- `internal/testkit/tmuxkit#sweep`
- `internal/testkit/tmuxkit/tmuxkit_integration_test.go`
- `internal/testkit/tmuxkit/tmuxkit_test.go`
- `PATTERN.md`
- `pattern/PATTERN-test-isolation.md`

**Intent:**
First, survey the leak (D9): snapshot `/tmp/tmux-$UID/`, run `go test -tags integration,tmux -count=1 ./...` (every tier but `llm`, after cards 13 and 14), and list the socket files the run added, grouped by prefix.
Record the list in this card's commit message; card 18 fixes each source it names that this card does not.

`KillOnCleanup` registers, in `t.Cleanup`, a `kill-server` on the `-L` key `key` and then the removal of that key's socket file under the current `TMUX_TMPDIR`'s per-user directory.
It removes only the one path for its key, never globs or sweeps the directory, and only when the path is a socket and a connection to it fails after `kill-server` ran, so it never touches a live server, including one another test owns under a different key.
It resolves the directory from `TMUX_TMPDIR` as tmux does, so a test outside `tmuxkit.Main`'s isolation reaches only its own key's file in the default directory.
It is the one helper for a key a test did not mint itself, such as a `reedengine.ServerName` key of a fixture hub;
`Socket` mints its random key as today and delegates its cleanup to `KillOnCleanup`.
`sweep` removes each socket file after killing its server, so `Main`'s temp directory holds no socket when it is removed.
`KillOnCleanup` is a tmuxkit export other than `Main`, so `cmd/lyx/tmuxkitspawn_test.go` already counts it as a tmux spawn barred from untagged files.

Tests extend the existing ones rather than adding new ones: `TestSocket_KillsServerAtCleanup` also asserts the socket file is gone after the owning subtest's cleanup, a row drives `KillOnCleanup` directly with a key the test chose, and `TestSweep_KillsServersUnderItsDirectoryOnly` asserts the swept socket files are removed and a socket outside the directory is untouched.
`tmuxkit_test.go` gains rows only where the socket-path derivation it already tests changes.

`PATTERN.md`'s `PATTERN-test-isolation` line and its background gain the socket rule (D8, D9): a test that starts a tmux server takes its key from `tmuxkit` (`Socket`, or `KillOnCleanup` for a key it did not mint), which removes the socket file after `kill-server`, and a test that builds a subprocess environment from scratch passes `TMUX_TMPDIR` through.
Bound: socket files already in `/tmp/tmux-$UID/` from earlier runs are not cleaned (D9).

**ImpactSummary:** tmuxkit's `Socket` keeps its signature and gains socket-file removal; `sweep` is unexported; callers of `Socket` see only cleaner teardown.
