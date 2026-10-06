# Shuttle crucible: harden turn reading against real Claude behaviour

`shuttle-crucible` · Note · enhancement, shuttle

Harden shuttle against how Claude Code behaves in practice, found by running it, rather than one fix per misread found in a live run (operator, 2026-10-06).

Seen on 2026-10-06 alone: false "asking" stops at Plan-Write (twice, one spawning a second plan agent) and on a Webster master whose fork was still measuring (#407), a fork killed by an API error recorded as `dead_reason: asking`, and a burler turn end never taken at all (#411).
Each came from behaviour shuttle had not been written for: forks and background shells outliving a turn, messages arriving mid-run, skill loads, a session `continued-in` another, API errors.
Shuttle reads undocumented Claude Code internals (transcript line shapes, `custom-title`), which change between Claude Code versions.

Two parts:

- A replay corpus: every misread from a real run is saved as a fixture (the transcript and shuttle's events, with what happened), and shuttle's classification is tested against the whole corpus. Cheap and deterministic; today's cases are the first fixtures.
- Live runs: real Claude sessions driven through scripted scenarios that provoke the hard cases (long background shells, forks, a message mid-turn, skill loads, a compact, an interrupt, a question), checking shuttle's reading against the known answer. Costs tokens; catches a Claude Code release that changes behaviour before a task run meets it.

Starting point (operator, 2026-10-06): loomyard has no crucible of its own yet.
The repo's `crucible/` holds the hand-run review-and-fix loop that hardened reed, written for millhouse (`_mill/` paths, round-agent profiles copied into `.claude/agents/`), and the operator expects it can be rewritten for this more easily than starting over.
Its README names the behaviour-based form, running a live-substrate module in a sandbox, a `hardener` board note, which is no longer on the board.
