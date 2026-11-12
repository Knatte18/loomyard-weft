# Give lym shuttle run a per-run advisor option

`shuttle-run-advisor-option` · Task · enhancement, shuttle, claude · running · Drift-Recheck

## Seen (operator, 2026-11-12)

The advisor trial (`docs/research/loom/advisor-model-trial-2026-11-11/report.md`) stopped because no route sets Claude Code's advisor for one shuttle run.
`claudeengine` writes a `--settings` document for every launch (`internal/shuttleengine/claudeengine/settings.go`), but it carries hooks and scope only, and `lym shuttle run` has no flag that adds a key to it.
Heddle's `claude-settings` target and the `binary` wrapper reach every session of a hub, so neither isolates one run.

## Wanted

`lym shuttle run` takes an optional advisor model for that run alone, and `claudeengine` writes it into the `--settings` document it already writes (`advisorModel`), or passes `--advisor`, whichever Claude Code accepts.
No advisor flag leaves every launch as it is today.
No lym config key is added; the option is per run only.

## Check first

The trial did not confirm the setting's value form (a bare model name or a full model spec), that it takes effect in a lym tmux session, or how advisor calls show in the usage records.
The task confirms the first two with one short launch and records the answer in the option's help text and in the docs of the shuttle module.
How advisor calls show in the usage records is for the bench to read, not this task.

## Not in scope

A config key for the advisor beside each role's model and effort; that follows only after the bench shows it pays.
The advisor prompt, and the bench itself.
