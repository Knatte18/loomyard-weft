---
round: 2
focus:
  - "Map format version (`testcover.MapFormatVersion`, `reachStep.staleLinked` in `internal/gatecli/reach.go`, `testreach.Fresh`, mutation's `freshMaps`): is a worktree or shared map an older binary wrote, still fresh by file hashes, handled so its unrun unresolved entries cannot keep a function reading as unreached? Check every reader that chooses between or reuses maps, including `worktreeMapWins` in `internal/testreach`, and whether remapping or preferring one map over another loses coverage or leaves a stale map in use."
  - "Reach caveats across packages (`reachStep.caveats`, `testreach.Reader.UnseenTests` and `UnseenLinkerTests`): does a changed function, new or not, get a caveat naming why for an unresolved, out-of-process, failed, timed-out or skipped test of its own package or of a package linking it? Check the linker set, ordering, duplicates and what `lym gate reach` prints as well as the gate test step."
  - "Reach caveat causes and tiers (`unseenCause`, `unmappedTierNames`, `namedTests` in `internal/gatecli/reach.go`): is each unseen test named under the cause that put it in the list, and is a test with no `tiergate.Require` or of a tier the hub does not declare reported? Check that the gate config the tier caveat reads reaches both the gate test step and the `lym gate reach` verb."
  - "Test coverage of the above (`internal/gatecli/reach_integration_test.go`, `internal/gatecli/reach_test.go`, `internal/testreach/reader_test.go`): would a test fail if the caveat assembly dropped an unseen-test source or misnamed a tier, or if an index of another `indexVersion` were read as current? Check each new assertion for a tautological expected value."
  - "Prose against code: the `judgeRuns` doc comment in `internal/gatecli/redundancy.go`, the `lym gate mutate` docs on a task package that is not refreshed, and `_lym/landing/summary.md` including `## Merge danger` (shared maps and index that other worktrees' binaries read, and what an older binary does with each): does each claim match what the diff does?"
---

The entries point at the sites where round 1 left open questions.
