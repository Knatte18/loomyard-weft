```yaml
lens: code-affected-set
outcome: reported
```

List per change (set found → covered or not):

- **env module + configreg + docs:** set is `configreg.Modules`, the configreg and strictness tests, `docs/overview.md`, and PATTERN-config-layers (the machine category and the degrading-callers list). Covered, with no stale module list found.
- **configengine and yamlengine behavior:** set is the `ValidateLocal`, `ValidateMachine` and `templateValueAt` doc comments, the `configengine` `doc.go`, the `Strip` doc and `TestStrip`. Covered.
- **Gate env precedence (`runTest`):** set is `gatecli/doc.go` and the tier, build and `green.go` consumers of `exe.env`. Covered, but the verify-command paths below are missed.
- **Drift `StepWithHubEnv`:** set is every `cfg.Step(` caller. Covered. The remaining `Step` callers (`baseline.go:185`, `verify.go:173`, `driftcli/cli.go:322`) only validate the step and never run it.
- **Reed `PaneEnv`:** set is `Geometry`, `hubgeom`, reed's `doc.go` and `spawn.go`. Covered.
- **Declared PATTERN changes and commit messages:** the directive declares no PATTERN changes, and no `Pattern-Departure:` trailers appear in the range. Nothing raised.

Findings:

1. **MEDIUM, class `scope`.** Other hub verify and test runs get no env-module variables. `internal/verifytree/verify.go:388` (the verify run the PATTERN-gate-slot entry names) and `internal/cardverify/runner.go:63` (a card's `**Verify:**` line) build their environment from `os.Environ()` with no `envconfig`. `internal/battencli/land.go:182` and `:208` do the same through `commandEnv(os.Environ())`. These paths run the hub's tests, as the tier runs do.
   - The issue says lym exports the variables to "every process it starts for the hub", and its example is `COMMONLIB_REPO` for tests.
   - The summary and the `envconfig` `doc.go` list only gate tiers and the shared build, Drift, and panes.
   - A verify or card command that reads `COMMONLIB_REPO` therefore fails where the same tier passes.
   - The fix is either to route these runs through the same variables or to record the narrower scope. The latter is the parent's call.
   - The handler should confirm this against the design's closed enumeration of processes.

2. **LOW, class `consistency`.** Gate precedence. `gatecli/run.go:136-138` appends the module's variables after `withBuildServerOff`, so an env-module `MSBUILD*` name replaces the build-server-off switches. The `doc.go` sentence "plus `proc.BuildServerOffEnv`, replacing an inherited value, plus the variables of the `env` config module" states this order. It contradicts the stated invariant that no build leaves MSBuild servers behind. The module refuses only `LYM_` names. This is minor and arguably intended precedence.

Tool calls (2 of 6):
1. `grep` for the `.yaml` file names of config modules across the repo, to find docs and code that list modules. My context lacked a repo-wide search for these.
2. `grep` in `docs/overview.md` and `orch-role.md`, plus a stencil search for tier `env` mentions, to check for a stale module list. My context lacked both files.
