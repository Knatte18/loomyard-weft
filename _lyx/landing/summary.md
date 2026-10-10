# CLI surface: command index for the orch, one help form, intake comments

The orch now gets a generated index of every command it may run, every command's help follows one test-enforced form, and `lyx board intake import` carries an issue's comments onto the board.
Before this, the orch rediscovered commands through `--help` one module at a time, the `Use`, `Short` and `Long` texts mixed styles, `[json-payload]` said nothing about a board command's keys, and an imported issue lost its comments.

Closes #479
Closes #480

## What changes

- `lyx help index` prints a nested one-line-per-command index of the running binary's command tree.
  It reads no files and caches nothing.
  It prints `operator` commands by default; `--audience role|internal` prints that audience instead, and a value outside the set is refused with an error naming the set.
- A new `internal/helpcli` module replaces cobra's built-in help command via `SetHelpCommand`.
  `lyx help [command]` keeps its behaviour, except that an unknown word at any depth is refused with a JSON `unknown subcommand` error naming it.
  `lyx --json` now lists `help` with its `index` child.
- Every invocable command carries an `audience` annotation (`operator`, `role` or `internal`).
  A tree walk in `cmd/lyx` refuses a missing or unknown value, a non-runnable group that carries one, and a hidden leaf that is not `internal`.
- The orch role stencil gains a `## Commands` section filled from the index, so the index arrives with the role file at `lyx orch start`, `--adopt`, `resume-context` and every watcher reload.
  The watcher cannot compute the index from its own command tree, since it outlives a deploy; `orchcli` hands it a source that runs `<recorded path> help index`.
- Reed records on each strand the absolute `lyx` path its pane prelude exported, so the watcher's index describes the binary the orch pane runs.
- A new `ly:cli` skill loads the index into an operator's own session.
- One form for `Use`, `Short` and `Long`, pinned by the same tree walk:
  - `Use` is the name, then positionals (`<arg>`, `[<arg>]`, `<arg>...`, `[<arg>...]`), then required flags only; optional flags live in `Flags:`.
  - `Short` starts lower-case, is one line and has no final period.
  - `Long` is capped at 2,000 bytes, and rationale moved to the packages' `doc.go`.
  - Every command was rewritten to fit; `lyx loom goto` and `lyx loom review approve` adopt the bracket form for their optional argument.
- Board payload commands name their keys in `Use` as one `{…}` token (`?` optional, `a|b` exclusive), with the grammar stated once in the `board` group's `Long` and on the index's `board` line.
  `set-deps` now addresses by `slug` or `id` like the other addressing verbs.
- `lyx board intake list` prints a `comments` count per issue from the list response alone.
  `lyx board intake import` fetches the comments before the board write and appends each under `## Issue comment, <YYYY-MM-DD>` (UTC), oldest first; under a fold into an existing entry the heading sits one level deeper.
  A comment whose whole trimmed body is lyx's own pointer comment is dropped; every other comment is carried verbatim, author not recorded.
- `PATTERN.md` and `pattern/PATTERN-cli-cobra.md` record the three new obligations, and `docs/overview.md`, the module `doc.go` files and the `test-budget.yaml` rows are updated.

## Design decisions to know

- The index goes through the role file rather than a typed skill load.
  The role file is re-read on every launch path including the reload after a compaction, which skips the skills step, and shuttle's `Send` refuses multi-line text.
  This departs from the board entry's wording.
- `orchengine` imports no cobra: it receives the index as a string, and the watcher's source runs only the strand's recorded path, never a `PATH` lookup.
- `startAutoReload` now goes idle with the reason when the role-file render fails, as `startClearing` and `reloadAfterCompaction` already did; before, it returned a failed tick.
  This is the only change to the watcher's reload steps.
- Groups carry no `audience`; a group's audience is the union of its leaves.
  Borderline diagnosis verbs (`webster status`, `webster reset`, `shuttle state`, every `status`) are `operator`.
- Role file growth is about 3.5k tokens per read, accepted by the operator.

## To check by hand

- Watcher index binary: the watcher renders from the path reed recorded at the orch strand's last launch, which can differ from the watcher's own executable.
  Confirm after a deploy that the next reload shows the new index.
- Existing strands: an orch strand launched before this change has no recorded path, so its watcher goes idle with a reason until `lyx orch stop` then `lyx orch start`.
- Operator-edited role stencil without the `{{.command_index}}` marker renders no index; confirm the render error names the stencil.
- The decision record states the missing-binary bound at `startAutoReload` in two places that disagree (D4 says idle, a later addition says tick error); the code goes idle, so check D4's wording matches in `internal/orchengine/doc.go`.
- `Long` trimming was judgment: spot-check that each cut page's refusal messages still name their way forward.
- Run `lyx help index` against a live orch relaunch, and `lyx board intake import` against a real issue with comments; both were tested only against fakes.
