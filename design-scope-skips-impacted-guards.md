# A computed or base gate scope skips the guard tests of an impacted package whose tier it does not run

`scope-skips-impacted-guards` · Note · bug, gate, testkit

## Seen (board tidy, 2026-11-27)

`card-gate-scope-gaps` landed with a bound it did not fix:
a computed or `--base` scope never runs a guard test of an impacted package when that test's tier is one the listed runs skip.
Its change description says this conflicts with `PATTERN-test-speed`.

## Idea

Decide per guard test where it runs, as note `whole-repo-guards-off-gates` does for `cmd/lym`'s guards:
in the task gate, at landing or in the upkeep run, so no guard is left to a scope that skips it.
