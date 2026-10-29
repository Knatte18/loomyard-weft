# Delete the loomyard CLAUDE.md after moving any rule only it holds

`claude-md-lym-rules-out` · Note · enhancement, stencil

## Seen

The loomyard repository's `CLAUDE.md` holds, beside the rules for developing LoomYard itself, text that is general or that lym's agents need in every repository (operator, 2026-10-29).
Its own section says a rule for every use of lym goes into lym, in the stencils and skills lym ships, never into this file.
The operator's view is that nothing in it needs to be there: the rules to follow are in `PATTERN.md`, which lym injects where it is needed, and a session opened by hand has no goal and needs none of it.

## Idea

Delete the loomyard `CLAUDE.md` once every rule that stands only in it has a new home:

- Check each rule against `PATTERN.md`, the stencils and the plugin skills, and move any rule that stands only in `CLAUDE.md` to the place that reaches the agents that need it.
- The cgo build requirement goes into `docs/` or an error message of the build script.
- `update-plugins.sh` gets a guard that refuses a run no operator started, if it has none, in place of the text rule.
- The terminology shorthands (Merriam, perch, board, Bolt) go to the orch rules, which are the only place that talks with the operator in them.

It follows from `warp-visibility`, which records that LoomYard adds no instruction file to any repository.
