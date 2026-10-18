# Fast batten start, local-only fabric pairs and one Command producer

`batten-start-speed` · Task · enhancement, batten, fabric, shed, config, loom · running · Publish

## Seen: batten's start (2026-10-17/18)

- Batten's start of `creel-notices` (row times from `lyx batten status creel-notices`): Worktree-Create done 13:15:13, Seed-Child about 1 s, Open-Window about 8 s, Seed-IDE about 2 s, Open-IDE about 2 s; about 10 to 12 s before Run-Shed, as the operator sees it.
- Open-IDE is off in the hub and does nothing, yet its row took about 2 s, as did Seed-IDE.
- Likely cause, read from the code, not measured: after every transition batten's `CommitStatus` seam (`internal/battencli/commitstatus.go`) commits the prime's batten `status.json` and then pushes it to GitHub synchronously (`fabricengine.PushAnchored`, waiting up to `StatusPushLockWait`, 30 s, for the push lock); one push costs about 1.5 to 2 s, and a push waiting on another push's lock costs more.
- Open-Window does not wait for the window: `OpenAttach` in `internal/termwindow/termwindow.go` returns after `cmd.Start()`, and opening Konsole takes milliseconds.
- `lyx fabric add` in the sandbox hub (5-file repo) took about 1.1 s, 92% of it two serial `git ls-remote` origin probes; `lyx fabric remove --remote` took 6 to 7 s, almost all network (force-with-lease push, archive-tag push, weft branch delete). `internal/fabricengine/add.go` has `SkipGit`/`SkipPush` for tests only; `internal/fabricengine/remove.go` always pushes an archive tag when the weft has an origin.

## Seen: the side-action rows (orch, 2026-10-17; was note `command-producer`)

Batten's Open-Window, Seed-IDE and Open-IDE rows share one Go type, `SwitchedProducer` in `internal/shedadapters`, but each is its own registry engine (`OpenWindow`, `SeedIDE`, `OpenIDE`) wrapping a told Go action behind a told `batten.yaml` flag.
A new side action therefore needs a new engine, a new Go action and a new flag.
Other places already run a shell command from config inside a bigger row: `landing.yaml`'s `deploy` (Land) and `publish_verify` (Publish), and darn's `verify`.
Note `regression-baseline-verify` adds `Job-Start` and `Job-Join`, which would be two more such rows.

### Operator view, not a decision (2026-10-17)

All these special rows should be one and the same producer whose only job is to run a CLI command; the flag and the command to run can live somewhere else entirely, in config.

### Orch recommendation, not a decision

- One registry engine, `Command`, used by every such row. The row names a config key; the key holds the command, with placeholders such as `{slug}` and `{pair}`. An empty command is the off switch, so no separate flag is needed.
- Per row, in the recipe: the failure policy (`warn`, as the IDE rows do today, or `halt`) and the mode (`sync`, or `async`, which starts the command detached and records its handle).
- A `Join` form of the same engine waits for an `async` row's handle and applies its failure policy; this is `Job-Join`.
- Each action that is Go code today gets a `lyx` verb (for example the terminal window and the IDE seed), so the default config spells it as a command.
- Later candidates: `deploy` and `publish_verify` as their own `Command` rows instead of commands buried in Land and Publish.

### Open

- Whether a task's `run_settings` may set the command, or only switch it on and off; a command from a board entry runs on the machine, so it may need the same trust class as `deploy`.
- Placeholders: which run facts a command may name.

## Operator view, not a decision (2026-10-18)

Every such push goes to the background, async: fabric is built for exactly this (its detached push, as `lyx fabric add` already uses through `SpawnPairPush`).
Loom's own per-transition status push (`internal/loomcli/wiring.go`, `fabricengine.PushPairAnchored`) is synchronous too, and so is the push in `internal/battencli/cleanup.go`.

## Scope

1. Measure where batten's start goes, row by row and inside each row, and cut the overhead so a row that does nothing costs milliseconds and Open-Window costs no more than starting the window; nothing needs batten's status on GitHub before the next row runs, so every per-transition status push (batten, loom and any other shed run's seam) becomes fabric's detached push.
2. Local-only pairs in fabric: an add with no origin probes and a remove with no push, archive tag or remote delete, for pairs that never reach origin (the batch pairs `weaver` and `parallel-batch-parts` describe).
3. One `Command` producer for the flag-gated side-action rows, as above.

Out: Weaver itself; `Job-Start`/`Job-Join` and moving `deploy`/`publish_verify` into rows of their own.

## Related

Notes `regression-baseline-verify`, `gate-language-neutral`, `weaver`, `parallel-batch-parts`.
