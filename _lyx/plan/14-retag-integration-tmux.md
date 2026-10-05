# Card 14 — Move tmux-starting integration files to the `tmux` tier

**Edit:**
- `internal/loomcli/integration_driverbootstrap_test.go`
- `internal/pairteardown/teardown_integration_test.go`
- `internal/reedcli/cli_integration_test.go`
- `internal/reedcli/list_integration_test.go`
- `internal/reedcli/naming_integration_test.go`
- `internal/reedcli/unlessname_integration_test.go`
- `internal/reedcli/watchdog_integration_test.go`
- `internal/reedcli/watchdogreap_integration_test.go`
- `internal/reedengine/attachgeometry_integration_test.go`
- `internal/reedengine/contract_integration_test.go`
- `internal/reedengine/emptycmd_integration_test.go`
- `internal/reedengine/ensuresession_integration_test.go`
- `internal/reedengine/launchscript_integration_test.go`
- `internal/reedengine/mouse_boot_integration_test.go`
- `internal/reedengine/naming_integration_test.go`
- `internal/reedengine/nosession_integration_test.go`
- `internal/reedengine/replace_integration_test.go`
- `internal/reedengine/watchdog_integration_test.go`
- `internal/standalonegeom/naming_integration_test.go`
- `internal/standalonegeom/testmain_test.go`
- `internal/testkit/tmuxkit/tmuxkit_integration_test.go`
- `internal/webstercli/cli_integration_test.go`

**Create:**
- `internal/webstercli/cli_tmux_test.go`

**Intent:**
An `integration` file whose tests start a tmux server moves to `tmux` (D6), its platform terms kept (`integration && !windows` becomes `tmux && !windows`).
The target list comes from running `go test -tags integration ./...` with `tmux` removed from `PATH`: every listed file holds a test that skipped for the missing binary or failed in `reed.Up` or a standalone reed bring-up.
Files whose tests failed only on a `tmux has-session` liveness probe (`internal/fabriccli`'s remove tests, `internal/battencli/lifecycle_integration_test.go`) start no server and stay `integration`, which needs the tmux binary but no server.
Re-run that detection on this tree before editing, and add any file it newly names.

A listed file that also holds tests needing no tmux server keeps them under `integration`: move the tmux-starting tests into a new `<name>_tmux_test.go` beside it.
`internal/webstercli/cli_integration_test.go` is one such file: only `TestRunCLIIn_StandalonePreRun_ReachesRunsOwnValidationGate` brings up reed, so it moves to `cli_tmux_test.go` and the rest stays.
A helper that both an `integration` and a `tmux` file in one package use moves to an untagged `_test.go` file when it spawns nothing, else to a file constrained `integration || tmux`, the shared-helper rule recorded after Discussion.
`go vet -tags integration ./...` and `go vet -tags tmux ./...` each pass on their own.

Each touched package keeps exactly one `TestMain` per tag set, and every `TestMain` calls `tmuxkit.Main` (`PATTERN-test-isolation`): the package's `!integration` `TestMain` is the one compiled under `-tags tmux`, so a package whose only `TestMain` is `integration`-tagged gains its `!integration` partner.
Where a moved file spawns git, that `!integration` `TestMain` must call `gitkit.HermeticGitEnv()` too, which an untagged file may: `internal/standalonegeom/testmain_test.go` does not today and gains the call.
`cmd/lyx/hermeticenv_test.go` checks only that the token appears somewhere in the package, so it cannot catch this gap; check each touched package's `!integration` `TestMain` by hand.
`cmd/lyx/tmuxisolation_test.go` and `cmd/lyx/hermeticenv_test.go` must pass unchanged.

Bound: these tests leave `cardGateCommand`'s `-tags integration` run (D6), and run under this plan's verify (`-tags tmux`) and by hand until note `test-tiers` decides gate selection.
No test assertion changes; tests move between files only where a file splits.

**ImpactSummary:** test-only build-constraint changes across reedcli, reedengine, standalonegeom, pairteardown, loomcli, webstercli and tmuxkit; production code untouched.
