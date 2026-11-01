# Add the review turn audit of the Discussion and Plan review segment

Adds `docs/research/review-turn-audit-2026-11-01/`: a study of where the turns, context and money of the Discussion and Plan review segment go on the three latest full tapestry runs (`warp-visibility`, `interactive-recipe`, `run-verbs-shared`).
The segment is the burler reviewer and fixer, the bouncer's seed and judge, and the lens forks of the reviewer and fixer.
It adds research files only: no Go, stencil, config or `PATTERN.md` change.

## What the study finds

- The segment costs $101.59 at list price; the main reviewer and fixer sessions are $74.27 of it and their forks $23.62.
  The orch's four figures that prompted the study (Plan reviewer $31, Discussion reviewer $18, fixers $25, forks $24) come out of the same price table to the dollar.
- The Opus Plan reviewer makes 89% of its turns single-call and 33 of 77 turns per session removable by a merge (41% of its weight, n = 3), though its stencil already carried the batching line of `bf00a043`.
- A reviewer or fixer that spawns forks wakes once per fork result, a no-call turn at the full context: 79 such turns, 8% to 9% of the weight of the three roles that do it.
- Priced as Sonnet with the same tokens, the Discussion and Plan fixers with their forks cost $22.89 against $34.15, a saving of $11.26 (33%); it breaks even at about 1.5 times the tokens.
  The sample holds no Sonnet fixer of these roles, so it does not measure fixer quality or turn growth.

`report.md` holds the tables and the recommendations; a recommendation changes nothing.

## Method and files

The study reuses the method and four scripts of `docs/research/turn-audit-2026-10-28`: `extract-turns.sh`, `session-metrics.sh`, `role-summary.sh` and `merge-kinds.sh` with `kinds-total.sh`.
`run-all.sh` runs them on a copy beside this folder's `slugs.txt` and rebuilds every table.
The new scripts are `session-costs.sh` (list-price cost per session, and the same tokens priced as Sonnet), `where-turns-go.sh` (weight and time per activity group), and `role-table.sh`, `rounds-table.sh`, `merge-saving.sh` and `fixer-cost.sh`, which aggregate what the report quotes.
`README.md` states how to rerun and add a run.

## Design decisions

- **Forks drop their first turn.**
  A fork transcript opens with its parent's spawning Agent message, replayed; counting it would charge the parent's turn twice.
  `run-all.sh` drops turn 1 of every fork and stops if a fork opens with any other call.
  The turn audit's tables include that replayed turn; this study's do not.
- **Sonnet sessions keep their own role row.**
  A reviewer or fixer that ran on Sonnet takes `@sonnet` after its role in the tables, so later Plan review rounds (Sonnet, no forks) stay apart from round 1 (Opus, with forks).
- **Prices come from `internal/shuttleengine/claudeengine/pricing.go`**, copied into `session-costs.sh` for the two models the sample holds; the script stops on any other model.
  `tools/tokencount` was not run, since `go run` is refused in the session.
- **No agent or model run.**
  The study reads existing transcripts only, as the entry's cap asks.
- **Note `fixers-on-sonnet` is not edited.**
  The report states the Sonnet figure beside it and leaves the note's body to the operator.

## Testing

The change adds no Go code.
`lym gate test` passed (tier1: 3 passed, 0 failed).
`run-all.sh` was run twice, with a fresh work directory and with the default one, and rebuilt the same tables.
