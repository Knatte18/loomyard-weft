# Engine-keyed stencil blocks and heddle's codex-file CLAUDE.md fallback

Part 3 of project `codex-engine` (decisions D11 and D12 of its Sketch).
It gives a stencil a way to carry text that differs by the engine a seat runs on, and makes the instructions a Codex session reads match what a Claude session reads.
No shipped stencil uses an engine block yet, so every Claude seat's prompt is unchanged; the assistant's stencils in `codex-assist-seat` are the first users.

## Engine blocks (D11)

- A stencil includes an engine block as the bare `{{template "engine-<block>"}}`.
  Each engine keeps its own copy at `engine/<engine>/<block>.md` under the stencils directory, registered under the name `stencilstore.EngineBlockName` forms (`engine/<engine>/<block>`).
  `stencilstore.RelPath` maps a name holding a `/` to `<name>.md`, so seeding, reading, `lym stencil` and the history paths treat these names like any other stencil.
- `stencilstore.ReadSet(baseDir, engine, name)` is the one resolver: it reads a stencil with every block it includes, an engine block read from the told engine's copy.
  An engine with no copy is a `*stencilstore.MissingEngineBlockError` naming the block, the engine and the file, with a way forward, so a later caller can tell this refusal apart from other read errors (the assistant seat will not start on it instead of refusing the table).
  An engine block read with no engine named is refused.
- `seatengine` reads every seat's stencil set for the seat's engine in `Table.Validate`, `Table.CheckMarkers` and prompt composition: the engine its model choice names, else the new `Geometry.DefaultEngine`.
  `Validate` and `CheckMarkers` now take the default engine as a second argument.
  A block missing for a seat's engine refuses the table, naming the block, the engine and the file.
- `burlerengine` composes each half's instruction files (explore, review, review format, fix, and their parts) through `ReadSet` for the half's engine, the one its model choice names, else the new `Geometry.DefaultEngine`.
- Shuttle's default engine (`shuttle.yaml`'s `engine`) reaches the seat engine, the burler engine and the MultiLLM entry (new `shedrecipe.Env.DefaultEngine`) through `loomcli` and `burlercli` wiring, and the darn preflight's marker check.
- `contracts/stencils`' registry test walks the per-engine subfolders, so a shipped engine block must be registered.
- `PATTERN.md` gains `PATTERN-engine-blocks`.

## heddle's codex-file falls back to CLAUDE.md (D12)

- The `codex-file` target (`AGENTS.override.md`) opens with the anchor's `AGENTS.md`, else its `CLAUDE.md`, else neither, and a `CLAUDE.md` change re-renders it.
  Codex loads one instruction file per directory, the override first, so without this a rendered override would hide the `CLAUDE.md` the launch's fallback would otherwise load.
- Every repository-layer entry aimed at `claude-file` also renders into `codex-file`, so a Codex reviewer gets the repository's agent instructions the Claude chair gets (the operator's rule of 2026-10-27).
  Shipped entries stay on `claude-file` alone: they direct Claude's own tools.
- A render whose target list names `claude-file` renders `codex-file` too, so an existing worktree picks up a changed repository entry at its next render.
- A consequence: a worktree whose repository has `claude-file` entries now gets an `AGENTS.override.md` (git-excluded, like `CLAUDE.local.md`).

## Design decisions

- The default engine is told as a plain name on each geometry rather than read through the shuttle seam, so table validation stays callable without a runner, and the seat fakes in tests needed no new method.
  A test that passes `""` is fine as long as its stencils include no engine block.
- The assistant-specific half of D11 (an assistant whose engine lacks a block does not start while the chair runs) depends on the assistant seat kind of D1, which `codex-assist-seat` builds; this part provides the typed error it will check for.

## Tests

- `stencilstore`: `TestReadSet` covers plain includes, the Claude and Codex copies of one block, a missing copy (the typed error and its message), and an engine block with no engine; `TestRelPath` gains an engine-block row.
- `seatengine`: `Validate` rows for a block read for the seat's own engine, for the default engine, and missing for the chair's or an advisor's engine; `CheckMarkers` rows for a marker held only in the codex copy; `TestPrompts_RenderEachSeatsEngineBlocksForItsEngine`.
- `burlerengine`: `TestComposePrompt_RendersEngineBlocksForTheHalfsEngine` covers both halves on both engines.
- `heddleengine`: the codex-file opening as a table (AGENTS.md, CLAUDE.md, both, neither), a repository `claude-file` entry rendering into both files, Only `claude-file` writing `codex-file`, and the seam re-rendering on a `CLAUDE.md` change.
