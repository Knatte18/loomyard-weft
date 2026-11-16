# A Codex session gets exactly what a Claude session in the same role gets

`codex-same-as-claude` · Note · bug, codex, shuttle, stencil, heddle

## Rule (operator, 2026-11-15)

A Codex session lym gives a role gets exactly what a Claude session in that role gets:
the same skills, the same standing instructions, the same project instructions, the same guards and the same prompt.
Giving it less is a bug, not a property of Codex.

## Seen (`codex-writer-bench-neve`, `c0815831`)

The bench replayed the darn writer with `astra[high]` through a bare `lym shuttle run`, and its weak points trace to what the session never got:

- No skills: the Claude writer loads `scribe:prose`, `scribe:code-quality` and the rest through its system prompt; the Codex session loaded none, so it hard-wrapped prose at a fixed column, pinned versions and line numbers, and wrote a change description with no design decisions.
- The Darn-Write stencil speaks Claude's tools: Read, Edit, Write, the Agent tool, "the subagent passage of your system prompt". Codex has none of them by those names, so the rule against `cat` and `sed -n` read as void.
- No guard refused its `cat`, `sed -n` and `rg`, where hooks refuse them for Claude.
- The repository has no `AGENTS.md`; `codexengine` falls back to `CLAUDE.md`, but `CLAUDE.local.md` and the user's global `CLAUDE.md` reach a Claude session and maybe not a Codex one.

## Idea

Find every route by which a Claude session in a role gets instructions, and give a Codex session in that role the same, through `codexengine`:

- the role's skills (`codexengine` has a skill load; check every role and recipe passes the same list to both engines);
- heddle's standing passages for the role (they are read per engine: check the Codex set matches the Claude set);
- the project and user instruction files;
- the shell and edit guards the hooks apply;
- stencil wording that names a Claude tool, made engine-neutral or given a Codex rendering.

Add a check that fails when a role's Codex launch carries less than its Claude launch.
Then rerun the writer bench, and its OSL twin, with the full set before the operator judges level 2.

## How parity is kept (operator, 2026-11-16)

The benches so far were bespoke: the orchestrating Claude session assembled Codex's input by hand,
and each time Codex started from less than Claude, never the other way round.
So parity is built into lym, never left to whoever sets up a run:

- One launch for both engines.
  A role's launch spec (prompt, skills, standing passages, instruction files, guards) is built once, engine-blind;
  the engine only renders it.
  A Codex launch can then differ from a Claude launch only where an engine renders the same item its own way.
- A parity test.
  For each role and recipe, the test builds the launch for both engines and fails when the Codex launch lacks an item the Claude launch carries.
- Engine-neutral stencils.
  No stencil names a Claude-only tool;
  where a tool must be named, the engine supplies its own name for it.
- A Codex session runs through the production route.
  The model alias alone picks the engine, such as `{"darn":{"writer":"astra[high]"}}` in a task's `run_settings`,
  and the same producer builds the same launch.
  Codex goes nowhere through a bare `lym shuttle run` with a hand-written prompt.
- The bench rule follows from it.
  A Codex bench runs the real producer with only the model swapped;
  it never hand-builds Codex's prompt.
  Its report opens with what each engine's session got, from the launch records, so an unequal start shows before any result.
