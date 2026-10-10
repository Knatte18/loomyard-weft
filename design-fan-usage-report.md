# Compare fanned and solo review rounds from their usage records

`fan-usage-report` · Note · enhancement, burler

From review-lens-fan PR-Gate (2026-10-08): the Discussion and Plan fans ship off by default until round usage records show what they cost and find.

- A small report verb, or a batten run summary, compares the round usage records (`_lyx/reviews/*/round-N-usage`) of fanned and solo segments, so the operator gets the comparison without reading the records by hand.
- `excludedLenses` drops a whole focus file when the previous round's usage record is missing or unreadable and only logs a Warn, so a lost record silently widens a round back to the full fan; the run's facts should show it.

Operator (2026-10-10): the `standard` fan is turned on for Discussion-Review and Plan-Review in the prime loom config (`discussion_fan`, `plan_fan`) as a trial; after a few fanned runs land, an agent compares their round usage records against the solo rounds before this date.

## First fanned run: webster-replan-state Discussion (2026-10-10)

Read from the round-3 escalation, as lens tags in the review text (mentions, not unique findings):

| Round | generic | correctness | error-handling | test-gaps | Findings (B/M/L) |
| --- | --- | --- | --- | --- | --- |
| 1 | 6 | 2 | 4 | 2 | 3/6/5 |
| 2 | 6 | 4 | 5 | 3 | 1/9/9 |
| 3 | 6 | 3 | 1 | 8 | 1/9/6 |

Rounds 1 and 2 spent about 300k and 620k fresh tokens, of which the forks took 50% and 78%.
Blocking findings fell, but the total did not: each round found finer detail in the same decisions, much of round 3's from the `test-gaps` lens.
The loop hit its budget after round 3 and was granted a fourth round.

## Fan trial paused (operator, 2026-10-10)

The fan's design: the master reviewer reads most of what the round needs first, then forks, so each lens fork inherits that reading through the cache and needs little new reading, which keeps a fork cheap enough for a Sonnet medium reviewer.
In the first fanned run the forks' fresh tokens were 50-78% of a round (round 2: 485k over 5 forks, about 97k each) and their cache reads 12M, which suggests each fork read much on its own after the fork point.
The hub turned `discussion_fan` and `plan_fan` off, so Discussion and Plan review run the single `opus[medium]` reviewer again; pairs created before the change keep the fan.
The report compares the fanned rounds of `webster-replan-state` (and `cli-surface`, if its rounds fan) against solo rounds, and, before the fan goes back on, measures from the transcripts how much the master read before forking and what each fork read after it, so the fix targets the right side: the master's pre-read or the fork stencil's licence to read further.

## First analysis (2026-10-10, webster-replan-state Discussion rounds 1-4 and Plan round 1)

- Cause: the master forks after reading only the target, instructions and stencil (39-74k context), no code, then tells every fork to verify the Decisions' premises against the code. In Discussion round 1 the master read 19 code files (~35k) first, and its forks took 32 turns and 152k fresh; rounds 2-4 took 81-125 fork turns and 435-485k fresh.
- Inheritance works (each fork reads its 39-84k prefix from cache); cost scales with fork turns, about 3.5-4k fresh and ~100k cache read per turn. The correctness fork is the heaviest (33-63 turns). 32-46% of fork file reads duplicate another fork's.
- Fanned Discussion rounds cost 301-638k fresh against 86-121k for solo Opus rounds; Plan round 1 cost 901k fanned against 202-227k solo. About half of raw fork findings were duplicates across lenses; the fan raised more MEDIUM and LOW findings, not more BLOCKING ones.
- Responsible text: `clusterRulesBlock` in `internal/burlerengine/prompt.go`. "After exploring the target fully" never says to read the code the target cites, "Fetch only what your lens needs" sets no budget, and the master's own holistic review runs after the fork point, where no fork inherits it. The `standard` fan also points code lenses at design documents.
- Proposed fixes: the master reads all cited code before spawning but judges nothing there, since a review before the fork point would colour every fork, and each fork gets a tool-call cap (about 6); Discussion and Plan review use the existing `discussion` and `plan` fans (4 forks) instead of `standard`. Estimate: about 200-250k fresh per round, about 2x solo Opus instead of 5-6x.
