# Fixer-report, code review round 1

F1 is `decision`-class and went to the parent, so it is left out of this report.
Each other finding is fixed as proposed and committed on its own; tier 0 of `lym gate test` passed for `claudeengine`, `codexengine` and `creelcli` before the commits.

## Changes

- **F2** (LOW), commit `43559ea6`: the `hookWaitVerb` comment in `internal/creelcli/hookwait.go` and the `hookReadVerb` comment in `internal/creelcli/hookread.go` now say the verb is the one a strand's engine hooks spell, naming no engine.
- **F3** (LOW), commit `e3c23588`: `TestPrepare_CreelHooksInAHub` in `internal/shuttleengine/codexengine/prepare_test.go` writes `binary: /opt/codex/bin/codex-custom` into the engine's `codex.yaml` and expects it as the `bin` of the `creelDelivery` it builds, so a `Prepare` that hard-coded `codex` fails.
  The test's doc comment says the waiter uses the executable `codex.yaml` names.
- **F4** (LOW), commit `1d32b2a5`: in `internal/shuttleengine/claudeengine/settings.go`, the `strandFlagName` wrapper and the `creelTrustSentence` constant are deleted, and their uses in `settings.go`, `settings_test.go` and `prepare_test.go` call `shuttleengine.StrandFlagNameShell()` and `shuttleengine.CreelTrustSentence` directly.

## Self-check

All four forks returned a report.

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

Nothing disputed.

## Deferred

Nothing deferred.
