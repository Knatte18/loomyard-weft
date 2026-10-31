# Landing commits end with a plain credit block, not Co-Authored-By

A landing commit and its pull request no longer carry `Co-Authored-By` or `Lym-Task` lines.
They end instead with one plain-text credit block, on by default:

```
Created using
 - LoomYard https://github.com/Knatte18/loomyard
 - Claude <noreply@anthropic.com>
```

The operator does not want an engine credited as a co-author, and sees no use for the `Lym-Task: <slug>` line.

## What changes

- The block is the heading, the LoomYard line, then one line per engine that ran a session in the task, in engine-name order, each once.
  The engine detection is the existing engines-run read, unchanged.
- The block is written after a blank line as plain text, never as git trailers.
- The landing commit message, the pull-request merge message at Finalize and the pull-request body PR-Review sets all come from one composition, so a commit and its pull request carry one signature.
- PR-Review sets the body on every entry that edits it, so a fixer session added later shows on the next edit.
- `taskTrailer` and the `Lym-Task` line are removed.
- Docs that named the trailer are updated: `internal/landingshed/doc.go`, `docs/overview.md`, `contracts/specs/loom-final-summary-spec.md` and `contracts/stencils/landing/landing-template-describe.md`.
  The `summaryparser` Co-Authored-By finding detail and the `envkit` seam list follow.

## Config

Four `landing.yaml` keys replace `trailer_text` and `engine_trailers`, all cross-pair keys:

- `credit` (bool, default `true`) turns the block on or off.
- `credit_heading` (default `Created using`), `credit_loomyard` and the map `credit_engines` hold one line's text each.
- `credit_engines` keeps the engine-name check: a key naming no engine fails the load.
- An empty or whitespace-only text adds no line; a block with no item lines, or with `credit` off, is empty and the message ends in the body.

## Design decisions

- A bool states "off" plainly, where emptying the texts would be an implicit switch.
- `trailer_text`, `engine_trailers` and `co_authored_by` are retired keys: a hub file still holding them loads, and reconcile prunes them.
  The `co_authored_by` to `engine_trailers` migration is removed, since carrying those values over would keep the co-author text this change removes.
- The template carries the Claude line only.
  `credit_engines` refuses an engine name `modelspec` does not know and `codex` is not one yet, so the Codex engine's task adds `codex: "Codex <codex@openai.com>"` with the engine.
- `fabricengine`'s `Warp-SHA` trailer is untouched; it is weft bookkeeping and never lands in the code repository.

## To check by hand

- A pull request opened before this change keeps its old body until PR-Review next edits it.
- PR-Review with `credit` on but an empty block, in `internal/landingshed/prreview.go`: confirm the empty-block test covers the created and edited body.
- PR-Review's engines-run read failure in `internal/landingshed/prreview.go`: confirm the test covers it as an error, as at Finalize.
