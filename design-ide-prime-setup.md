# lyx seeds the prime VS Code setup: title, orch start task, keybindings

`ide-prime-setup` · Notes · feature

Operator 2026-10-04: the prime's `.vscode/` was copied by hand from millhouse (title "LYXHUB", millhouse keys, a mill `launch.json`); lyx should seed it instead, and the setup should fit the orchestrator workflow.
Assume no `.vscode/` exists beforehand, or that it may be deleted and re-seeded by lyx.

- Window title: `<SHORTNAME>: <name>` from `.lyx-shortname`, upper-cased: `LY: prime` for the prime, `LY: <slug>` for pairs (today `internal/vscode/config.go` writes the bare slug).
- Prime folderOpen chain: `reed up` then `reed attach` only; no `reed add claude` (it adds a plain Claude strand the orchestrator workflow does not use).
- Prime tasks run on demand: "lyx: orch start" (`lyx orch start`: launches or attaches the orchestrator and takes the terminal) and "lyx: open board" (opens the board viewer, see `board-viewer`).
- Keybindings: lyx writes a `// lyx:begin` … `// lyx:end` block into the user `keybindings.json`, the way mill writes its `// mill:begin` block (mill holds Alt+Shift+1–6), e.g. Alt+Shift+0 for orch start and Alt+Shift+9 for open board, binding `workbench.action.tasks.runTask` to the task labels.
- No `launch.json`: mill used it to run Python scripts through the debugger; lyx is a binary, so tasks are the mechanism.
- Pairs keep their current chains (driven: attach only).
