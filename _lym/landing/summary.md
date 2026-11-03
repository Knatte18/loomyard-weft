# Engine-keyed stencil blocks and heddle's codex-file fallback

A stencil that more than one engine renders can now carry engine-specific text in an engine block, and a Codex session gets the repository's agent instructions a Claude session gets.
This is part 3 of project `codex-engine` (decisions D11 and D12).

## Engine blocks

- A stencil pulls in an engine block with the bare `{{template "engine-<block>"}}`.
  Each engine keeps its own copy at `engine/<engine>/<block>.md` under the stencils directory, registered as `engine/<engine>/<block>`.
- `stencilstore.ReadSet` is the one resolver.
  It reads a stencil with every block it includes, an engine block from the copy of the engine it is told.
  An engine with no copy of an included block fails with `MissingEngineBlockError`, naming the block, the engine and the file; an empty engine is refused when a stencil includes an engine block.
  A stencil without engine blocks reads the same whatever engine is given.
- `seatengine` reads each seat's stencil and blocks for the seat's engine: the engine its model choice names, else the new `Geometry.DefaultEngine`.
  `Table.Validate` now takes the default engine and refuses a seat whose engine lacks a block.
- `burlerengine` renders every instruction stencil (explore, review, review format, fix, focus directive, fork reading, cluster rules, self-check) through `ReadSet` for the engine of the half being rendered, with the same fallback to `Geometry.DefaultEngine`.
- `DefaultEngine` is wired from `shuttleCfg.Engine` in `loomcli` (burler, the MultiLLM seats and the landing shed) and in both modes of `burlercli`.
- `contracts/stencils/registry_test.go` walks the per-engine subfolders, so the registry and the on-disk tree stay in step for engine blocks.
- `PATTERN.md` gains `PATTERN-engine-blocks`.

No shipped stencil includes an engine block yet; this change adds the mechanism and its tests.

## Heddle codex-file fallback

- The `codex-file` target opens with the anchor's `AGENTS.md`, else its `CLAUDE.md`, else nothing.
- A repository entry aimed at `claude-file` now also reaches `codex-file`.
  A shipped entry reaches it only by naming it.
- A render whose `Only` names `claude-file` renders `codex-file` too, so an existing worktree picks up a changed repository entry at its next render.

## Check by hand

- `burlerengine`: the engine choice of each half (review and fix) is only partly pinned by tests; confirm a half whose model choice names an engine renders that engine's blocks, and one that names none falls back to `DefaultEngine`.
- `seatengine`: `Run`, `Probe` and `Resume` pass `DefaultEngine` through to the stencil read; confirm each entry point does, since the tests pin them only in part.
- Run a Codex seat and a Claude seat against a stencil with an engine block, and a worktree with only `CLAUDE.md`, to see the rendered `codex-file`; no shipped stencil exercises this path yet.
