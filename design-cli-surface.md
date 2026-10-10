# CLI surface: a command index for the orch, one help form, intake keeps comments

`cli-surface` · Task · bug, enhancement, orch, board, selfreport · awaiting · PR-Gate

## Goal

The orch knows every `lyx` command it may use without reading `--help` first, and every command's help reads the same way:

- `lyx help index` prints a generated one-line index of the command tree, and the orch watcher injects it into the orch's context (about 3k tokens; operator, 2026-10-10).
- Every command's `Use`, `Short` and `Long` follow one documented form, enforced by a test that walks the command tree.
  `[json-payload]` is fixed as part of that form, since it says too little: only `lyx board` takes a JSON payload, the payload form is the same for every board command, so it is stated once and each command's `Use` names its keys (operator, 2026-10-10).
  `set-deps` takes `id` like the other addressing verbs.
- `lyx board intake` carries issue comments onto the board.

The three parts below are the merged notes, each kept as written.

## Part 1: help index (#479)

Imported from [issue #479](https://github.com/Knatte18/loomyard/issues/479).

The orch needs an overview of every `lyx` command to choose among them; today it rediscovers them through `--help`, one module at a time.
A full help dump is too large to load, and the one-line descriptions Cobra already carries are almost small enough.

### Measured 2026-10-09

- Every command's `--help`, minus global flags, `-h` and the "Use … --help" hint: 141 commands, 4,100 lines, 159 KB, about 40k tokens.
  Most of it is design rationale and internal history rather than usage, such as `quarry --help` explaining when the `{"ok":false}` envelope appears.
- One line per command from Cobra's `Short`, nested by module: 140 lines, 12 KB, about 3k tokens.
- `lyx --help` runs in under 10 ms, so an in-process walk of the command tree costs nothing worth measuring.

### Proposal

- **`lyx help index`** walks the Cobra tree in memory and prints one line per command: its full path and its `Short`.
  It reads no files and caches nothing, so it always matches the running binary.
- **An internal annotation** on commands that only lyx itself or one role calls, kept out of the index injected into the orch, while `--help` still lists them.
  Examples: `lyx reed switch` (run by the Alt+Left/Alt+Right tmux bindings through `run-shell`), the `lyx webster` batch verbs, `lyx burler await-review`, `lyx fabric merge-stage`.
- **Consistent `Short` texts** that say when to use a command, not only what it is: today they mix capitalisation, and some say too little ("task-tracker board", "unified warp↔weft git-coordination").
- **Trimmed long help**: usage, flags and one example; design rationale belongs in the package's `doc.go`.

### Who gets the index

Only the orch, and the operator's own sessions.
A spawned role gets the exact commands it runs from its stencil and nothing more: the driver runs `lyx shed step`, Merriam the `lyx webster` batch verbs, the planner `lyx quarry`.
More would only give a role commands it should not run.

- **The orch:** the watcher injects the index at `lyx orch start`, at `--adopt` and after every cycle, since a compaction drops what was loaded.
  The prompt says the index is complete and `lyx <command> --help` gives the details, so the orch never needs to call `lyx help index` itself.
  Same mechanism as the skill loading in `lyx-skills-and-stencils`.
- **The operator's sessions:** a skill with a short workflow section and `` !`lyx help index` `` in its `SKILL.md`, so Claude Code runs the command once when the skill loads and the content is always current.

### Why a CLI index, not an MCP server

An MCP server configured for the project loads into every Claude session started there, every spawned Burler, Webster and driver session included, so every role pays for tools it must not use.
With the CLI, only the orch pays for the index, and the CLI is already built, tested, and the same for an agent and a person at a terminal.


### Issue comment, 2026-10-09

Measured a variant with each command's signature on its line: the first `Use` line (minus `[flags]`), then the flag names without descriptions, then `Short`.

```
- `lyx batten goto [<run-id>] --to <producer>` `--to`: move a halted lifecycle onto a named row, paused, with a fresh segment budget
- `lyx board upsert [json-payload]` `--body-file`: Create or update a single task
```

140 lines, 14 KB, about 3.5k tokens: about 500 tokens more than the `Short`-only index, and the orch can call most simple commands without a `--help` first.
Flag descriptions would add another 14 KB and are not worth it.
Proposal: `lyx help index` prints this form.

The walk also showed that `Use` and `Short` follow no common form (flags inside `Use` on some commands, mixed argument spellings, `[json-payload]` without keys, mixed capitalisation), which makes the index uneven; filed separately as #480.



### Issue comment, 2026-10-09

### Marking who a command is for, at the Cobra level

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

### Shorter index lines

Each line names only its own command word and the rest of its signature, since `lyx` and the parent path are implied by the nesting, and lists only flags its signature does not already spell:

```
- `batten`: drive one task worktree's whole lifecycle as a single Shed run
  - `goto [<run-id>] --to <producer>`: move a halted lifecycle onto a named row, paused, with a fresh segment budget
  - `run [<run-id>]` `--child-driver --driver --window`: start or resume a task worktree's whole lifecycle run to its next halt
```

That saves 11–12% (14.2 KB to 12.6 KB with signatures).
`Short` texts are not to be shortened; some are too short to say when a command applies (#480).

### A first classification

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





## Part 2: a common form for Use, Short and Long (#480)

Imported from [issue #480](https://github.com/Knatte18/loomyard/issues/480).

The `Use` and `Short` texts of lyx's Cobra commands follow no common form, so a generated command index (#479) and every `--help` an agent reads are uneven.
Found 2026-10-09 by walking every command's `--help` (140 commands).

### `Use` lines

- **Flags inside `Use` on some commands, not on others.** 13 commands spell flags into `Use`, so a help page lists them twice (in `Usage:` and `Flags:`), while every other command leaves them to `Flags:`.
  Examples: `lyx fabric clone [--shortname <shortname>] [--reset] [--subpath <rel>] [--force-bootstrap] [--into <dir>] <weft-url> [<warp-url>]`, `lyx fabric cleanup [--apply] [--force] [--remote]`, `lyx fabric remove [--force] [--remote] <slug>`, `lyx loom review approve [slug] [--review <file>]`, `lyx shed seed <run-id> --recipe <name>`.
  A required flag (`goto --to <producer>`, `shed seed --recipe <name>`) is a fair exception; an optional one is not.
- **Argument spelling differs:** `lyx batten goto [<run-id>]` and `lyx shed goto [<run-id>]`, but `lyx loom goto [run-id]` and `lyx loom review approve [slug]`.
- **`[json-payload]` names no keys.** The 10 board commands that take a JSON payload say only `[json-payload]`; the keys are in the long help, so an agent always needs `--help` before a first call.
  A short key list in `Use` or in the first line of the long help, such as `upsert '{"slug", "title"?, "brief"?, …}'`, would be enough for most calls.

### `Short` lines

- **Capitalisation is mixed:** 36 start with a capital letter (all 20 `board` commands, `stencil`, `quarry`, `ide`) and 104 with a lower-case one.
- **Some say what a module is, not when to use it:** `lyx board`: "task-tracker board"; `lyx fabric`: "unified warp↔weft git-coordination".

### Expected

One documented form for `Use` and `Short`, enforced by a test that walks the command tree, as `PATTERN.md`'s other CLI rules are:

- `Use` carries positional arguments in one spelling (`<arg>` required, `[<arg>]` optional) and required flags only.
- `Short` starts in lower case, has no final period, and says what the command does in terms of when to reach for it.


### Issue comment, 2026-10-09

### `[json-payload]`: name the shapes once, reference them per command

Only `lyx board` takes a JSON payload, in 10 commands.
The form is always one JSON object with unknown keys rejected, and the keys fall into a few shapes that recur across commands:

| Shape | Keys | Used by |
| --- | --- | --- |
| `<addr>` | `slug` or `id` | `get`, `remove`, `promote`, `set-status`, and `merge`'s `set_status` |
| `<task>` | `slug`, plus optional `title`, `brief`, `body`, `depends_on`, `isolated`, `status`, `kind`, `labels`, `recipe`, `short_name` | `upsert`, each element of `upsert-batch`, `merge`'s `upsert` |

Proposal: define each shape once, in the `lyx board` group's help, and have each command's `Use` reference it instead of saying `[json-payload]`:

```
board: payload shapes: <addr> = {slug|id}; <task> = {slug, title?, brief?, body?, depends_on?, kind?, labels?, …}
  get <addr>
  remove <addr>
  promote <addr>
  set-status <addr>+{status}
  set-deps {slug, depends_on}
  upsert <task>
  upsert-batch {tasks: [<task>]}
  merge {remove_slugs?, upsert: <task>, set_status?: <addr>+{status}}
  intake close {issue, reason, completed?}
  intake import {issue, slug|into, title?, brief?, labels?}
```

The shapes are then stated in one place, the command index (#479) carries them for the cost of one line, and an agent can call most board commands without reading `--help` first.
A test can hold the shapes and the payload parsers together, so a key added to one cannot be missing from the other.

### `set-deps` takes no `id`

`get`, `remove`, `promote` and `set-status` address an entry by `slug` or `id`; `set-deps` takes `slug` only.
It should take `<addr>` like the others.



### Issue comment, 2026-10-09

### Long help: too long at the top

Every command's `--help` together is 159 KB, about 40k tokens.
The median page is about 740 bytes, which is fine; the problem is the top, where about ten pages run 2.5–5.4 KB each:

| Page | Bytes |
| --- | --- |
| `lyx loom` | 5,419 |
| `lyx webster` | 5,088 |
| `lyx fabric clone` | 4,973 |
| `lyx loom start` / `lyx start` | 4,385 / 4,380 |
| `lyx webster reset` | 4,157 |
| `lyx fabric cleanup` | 3,936 |
| `lyx webster validate` | 2,866 |
| `lyx loom resume` | 2,782 |
| `lyx fabric remove` | 2,740 |

Measured 2026-10-09 by walking every command's `--help`, without global flags, `-h` and the "Use … --help" hint.

Three patterns recur:

- **A group page restates its subcommands in prose.** `lyx loom --help` describes all its subcommands in one long paragraph, and Cobra then lists them again under "Available Commands"; the list alone is enough.
- **Implementation detail.** `start` "seeds the status file, commits the seed, and spawns/attaches the detached driver session"; `fabric clone` explains that the binding is "a plain single-line record, .lyx-warp, kept at the board root".
  That belongs in the package's `doc.go`.
- **Edge cases and refusals stated up front.** `fabric clone` spends several paragraphs on when `--reset` and `--subpath` are refused.
  The error message at the moment of refusal, naming the remedy, serves better.

Proposed norm for `Long`: what the command does, when to use it, what its arguments and flags mean, and one or two examples.
Everything else moves to `doc.go` or to the error messages.
Under that norm the largest pages would likely shrink to about a third, which also makes `lyx <command> --help | grep …`, the orch's usual lookup, cheaper and less noisy.





## Part 3: intake carries issue comments

`lyx board intake import` writes only the issue title and body (`issueContent` in `internal/boardengine/intake.go`); `selfreportengine` never fetches comments, so every comment on an issue is lost at import.
The operator treats a comment as part of its issue: #479 and #480 carried as much in comments as in their bodies and were completed by hand on 2026-10-10.

### Expected

- `import`, for a new note and for `into`, appends each comment after the body under a `## Issue comment, <date>` heading, oldest first.
- `intake list` shows the comment count, so an orch sees that an issue has comments before it decides how to import it.
- The pointer comment lyx itself posts on import is never carried back in.
