# lyx help index: a generated one-line command index, injected into the orch only

`help-index` · Note · enhancement, orch

Imported from [issue #479](https://github.com/Knatte18/loomyard/issues/479).

The orch needs an overview of every `lyx` command to choose among them; today it rediscovers them through `--help`, one module at a time.
A full help dump is too large to load, and the one-line descriptions Cobra already carries are almost small enough.

## Measured 2026-10-09

- Every command's `--help`, minus global flags, `-h` and the "Use … --help" hint: 141 commands, 4,100 lines, 159 KB, about 40k tokens.
  Most of it is design rationale and internal history rather than usage, such as `quarry --help` explaining when the `{"ok":false}` envelope appears.
- One line per command from Cobra's `Short`, nested by module: 140 lines, 12 KB, about 3k tokens.
- `lyx --help` runs in under 10 ms, so an in-process walk of the command tree costs nothing worth measuring.

## Proposal

- **`lyx help index`** walks the Cobra tree in memory and prints one line per command: its full path and its `Short`.
  It reads no files and caches nothing, so it always matches the running binary.
- **An internal annotation** on commands that only lyx itself or one role calls, kept out of the index injected into the orch, while `--help` still lists them.
  Examples: `lyx reed switch` (run by the Alt+Left/Alt+Right tmux bindings through `run-shell`), the `lyx webster` batch verbs, `lyx burler await-review`, `lyx fabric merge-stage`.
- **Consistent `Short` texts** that say when to use a command, not only what it is: today they mix capitalisation, and some say too little ("task-tracker board", "unified warp↔weft git-coordination").
- **Trimmed long help**: usage, flags and one example; design rationale belongs in the package's `doc.go`.

## Who gets the index

Only the orch, and the operator's own sessions.
A spawned role gets the exact commands it runs from its stencil and nothing more: the driver runs `lyx shed step`, Merriam the `lyx webster` batch verbs, the planner `lyx quarry`.
More would only give a role commands it should not run.

- **The orch:** the watcher injects the index at `lyx orch start`, at `--adopt` and after every cycle, since a compaction drops what was loaded.
  The prompt says the index is complete and `lyx <command> --help` gives the details, so the orch never needs to call `lyx help index` itself.
  Same mechanism as the skill loading in `lyx-skills-and-stencils`.
- **The operator's sessions:** a skill with a short workflow section and `` !`lyx help index` `` in its `SKILL.md`, so Claude Code runs the command once when the skill loads and the content is always current.

## Why a CLI index, not an MCP server

An MCP server configured for the project loads into every Claude session started there, every spawned Burler, Webster and driver session included, so every role pays for tools it must not use.
With the CLI, only the orch pays for the index, and the CLI is already built, tested, and the same for an agent and a person at a terminal.
