# Burler as two parallel agents: a review-only reviewer and a fixer that validates

`burler-split` · Note · feature, burler

Idea (operator, 2026-10-03), a large design change for later: split a Burler round into two agents spawned in parallel.

- **A, handler and fixer**, the stronger model.
- **B, reviewer**, writes the review only and never fixes.

Both start at once and orient themselves in the code independently.
B then forks its lens fan as today, and when every fork is done, writes the combined report as Burler does now.
The report goes to A, which has already oriented itself, fixes, and acts as an extra validator that takes B's findings with a grain of salt.

What it buys: A's orientation overlaps B's review instead of following it, and the reviewer never grades its own fixes.
Related: `burler-round-models` (model per round), `review-finding-class` (landed: the judge reads facts, not artifacts).

Models (operator, 2026-10-07): the review half, one agent forking a lens each into a broad review, is good enough on `sonnet[medium]`; the fixer and validator is the one that must be stronger.
Today one agent does both under the single `review` model-spec in `loom.yaml`, so they cannot be set apart; the split gives each half its own model-spec.
Seen 2026-10-07 in `bugfix-fabric`: on `sonnet[medium]` the fix half cleared only the lines a review named, re-broke a line fixed the round before, and circled through rounds 3 and 4 on comment line breaks alone; `review` was raised to `opus[medium]` for both halves as a stopgap.
