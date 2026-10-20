# Opening an agent into reed: engine bypass mode by default, explicit engine, model and effort, and reed add points to shuttle open

`reed-agent-launch-bypass` · Note · bug, reed, shuttle, claude, codex

Imported from [issue #773](https://github.com/Knatte18/loomyard/issues/773).

Seen on OSL-1046 (2026-10-20): `lyx reed add --help` gives the example `lyx reed add --cmd "claude --session-id %SID%" --role producer`. Following it launches `claude --session-id %SID%` verbatim (see the strand's .lyx/reed/launch/<guid>.sh), and claude exits with "Invalid session ID. Must be a valid UUID.". `%SID%` appears only in the help text in internal/reedcli/add.go; nothing substitutes it.

Workaround: generate a UUID (/proc/sys/kernel/random/uuid) and pass it literally in --cmd and --resume-cmd.

Expected: either reed substitutes a generated session id for %SID% in --cmd and --resume-cmd, or the help example shows a command that works as written.

## Issue comment, 2026-10-20

Correction to the framing above: an operator should never have to build an agent command line for this.
The command for it exists: `lyx shuttle open --role <name>` opens an interactive LLM session in its own window, under its strand name.
The orch reached for `lyx reed add` instead, following its help example, and got neither a working session id nor bypass permissions.

Gaps that let this happen:

- `lyx reed add --help` shows a claude command line as its example and does not point to `lyx shuttle open` for an agent session.
- Bypass permissions is opt-in: `lyx shuttle open` without `--permission-mode` keeps the engine's interactive default.
  Every agent instance that runs in reed needs bypass permissions to work correctly (operator, 2026-10-20), so no caller should have to pass it.

Expected: every agent session lyx launches into reed starts with its engine's bypass mode without a flag, `lyx shuttle open --role <name>` included, and `reed add`'s help sends an agent launch to `lyx shuttle open`.
This is per engine, not claude-only: claude takes `--permission-mode bypassPermissions`, and when Codex arrives as an engine it gets its own equivalent, which the operator understands to be `--yolo`.

## Operator ask (2026-10-20): open the agent you want

With both claude and codex as engines, opening an operator agent must let the operator choose the engine, the model and the effort level, and show what was chosen; `lyx shuttle open` today is too much a matter of chance in what one gets.

What exists: `--model` takes a model spec (`internal/modelspec`), either an alias with items, `<alias>[<effort>]`, or the escape form `<engine>:<model-id>[<effort>]`, so engine, model and effort can already be named.
What is missing:

- None of this is discoverable from `lyx shuttle open --help`, which says only "model spec; empty defers to the provider default", with no grammar, no aliases and no effort.
- Without `--model` the engine and model are whatever the provider defaults to, and the command's output does not say which engine, model and effort the session got.
- A way to list the choices, the registry's aliases with their engine, model and default effort, from the command itself.