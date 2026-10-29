# Take every non-LoomYard-specific rule out of the loomyard CLAUDE.md

`claude-md-lym-rules-out` · Note · enhancement, stencil

## Seen

The loomyard repository's `CLAUDE.md` holds, beside the rules for developing LoomYard itself, text that is general or that lym's agents need in every repository (operator, 2026-10-29).
Its own section says a rule for every use of lym goes into lym, in the stencils and skills lym ships, never into this file.
Whatever sits in it against that rule makes lym's use less invisible, since the agents learn it from a repository file.

## Idea

Go through `CLAUDE.md` rule by rule and take out everything that is not specific to developing LoomYard:

- A rule every lym agent needs moves into the stencil or skill that reaches that agent.
- A general rule that no lym agent needs goes to the plugin skill that already covers it, or is dropped.
- What stays is only what the agents that develop this repository need.

It feeds `warp-visibility`, which asks what lym's agents read from a `CLAUDE.md` today.
