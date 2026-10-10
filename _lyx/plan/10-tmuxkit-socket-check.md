# Card 10 — tmuxkit-socket-check

**Create:**
- `plan:internal/testkit/tmuxkit#checkSockets` -> `func checkSockets(dir string, allowed bool) []string`
- `plan:internal/testkit/tmuxkit#tmuxAllowed` -> `const tmuxAllowed = true`
- `internal/testkit/tmuxkit/allow_tmux.go`
- `internal/testkit/tmuxkit/allow_none.go`

**Edit:**
- `plan:internal/testkit/tmuxkit#afterRun`
- `internal/testkit/tmuxkit/tmuxkit_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/testkit/tmuxkit#registeredKeys`

**Prosa:**
- `pattern/PATTERN-test-isolation.md`
- `pattern/PATTERN-test-speed.md`

**Intent:** D6's runtime half: tier 1 and tier 2 are tmux-free.
`allow_tmux.go` (`//go:build tmux || llm`) declares `tmuxAllowed = true` and `allow_none.go` (`//go:build !tmux && !llm`) declares it `false`, so the allowance is a property of the test binary's tags.
`checkSockets` lists the socket files under `dir`'s per-user tmux directory; with `allowed` false every socket is a finding, and with it true only a socket whose key is not in `registeredKeys`.
`afterRun` runs it after the sweep, before the leftover scan, with `tmuxAllowed`, and a non-empty result fails the package with exit 1, naming each socket and the way forward: tag the file `tmux`, or register the key through `Socket`, `KillOnCleanup` or `PackageServer`.
Tests in `tmuxkit_test.go`, untagged and without tmux: `TestCheckSockets` listens on a Unix socket at `<dir>/tmux-<uid>/<key>` through `net.Listen` and asserts the key is named with `allowed` false, is named with `allowed` true while unregistered, and is clean once `registerKey` has it; a second row with no socket is clean under both.
`tmuxkit`'s budget row rises by one.
`PATTERN-test-speed.md`'s tier table says tier 2 has no tmux server and the gate table names the end-of-package check; `PATTERN-test-isolation.md`'s tmux isolation section records the check and its bounds: a test that points `TMUX_TMPDIR` elsewhere, a server started by a binary other than tmux, and a package with only untagged files, which never calls `Main`.

**ImpactSummary:** `afterRun` gains a check every tagged package runs through; the integration tier of any package that still starts a server fails here, which the Verify line surfaces for the packages that drive reed.

**Verify:** lyx gate test --tags integration ./internal/loomcli ./internal/reedcli ./internal/reedengine ./internal/shedcli ./internal/webstercli ./internal/orchcli ./internal/burlerengine ./internal/landingshed
