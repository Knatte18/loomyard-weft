# One generic Command producer for every flag-gated side-action row

`command-producer` · Note · enhancement, shed, batten, config

## Seen (orch, 2026-10-17)

Batten's Open-Window, Seed-IDE and Open-IDE rows share one Go type, `SwitchedProducer` in `internal/shedadapters`, but each is its own registry engine (`OpenWindow`, `SeedIDE`, `OpenIDE`) wrapping a told Go action behind a told `batten.yaml` flag.
A new side action therefore needs a new engine, a new Go action and a new flag.
Other places already run a shell command from config inside a bigger row: `landing.yaml`'s `deploy` (Land) and `publish_verify` (Publish), and darn's `verify`.
Note `regression-baseline-verify` adds `Job-Start` and `Job-Join`, which would be two more such rows.

## Operator view, not a decision (2026-10-17)

All these special rows should be one and the same producer whose only job is to run a CLI command; the flag and the command to run can live somewhere else entirely, in config.

## Orch recommendation, not a decision

- One registry engine, `Command`, used by every such row. The row names a config key; the key holds the command, with placeholders such as `{slug}` and `{pair}`. An empty command is the off switch, so no separate flag is needed.
- Per row, in the recipe: the failure policy (`warn`, as the IDE rows do today, or `halt`) and the mode (`sync`, or `async`, which starts the command detached and records its handle).
- A `Join` form of the same engine waits for an `async` row's handle and applies its failure policy; this is `Job-Join`.
- Each action that is Go code today gets a `lyx` verb (for example the terminal window and the IDE seed), so the default config spells it as a command.
- Later candidates: `deploy` and `publish_verify` as their own `Command` rows instead of commands buried in Land and Publish.

## Open

- Whether a task's `run_settings` may set the command, or only switch it on and off; a command from a board entry runs on the machine, so it may need the same trust class as `deploy`.
- Placeholders: which run facts a command may name.

## Related

Notes `regression-baseline-verify`, `gate-language-neutral`.
