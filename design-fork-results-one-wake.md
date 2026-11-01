# A reviewer or fixer wakes once for all its forks, not once per fork

`fork-results-one-wake` · Task · enhancement, burler, stencil · running · PR-Review

## Seen (research, landed 2026-11-01)

`docs/research/review-turn-audit-2026-11-01/report.md`: a Discussion reviewer, Discussion fixer or Plan fixer that forks its lenses wakes once per fork, since each fork's result arrives as its own message, and each wake-up is a turn with no call at the parent's full context.
On three tapestry runs that was 79 turns, 8% to 9% of the three roles' weight and 13% to 19% of their time.
The Plan reviewer works on after its spawn, so its wake-ups are mixed into its other turns.

## Idea

The parent gets every fork's result in one turn: it spawns its forks in one message and is woken once they are all done, rather than once per fork.
First read how the review stencils spawn the forks and how their results reach the parent, then pick the form that keeps the forks running in parallel.
Measured with the scripts of `docs/research/review-turn-audit-2026-11-01`: the `fork-wait` turns per session.
