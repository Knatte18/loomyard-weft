# Lens fan: master pre-reads the cited code before forking, forks get a read budget

`fan-master-preread` · Task · enhancement, burler, stencil

## Seen

The lens fan's design: the master reviewer reads most of what the round needs first, then forks, so each lens fork inherits that reading through the cache and needs little new reading, which keeps a fork cheap enough for a Sonnet medium reviewer.
The first fanned run (`webster-replan-state`, Discussion rounds 1-4 and Plan round 1) did not work that way; the analysis is in `fan-usage-report`:

- The master forked after reading only the target, instructions and stencil (39-74k context), no code, then told every fork to verify the Decisions' premises against the code.
  In Discussion round 1 the master read 19 code files first, and its forks took 32 turns and 152k fresh tokens; rounds 2-4 took 81-125 fork turns and 435-485k fresh.
- Inheritance through the cache works; cost scales with fork turns, about 3.5-4k fresh tokens per turn.
  32-46% of fork file reads duplicated another fork's.
- Fanned Discussion rounds cost 301-638k fresh against 86-121k for solo Opus rounds; Plan round 1 cost 901k fanned against 202-227k solo.
  The fan raised more MEDIUM and LOW findings, not more BLOCKING ones, and about half of raw fork findings were duplicates across lenses.

The responsible text is `clusterRulesBlock` in `internal/burlerengine/prompt.go`: "After exploring the target fully" never says to read the code the target cites, and "Fetch only what your lens needs" sets no budget.
The `standard` fan also points code lenses at design documents.

## Idea

Revise the fan so it works as designed:

- the master is told every lens of the fan up front, and before it spawns any fork it explores, for each lens, what that lens will need and reads it, so the forks inherit one shared reading instead of each exploring on its own (lyx cannot pick the relevant code itself: Discussion review is exploratory, so the master's judgment chooses what to read);
- the master judges nothing before the fork point: a review written before forking would colour every fork, so the master's own holistic review stays after the forks are spawned;
- each fork is told its context already holds the code, and gets a tool-call budget (about 6) for reading only what is missing; forks spending their budget show in the usage record that the master read too little;
- Discussion-Review and Plan-Review use the `discussion` and `plan` fans instead of `standard`.

Estimate: about 200-250k fresh tokens per round, about 2x a solo Opus round instead of 5-6x.

## Operator decisions (2026-10-10)

- The fan stays off in the hub until this task lands and a fanned run shows the estimate holds.
- Until then Discussion-Review and Plan-Review run solo on `opus[medium]`, and Webster-Review on `sonnet[medium]`.
