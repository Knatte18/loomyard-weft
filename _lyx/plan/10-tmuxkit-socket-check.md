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
The per-user directory `tmux-<uid>` is never a finding by itself: tmux creates it on any invocation that names a `-L` key, a client probe of a server that never started included, and tier-2 tests reach such probes through production, as `fabriccli`'s and `battencli`'s removals do through `pairteardown`'s `Engine.Status` and `Engine.Down`; a client probe starts no server, which is all D6 bars.
Bound, under any tag: reed's own teardown removes its server's socket file, so a server that its test booted and reed tore down within the run leaves no socket and escapes the check, unregistered or in tier 2 alike.
The sweep kills every server under the directory and removes its socket file, so the check must see the directory before the sweep does: `afterRun`, which card 7 handed the sweep as a parameter, now runs, in order, `checkSockets` with `tmuxAllowed`, keeping its result, then `sweep`, then the leftover scan; a non-empty check result fails the package with exit 1, naming each socket and the way forward: tag the file `tmux`, or register the key through `Socket`, `KillOnCleanup` or `PackageServer`.
`Main` passes the real sweep; a test passes a stand-in that removes every socket file under the directory, as the real sweep leaves it.
Tests in `tmuxkit_test.go`, untagged and without tmux: `TestCheckSockets` listens on a Unix socket at `<dir>/tmux-<uid>/<key>` through `net.Listen` and asserts the key is named with `allowed` false, is named with `allowed` true while unregistered, and is clean once `registerKey` has it; a second row with no socket and no per-user directory is clean under both; a row with the per-user directory present and no socket in it, as a client probe leaves it, is clean under both; a further row drives `afterRun` over the listening unregistered socket with the removing stand-in sweep and asserts exit 1 naming the key although the sweep removed the file, which pins the order.
`tmuxkit`'s budget row rises by one.
`PATTERN-test-speed.md`'s tier table says tier 2 has no tmux server and the gate table names the end-of-package check; `PATTERN-test-isolation.md`'s tmux isolation section records the check, that a client probe's per-user directory is no finding, and its bounds: a test that points `TMUX_TMPDIR` elsewhere, a server started by a binary other than tmux, a package with only untagged files, which never calls `Main`, and, under any tag, a server that reed tore down itself within the run.

**ImpactSummary:** `afterRun` gains a check every tagged package runs through; the integration tier of any package that still starts a server fails here, which the Verify line surfaces for the packages that drive reed.

**Verify:** lyx gate test --tags integration ./internal/loomcli ./internal/reedcli ./internal/reedengine ./internal/shedcli ./internal/webstercli ./internal/orchcli ./internal/burlerengine ./internal/landingshed
lyx gate test --tags tmux ./internal/loomcli ./internal/reedcli ./internal/reedengine ./internal/webstercli ./internal/pairteardown ./internal/standalonegeom ./internal/testkit/tmuxkit
lyx gate test --tags llm ./internal/orchcli ./internal/burlerengine ./internal/seatengine ./internal/shuttlecli ./internal/treadleengine -- -run '^$'
