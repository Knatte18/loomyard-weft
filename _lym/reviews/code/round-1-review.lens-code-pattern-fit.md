```yaml
lens: code-pattern-fit
outcome: reported
```

Task: judge the diff's fit with PATTERN.md and CLAUDE.md. Lens: code-pattern-fit.

**Rules checked**

- `PATTERN-shuttle-provider-seam`: held. Claude's hook JSON (`hookContext`, `ToolBoundaryContext`) sits in `claudeengine`. Only `shuttleengines.ToolBoundaryRenderer` reaches the concrete engine, built with a zero config like `RecordReaders`. `creelcli/hookread.go` imports `shuttleengines`, not `claudeengine`. The optional-capability interface is in `shuttleengine/engine.go`.
- `PATTERN-agent-name`: held for Go. The flag name is formed only by `agentname.FileName`, which `flags.go` uses. The shell hook in `settings.go:122-126` re-forms it. That is a second formation, and it is the only way a POSIX `sh` hook can do it, so it is not a defect in itself. The pin is `TestCreelMailHook*` in `hookcmd_integration_test.go:267-299`, which builds the flag file with `agentname.FileName` and runs the hook under `sh`. That test covers the only drift risk, so I raise no finding.
- `PATTERN-durable-vs-ephemeral-state`: held. `CreelFlagDirName` is a `lymdirs` constant. The flags sit under the never-tracked `.lym/creel`. `claudeengine.CreelFlagDir` joins `lymdirs` constants itself, so the flag path is formed in two places (it and `hubgeom.CreelGeometry`). `hubgeom_test.go:311` asserts the two are equal, so they cannot drift unnoticed.
- `PATTERN-tolerant-records`: held. No new journal record kind. `journalWriter.appended` is an in-memory counter, and the flags are derived state, never journaled.
- `PATTERN-pane-text-injection`: held. The hook types nothing. `typeNotice` still sends through `delivery.Send`, now behind a reservation.
- `PATTERN-leaf-packages`: held. `creelengine` newly imports only `agentname` and `logger`. `agentname` is already imported in `reminder.go`, and `logger` is already used in `server.go`. `hubgeom` and `claudeengine` newly import `lymdirs`. `claudeengine` also newly imports `agentname`.
- `PATTERN-cliwire-sole-wiring` and `PATTERN-config-layers`: held. The `runVerb` resolution moved unchanged into `hubClient` and was not duplicated or widened.
- `PATTERN-refusal-way-forward`: held. The entry covers webster, shed and loom refusals, not creel. The new refusals name a way forward anyway: `ClaimAtBoundary` names `lym creel read`, and the engine refusal in `hookRead` lists the registered engines.
- `PATTERN-shell-guard-table`: held. The hook is a settings command, not a guard or a pane-shell string. It quotes with the existing `shQuote`.
- `PATTERN-spawn-observability`, `PATTERN-daemon-wakeups`, `PATTERN-stencil-ownership`, `PATTERN-lym-temp-root`: not touched. `lym` itself is spawned by Claude, not by `lym`. No wait loop and no stencil changed. No temp path is created.
- `PATTERN-test-speed` and `PATTERN-test-isolation`: I can't judge the `tierN` tags or the `sh` integration test's tier, because the manifest doesn't show the build tags. The handler or the tests lens should confirm them.
- CLAUDE.md "Docs land in the same commit": held. The `doc.go` files of `creelengine`, `creelcli`, `claudeengine`, `shuttleengine`, `shuttleengines` and `agentname` changed, and so did `docs/overview.md` (module table and creel entry). `hubgeom` and `lymdirs` have no `doc.go` mention of creel (grep returned nothing), so there is nothing to update.
- CLAUDE.md markdown rules: held. The added `docs/overview.md` lines are one sentence per line, and the new text uses the repo's terms.
- Declared PATTERN changes: none are declared in the directive's constraints section, and the diff touches neither `PATTERN.md` nor `pattern/`. Nothing to land.
- Commit messages and departures: held. There is one commit, with no `Pattern-Departure:` trailer, and no difference from the answer key needs one.

**Findings**

None.

**Tool calls**
1. `grep` for `CreelFlagDir` across the Go files. My context lacked the callers outside `settings.go`, and I needed to know whether a test ties the two flag-directory formations together.
2. `grep` for "creel" in `hubgeom/doc.go` and `lymdirs/doc.go`. The diff did not show those docs, and I needed to know whether either named creel's geometry and so needed an update.
3. `grep` for `FileName`, `flag name` and `StrandNameEnv` in the `claudeengine` tests. The manifest did not include the test files, and I needed to know whether a test pins the shell-formed flag name to `agentname.FileName`.
