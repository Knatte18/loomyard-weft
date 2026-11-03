# Fixer-report, code review round 1

## Changes

- **F1 — fixed as proposed** (commit `16093168`).
  `TestSeedDiscussedRecord_Real` in `internal/loomcli/discussionrecord_integration_test.go` gained three rows:
  a flagged entry whose record file was removed from the board directory (false, nil, nothing written);
  a discussed entry whose pair already holds a different record and a support log (record overwritten, support log kept byte for byte, both committed);
  a regular file where the discussion directory belongs (false and an error naming the directory).
- **F2 — fixed differently** (commits `3a5c264e`, `c9a98832`).
  The proposed unit test of `commitStatusDeps` for a motif and a tapestry `loomCLI` is in `TestRecipeKeyedWiring` (`internal/loomcli/recipekeyed_test.go`).
  The proposed test did not pin the call sites, since a call site that bypasses the method passes it.
  I added `TestWireLightweight_CommitStatusWritesFinishedStatus` (`internal/loomcli/commitstatus_integration_test.go`), which drives the seam `wireLightweight` installs through a final `done` transition against a real pair and its board, and asserts a motif run leaves `draft` and a tapestry run keeps its last running status.
  That pins the call site at `wiring.go:319`.
  The call site in `wire` (`wiring.go:957`) is not pinned: driving the seam `wire` installs needs the full config load and the real commit path.
  It is the same expression as the pinned one.
  The `Env.SeedDiscussedRecord` half is disputed below.
- **F3 — fixed as proposed** (commit `1adb4ab2`).
  `docs/research/weaving-glossary.md` lists motif in the recipe names of the searched-against list and in shed's recipe list, and the motif row says a board task may name it, it runs Discussion alone, and tapestry runs it inline as its Motif block.
- **F4 — fixed differently** (commit `aef4c9ab`).
  The proposed test in `internal/battenshed` that drives a Finalize-less child through Land cannot exercise the premise.
  `landProducer.Call` (`internal/battenshed/land.go`) reads the parent branch, the prime's branch and dirt, the pull, the deploy and the board prune, and never the child's rows or Finalize, so a Finalize-less child changes nothing it does.
  Worktree-Teardown's consumer of the child's end state is the board-claim settlement in `internal/pairteardown/claim.go`.
  A finished motif run leaves its entry as `draft`, which `boardengine.IsRunStatus` rejects, so `settleClaim` returns `claimKeep` and the entry is left untouched.
  No code path assumes a Finalize landing commit, so no code change was made.
  `internal/pairteardown/claim_test.go` pins that behaviour for a draft entry whether batten never seeded, finished or paused.
  The `notify_on_land` notice is posted from `recordsStopped` in `land.go`, which also reads no child state.

## Gate repair

The verify gate failed `TestTierTags_OneTierPerTestFile`: `TestSettleClaim_DraftEntryOfFinalizeLessRun` in `internal/pairteardown/claim_test.go` did not open with `tiergate.Require`.
It now opens with `tiergate.Require(t, 0)`.

## Disputed

- **F2, the `Env.SeedDiscussedRecord` half — premise false.**
  `TestWire_EverySeamFilled` (`internal/loomcli/wiring_seams_test.go:159`) wires a tapestry run and fails for any nil `shedrecipe.Env` field not listed in `intentionallyNil`.
  `SeedDiscussedRecord` is in neither that map nor `envkit`'s `nilLegal`, so deleting `wiring.go:882` fails that test already.

## Deferred

Nothing deferred.

## Self-check

- fix-code-correctness, MEDIUM: the `commitStatusDeps` assertion pins the method and not the call sites at `wiring.go:307` and `:326`. Fixed with my own change: the integration test above pins the `wireLightweight` call site, and the `wire` call site stays unpinned, as stated under F2.
- fix-code-correctness, no finding: the F1 rows, the F2 Env half, F4 and F3 hold; the F4 test pins current behaviour and cannot fail with a defect in place, because the code was already correct (taken: the review's premise there was a missing test).
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings; it confirmed the `wire` call site and the glossary lists as covered.

No fork returned without a report.
