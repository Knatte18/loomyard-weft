# Cobra Use and Short texts follow no common form

`cli-help-form` · Note · bug, board

Imported from [issue #480](https://github.com/Knatte18/loomyard/issues/480).

The `Use` and `Short` texts of lyx's Cobra commands follow no common form, so a generated command index (#479) and every `--help` an agent reads are uneven.
Found 2026-10-09 by walking every command's `--help` (140 commands).

## `Use` lines

- **Flags inside `Use` on some commands, not on others.** 13 commands spell flags into `Use`, so a help page lists them twice (in `Usage:` and `Flags:`), while every other command leaves them to `Flags:`.
  Examples: `lyx fabric clone [--shortname <shortname>] [--reset] [--subpath <rel>] [--force-bootstrap] [--into <dir>] <weft-url> [<warp-url>]`, `lyx fabric cleanup [--apply] [--force] [--remote]`, `lyx fabric remove [--force] [--remote] <slug>`, `lyx loom review approve [slug] [--review <file>]`, `lyx shed seed <run-id> --recipe <name>`.
  A required flag (`goto --to <producer>`, `shed seed --recipe <name>`) is a fair exception; an optional one is not.
- **Argument spelling differs:** `lyx batten goto [<run-id>]` and `lyx shed goto [<run-id>]`, but `lyx loom goto [run-id]` and `lyx loom review approve [slug]`.
- **`[json-payload]` names no keys.** The 10 board commands that take a JSON payload say only `[json-payload]`; the keys are in the long help, so an agent always needs `--help` before a first call.
  A short key list in `Use` or in the first line of the long help, such as `upsert '{"slug", "title"?, "brief"?, …}'`, would be enough for most calls.

## `Short` lines

- **Capitalisation is mixed:** 36 start with a capital letter (all 20 `board` commands, `stencil`, `quarry`, `ide`) and 104 with a lower-case one.
- **Some say what a module is, not when to use it:** `lyx board`: "task-tracker board"; `lyx fabric`: "unified warp↔weft git-coordination".

## Expected

One documented form for `Use` and `Short`, enforced by a test that walks the command tree, as `PATTERN.md`'s other CLI rules are:

- `Use` carries positional arguments in one spelling (`<arg>` required, `[<arg>]` optional) and required flags only.
- `Short` starts in lower case, has no final period, and says what the command does in terms of when to reach for it.


## Issue comment, 2026-10-09

## `[json-payload]`: name the shapes once, reference them per command

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

## `set-deps` takes no `id`

`get`, `remove`, `promote` and `set-status` address an entry by `slug` or `id`; `set-deps` takes `slug` only.
It should take `<addr>` like the others.



## Issue comment, 2026-10-09

## Long help: too long at the top

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



