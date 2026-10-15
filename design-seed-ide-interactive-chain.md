# batten Seed-IDE writes the interactive VS Code chain into a driven pair, so opening it in VS Code starts a bare claude

`seed-ide-interactive-chain` · Note · bug, ide, batten

Imported from [issue #633](https://github.com/Knatte18/loomyard/issues/633).

Seen (2026-10-15): in the running pair shed-tackler, the reed shell window held a bare interactive claude (no prompt, no role) that no agent or operator started. Its bash and claude started at 09:41:39, the same second VS Code's extension host ran 'lyx reed up' and 'lyx reed attach' on the pair: the pair's .vscode/tasks.json, written by batten's Seed-IDE row, carries the folderOpen 'Start Claude' chain (reed up, reed add --if-absent --unless-name orch --cmd claude --name claude --focus, reed attach). Cause: ideengine.SeedTask calls writeInteractiveConfig, which writes vscode.TaskChainInteractive; vscode.TaskChainAttachOnly, documented as the driven pair's chain, has no caller. Expected: Seed-IDE writes the attach-only chain, so opening a driven pair in VS Code attaches to the run and adds no claude strand. Workaround: none applied; the stray claude sits idle in the pair's session.

## Operator decision (2026-10-15)

The folderOpen task chain goes out of the VS Code seeding: it is no longer needed, since runs are watched in their own terminal window, and above all `reed add claude` must not run from inside VS Code.
Open for the task's scope: whether this covers only batten's Seed-IDE or also `lyx ide spawn`, which writes the same interactive chain (`internal/ideengine/spawn.go`).