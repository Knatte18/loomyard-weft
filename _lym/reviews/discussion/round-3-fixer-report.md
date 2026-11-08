# Round 3 fixer-report — discussion

All five findings are fixed in `_lym/discussion/decision-record.md`.
The support log has a Round 3 entry.

## Changes

- F1 (MEDIUM), fixed as proposed.
  D7 step 4 now says the step joins `internal/gateslot`'s non-tier holders, takes one gate slot after the key's tiers release theirs and holds it to the step's end, and does not bound the wait for it, as D5 states for the verb.
  `redundancy_budget_min` counts from the moment the step holds its slot, builds included.
  Once the budget is spent the step starts no build or test, and each test it did not run is listed as not judged for the spent budget.
  The cost line in the D7 rationale now states the worst case: the budget, plus the longer of one build timeout and one test timeout, plus the grace.
  It names the slot wait as outside that bound.
  The never-changes-the-result rule is unchanged.
- F2 (LOW), fixed as proposed, using the record channel.
  As D5's verb exits it writes an exit record through `internal/state`, at a path `internal/lymdirs` names, holding the verb's process ID and the capped count.
  D6 reads that record once the child it waited on exits, and takes the count only when the record names that child's process ID.
  A refresh that was killed, or whose record is missing, unreadable or names another process, counts as reporting none left.
  The D6 acceptance bullet names the exit record and covers the killed and unmatched cases.
- F3 (LOW), fixed as proposed.
  D1 moves the keep scan (`scanKeeps`, `cmd/testtiming/keep.go`) into `internal/testcover` with the spawn scan.
  The tool keeps listing a kept candidate under Kept with its classification unchanged, while D7 step 2 drops it from judging.
  The D1 rationale names both scans.
  D7 step 2 now refers to D1's keep scan.
- F4 (LOW), fixed as proposed.
  D4 hashes each Go file the package directory tracks at HEAD, which is the set D5 compares and D5's dirty check covers.
- F5 (LOW), fixed as proposed.
  D5 reads the tracked Go files at HEAD through `fabricengine.CodeDirAt` (new) and `fabricengine.CodeFileAt` (new), and reads modified or untracked files through `Fabric.Status`.
  Only the HEAD SHA read, which the pattern exempts, runs outside `internal/fabricengine`.
  A `PATTERN-fabric-boundary` bullet joins Constraints.
  The proposed wording "no git runs outside internal/fabricengine" was qualified to exempt the HEAD SHA read, because D5 and D6 both read HEAD and `PATTERN-fabric-boundary` exempts SHA reads.

## Self-check

- fix-claims, LOW, D7 step 4 claimed that `internal/gateslot`'s doc lists the non-tier holders, and it does not: fixed as proposed. The step now "joins `internal/gateslot`'s non-tier holders".
- fix-claims, LOW, the record did not say how the holder's `HolderEnv` combines with a tier's `env`: fixed. D5 and D7 step 4 now say the builds take `HolderEnv` and each test runs under its tier's `env`.
- fix-consistency, MEDIUM, D5's Bound said "writes nothing but map records": fixed as proposed. The Bound now adds "and its exit record".
- fix-consistency, LOW, no D5 criterion pinned the exit record: fixed as proposed. A D5 acceptance bullet now names the record's process ID and its count, zero included.
- fix-consistency, LOW, the D7 not-judged criterion left out the spent budget: fixed as proposed. The bullet now lists a test left unrun once the budget is spent, and says that no coverage build starts after that.
- fix-proposal, LOW, the rationale's overrun read as adding the grace to the test timeout only: fixed as proposed. It now reads "plus D5's grace either way".
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
