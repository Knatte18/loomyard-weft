# Composite scenario tests instead of one test per corner

`composite-tests` · Note · enhancement

Idea (operator, 2026-10-06): after the prune tasks and `test-tiers`, the suite can shrink further by testing less of every corner on its own and more through composite tests that cover several behaviours in one run.

The prune tasks only delete what coverage shows redundant and fold near-identical tests into tables; they keep the one-behaviour-per-test shape.
Rough estimate at the time: pruning cuts test count by 10–20 % and integration wall time by 5–15 %, while the heavy cost sits in git and tmux fixtures each test builds on its own.

A composite test builds one expensive fixture (a hub, a pair, a reed server) and walks a scenario through several assertions, replacing a family of tests that each rebuild the same fixture to check one step.

Open:

- Where the line goes between a composite scenario and a test that names one failure: a composite test must still say which step broke.
- Which packages gain most: those whose tests each rebuild a git hub or a tmux server (`fabricengine`, `reedcli`, `battencli`, `loomcli`).
- How it fits `PATTERN-test-economy`, which today favours one table-driven test per kind of item.
