# A Codex session gets exactly what a Claude session in the same role gets

`codex-same-as-claude` · Task · bug, codex, shuttle, stencil, heddle · running · Discussion-Burler-Fix

## Rule (operator, 2026-11-15)

A Codex session lym gives a role gets exactly what a Claude session in that role gets:
the same skills, the same standing instructions, the same project instructions, the same guards and the same prompt.
Giving it less is a bug, not a property of Codex.

## Seen (`codex-writer-bench-neve`, `c0815831`)

The bench did not use lym's production route.
It hand-built the Darn-Write prompt, rendered Claude's edit directive into it, loaded no skills, and launched it through a bare `lym shuttle run`.
Its weak points (fixed-column prose, pinned versions, no design decisions in the description, `cat` and `sed -n` unrefused) trace to that.
The benches so far were all bespoke in this way, and each gave Codex less than Claude (operator, 2026-11-16).

## What lym already has

- A seat is built engine-blind in `seatengine`: the darn writer's seat carries the role's skills and the hub's language skills, whichever engine runs it.
- Engine blocks (`contracts/stencils/engine/<engine>/`, `{{template "engine-<block>"}}`) give a stencil engine-specific text; the edit directive has a Codex copy that names `apply_patch` and the shell guard.
- `codexengine` loads skills by reading each `SKILL.md`, runs shell guards, and falls back to `CLAUDE.md` when a directory has no `AGENTS.md`.

## The gaps to close

- Stencil text outside the engine blocks that names Claude-only things.
  Seen: darn's make-change fragments (`darn-fragment-make-change.md`, `-gated.md`) name the Agent tool, `subagent_type`, and "the subagent passage of your system prompt".
  Sweep every stencil a Codex seat can render for the same and move each such passage into an engine block.
- Standing passages: heddle reads them per engine; check the Codex set gives what the Claude set gives.
- Instruction files: check whether `CLAUDE.local.md` and the user's global `CLAUDE.md` reach a Codex session, and give it their content if not.
- A parity test: for each seat a Codex model may run, render the launch for both engines and fail when the Codex one lacks a skill, passage or instruction the Claude one has.

## The bench rule

A Codex bench runs the real producer with only the model swapped, such as `{"darn":{"writer":"astra[high]"}}` in `run_settings`,
and never hand-builds Codex's prompt.
Its report opens with what each engine's session got, so an unequal start shows before any result.
The writer bench and its OSL twin rerun this way once this lands.
