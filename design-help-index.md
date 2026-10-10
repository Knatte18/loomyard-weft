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


## Issue comment, 2026-10-09

Measured a variant with each command's signature on its line: the first `Use` line (minus `[flags]`), then the flag names without descriptions, then `Short`.

```
- `lyx batten goto [<run-id>] --to <producer>` `--to`: move a halted lifecycle onto a named row, paused, with a fresh segment budget
- `lyx board upsert [json-payload]` `--body-file`: Create or update a single task
```

140 lines, 14 KB, about 3.5k tokens: about 500 tokens more than the `Short`-only index, and the orch can call most simple commands without a `--help` first.
Flag descriptions would add another 14 KB and are not worth it.
Proposal: `lyx help index` prints this form.

The walk also showed that `Use` and `Short` follow no common form (flags inside `Use` on some commands, mixed argument spellings, `[json-payload]` without keys, mixed capitalisation), which makes the index uneven; filed separately as #480.



## Issue comment, 2026-10-09

## Marking who a command is for, at the Cobra level

"Internal" covers two different things: a command only lyx's own Go code or a key binding runs, and a command one spawned role runs because its stencil names it.
Neither belongs in the index injected into the orch's prompt.
Both stay visible in `lyx --help` (operator decision, 2026-10-09): the annotation decides what the orch is handed, never what help shows.

Proposal:

- **An `audience` annotation on every command**, in Cobra's `Annotations` map (lyx already uses it, such as `clihelp.SkipStencilSeedAnnotation`), with a closed set of values:
  - `operator`: the orch and the operator's own sessions.
  - `role`: a spawned role that gets the command from its stencil.
  - `internal`: only lyx's Go code or a key binding.
- **A test walks the command tree** and fails on a command with no `audience` or a value outside the set, so a new command cannot skip the decision.
- **`lyx help index` prints `operator` commands by default**; `--audience role` exists for inspection, though a stencil should name its role's few commands directly.

## Shorter index lines

Each line names only its own command word and the rest of its signature, since `lyx` and the parent path are implied by the nesting, and lists only flags its signature does not already spell:

```
- `batten`: drive one task worktree's whole lifecycle as a single Shed run
  - `goto [<run-id>] --to <producer>`: move a halted lifecycle onto a named row, paused, with a fresh segment budget
  - `run [<run-id>]` `--child-driver --driver --window`: start or resume a task worktree's whole lifecycle run to its next halt
```

That saves 11–12% (14.2 KB to 12.6 KB with signatures).
`Short` texts are not to be shortened; some are too short to say when a command applies (#480).

## A first classification

From the command index and who calls each command, as a starting point to review once when the annotation lands; about 45 of 140 commands are `role` or `internal`:

| Module | Commands | Called by |
| --- | --- | --- |
| reed | `switch`, `watchdog`, `add`, `resume` | the Alt+arrow bindings, the watchdog daemon, shuttle and orch code |
| shuttle | `run`, `send`, `interrupt`, `state` | the Go code that spawns agents |
| burler | `run`, `await-review`, `validate-review` | the round's Shed row and the fixer |
| webster | `begin-batch`, `await-batch`, `record-batch`, `recover-batch`, `run`, `rebaseline`, `restore-plan`, `accept-audit`, `validate`, `verify` | Merriam and the recovery session |
| loom | `step`, `run`, `commit-records`, `validate-plan`, `validate-discussion`, `validate-description`, `lint-comments`, `decision add`, `review delivered`, `review notify` | the driver and the writers |
| shed | `seed`, `step`, `run` | batten and the drivers |
| fabric | `merge-in`, `merge-stage`, `commit` | landing's Go code |
| quarry | `glyphs`, `resolve`, `toc`, `expand` | the planner |

Borderline: `webster status`, `webster reset` and `shuttle state` may serve the orch when it diagnoses a stuck run.



