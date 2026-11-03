# Fixer-report, code round 1

Every finding is fixed; one self-check finding is disputed.
Gate: `lym gate test --tier 1` over the changed packages and `--tier 0` over the unit packages ended green, and `--tier 0` over `reedengine` and `envconfig` ended green again after the self-check fix.

## Changes

- F1 (`gatecli`): a row in the gate scenario runs the shared build with an `env.yaml` in the hub, and asserts the module's `GOFLAGS` over the inherited one and the build's own `BUILD_MARK` over the module's.
  The same row asserts the build-server switch stays lym's over a module value, which pins F9.
- F2 (`gatecli`): a row in the gate scenario runs inside the hub fixture with the hub file alone, then with this machine's file added, and asserts the tier sees the machine value.
- F3 (`driftcli`, `loomcli`, `driftengine`): the two existing wiring tests write an `env.yaml` and a step `env`, and assert each command of the three sites, `takeRun`, `runCaptureChild` and `driftDeps`, ran with the module's variable and the step's own value winning a shared name.
  The `StepWithHubEnv` subtest gains a row for an env config that fails to load, asserting the error and no step.
- F4 (`reedengine`): a row in `TestLaunchStrandLocked_LaunchScript` tells `PaneEnv` and asserts the staged script holds the exports ahead of the command.
  Fixed differently for the dialect clause of the proposal: the row runs on the host's dialect only, since `launchStrandLocked` builds its line with `shell.ForGOOS()` and a test cannot reach the other dialect through it.
  `TestPaneEnvExports` already pins both dialects' exports.
- F5 (`gatecli`): a row in the gate scenario writes `BAD-NAME` to `env.yaml`, and asserts exit 1, a refusal naming the env config, `vars.BAD-NAME` and the way forward, and no tier run.
- F6 (`hubgeom`): a step in `TestGeometryOverRealHub` asserts the hub's reed geometry carries a `PaneEnv` returning the hub file's values with the machine file's value winning.
- F7 (`envconfig`): `LoadConfig` refuses `PATH`, named by the new `reservedPath` constant, with the `ErrInvalid` message shape and way forward; the `LoadConfig` comment, `doc.go` and `template.yaml` state it; rows for `PATH` and `Path` are in `TestLoadConfig_StacksHubAndMachineName`.
- F8 (`envconfig`): the `LYM_` comparison runs on the upper-cased name; a `lym_parent` row is in the same test.
- F9 (`gatecli`): `runTest` appends the module's entries before `withBuildServerOff`, and `gatecli/doc.go` says the module never undoes a build-server switch.
- Self-check fix (`reedengine`): `paneEnvExports` drops the names of `proc.BuildServerOffEnv`, a row in `TestPaneEnvExports` pins it, and `reedengine/doc.go` and `envconfig/doc.go` state it.

Commits: one per finding group, `envconfig` (F7, F8), `gatecli` (F9), `gatecli` (F1, F2, F5), `driftcli` (F3), `reedengine` (F4), `hubgeom` (F6), then `reedengine` for the self-check fix.

## Self-check

- fix-consistency, MEDIUM: strand panes export the module's variables after the build-server switches, so a hub value undoes them there; fixed with the change above, dropping those names in `paneEnvExports`.
- fix-affected-set, LOW: the same finding as the consistency one, merged; fixed.
- fix-consistency, LOW: `envconfig/doc.go` precedence paragraph said nothing of the build-server switches; fixed with one sentence.
- fix-proposal, LOW: the case-insensitive refusal also refuses `Path` and `lym_cache` on Linux; disputed, see below.
- fix-code-correctness: no findings.

## Disputed

- Ground `every fix worse`, self-check finding of the fix-proposal lens on the F7 and F8 check: the premise holds, since `Path` and `lym_cache` are legal names on Linux.
  The only other change, matching case-insensitively on Windows alone, makes the target worse: the hub file is shared by every machine (`internal/envconfig/doc.go`: "The hub file in the board dir holds the values every machine shares"), so a hub file that loads on Linux would be refused on Windows.
  One rule on every platform keeps a shared file valid everywhere.

## Deferred

None.
