# A gate whose run changes only text no build or test reads, such as docs, passes without running its tiers

`gate-skip-text-only-diff` · Task · enhancement, gate · starting · Seed-Child

## Seen

A darn docs run whose diff is a report under `docs/` still queues at Finalize for the full suite (tiers 0 to 2, scope all), behind every other run's gate calls.
Parent-Align does the same.
The hub's orch rules already say a hand push of `main` that changes only text no build or test reads, such as `docs/`, goes without the test run; the gate does not know that rule.

## Change (operator, 2026-11-28)

- In the gate, not in the landing rows: when the change a gate call covers (the worktree's diff against its parent, committed and uncommitted) touches only files no build or test reads, the call passes without taking a slot or running a tier, and its report says why.
  This holds for every gate key and scope, `scope: all` included, so Finalize and Parent-Align of a docs-only run pass at once in every recipe.
- Which files count as read by no build or test is decided from the repository, never from a guessed list:
  a file that a test embeds, opens or reads as a fixture (for example a doc the PATTERN checks or a doc test reads) counts as read, so a change to it still runs the tiers.
  The writer finds those readers in this repository and states the rule it chose in the gate's docs.
- A diff that touches any other file runs the gate as today.

No separate docs recipe (operator, 2026-11-28): this gate rule covers every recipe.
