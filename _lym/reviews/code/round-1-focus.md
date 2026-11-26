---
round: 1
focus:
  - "Entries marked unresolved now hold Blocks, Funcs and outcomes. Check every reader of a coverage map entry, including those outside the diff (redundancy, mutation, `lym gate redundant`, map refresh, merge and carry-over of entries, block and function tables, time estimates, the map run's failed and passed_alone listings): does each still behave correctly, and does any use coverage without going through `Measured` or `HoldsCoverage` as it should?"
  - "The map run now executes unresolved tests, which the scan says may run code in another process, in a call that builds no `lym` and sets no `gateslot.PrebuiltLymEnv`. Settle what such a run can do: what it spawns, writes or leaves behind, how failures and time bounds are handled, and whether the repository's rules and the docs on the map run and refresh budget still hold."
  - "Check `MapFormatVersion` 3 and `indexVersion` 2 end to end: what a reader or writer does with a map or index of the other version, in shared and worktree maps and across binaries. Compare that with what `_lym/landing/summary.md` says about existing maps and with the door and blast radius under its `## Merge danger`."
  - "Check `mayRunUnseen`, `unseenCause` and the caveat texts against each combination of spawn verdict and outcome, including an unresolved or out-of-process entry with `OutcomeNoCoverage` or another outcome. Is each statement a caveat makes true of every test it names, and do the doc comments and the change description agree with what is listed?"
  - "The board entry asks that the caveat name the reason wherever the map cannot see a call, for that function. Enumerate the ways a tier1 call can still be missing from the map (for example a test not in the map or changed since it, a package with no map or a failed build, a test in another package, a function outside the block or function tables) and check whether each yields a caveat that names it."
  - "Confirm that the diff removes the miss the entry reports, a function a tier1 test calls directly reading as unreached, for the named kinds of call and not only the fixture's. Check that the `Root-Cause:` trailer names the defect the fix removes."
  - "Test economy and coverage: which added behaviour has no test asserting it (the unmapped-tier caveat, the step wiring of `caveats`, the spawn verdict on a run entry with an outcome, the version bumps), and do the changed tests still check what their names say?"
---

Seed focus for round 1, drawn from the board entry, the change description and a read of the diff.
Each entry is a question to settle, not a verdict.
