# Give lym shuttle run a per-run advisor option

`lym shuttle run` takes a new `--advisor <model>` flag that sets Claude Code's advisor model for that one run.
The advisor trial stopped because no route set the advisor for a single shuttle run.
Heddle's `claude-settings` target and the `binary` wrapper reach every session of a hub, so neither isolates one run.

With the flag, `claudeengine` writes `advisorModel` into the `--settings` document it already writes for every launch.
Without it, the document and the launch are unchanged.
The resume line names the same settings file, so a revived session keeps its advisor.
No lym config key is added; the option is per run only.

## Design decisions

- `shuttleengine.Spec` gains an `Advisor` string.
  The value is provider vocabulary and is passed through as given, so Spec validation does not check it.
- The setting is chosen over a `--advisor` argument to `claude`.
  The settings document is the route the run already uses, and the resume line picks it up for free.
- `codexengine` refuses a non-empty `Advisor` in `validateSpec`, before it writes anything, since Codex has no advisor tool.
  The error names a way forward.
- `buildSettings` takes the advisor model as a new parameter, so every caller and test call site gains an empty argument.
  `settingsDoc` writes the key with `omitempty`, so a run naming no advisor leaves it out.
- The value form and the tmux behaviour were confirmed with one short launch: Claude Code 2.1.322 took the alias `opus` from the settings in a lym tmux run, resolved it to `claude-opus-5-5`, and called the advisor.
  The result is recorded in the `--advisor` help text and in the `claudeengine` package doc.
  The help text also says the advisor must be at least as capable as the session's model.
  How advisor calls show in the usage records is left to the bench.

## Tests and docs

- `TestPrepare_AdvisorReachesSettings` pins `advisorModel` present in `settings.json` with an advisor and absent without one.
- `TestRunCmd_ModelSpec` pins `--advisor` reaching `Spec.Advisor` unchanged and empty when the flag is absent.
- The Codex `TestPrepare_RefusesBeforeWriting` table gains an advisor case.
- `docs/overview.md`, the `claudeengine` and `codexengine` package docs, and the `lym shuttle run` long help describe the option.

## To check by hand

- No regression check ran for this task, and the live launch behind the recorded 2.1.322 result is not repeated by the tests.
  A newer Claude Code may take the setting differently, so re-run one short `lym shuttle run --advisor opus` launch if the Claude Code version has moved.
