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
