# Fixer-report, webster round 1

## Changes

- F1 (fixed as proposed, commit `74cc68a5`): `TestWireJunctions_ScratchLinkRefusedAfterClassify` gains `WeftWorktreeGoneBeforeAdoption` and `WeftWorktreeGoneBeforeRestore`.
  The existing `WeftWorktreeGoneDuringTheStep` row and both new rows share the helper `removeWeftWorktreeDuringTrackedCheck`.
  The adoption row asserts the root is still a real directory holding its note and the target is absent.
  The restore row asserts the target is not recreated and the root link is left as it was.
- F2 (fixed as proposed, commit `74cc68a5`): new `TestScratchNameAtSubpathAnchor` with a `.scratch` link inside the `backend` anchor, built by `linkAnchoredScratch`.
  `UnwireRemovesItAndKeepsTheRootLink` asserts an applied `Unwire` removes it, lists it in `JunctionsRemoved` and leaves the root `.scratch` link.
  `RemoveCountsIt` asserts `Remove` records `link_removed` at it and reports `LinksRemoved` 3.
  The link points at the same name under the weft anchor, as the ownership gate requires.
- F3 (fixed as proposed, commit `8a95db98`): the `WireJunctionsWith` doc comment names the scratch step, that it never fails wiring, and that it reports through `logger.Warn`.
- F4 (fixed as proposed, commit `9c9ad466`): `wireScratchLink` names the warp worktree path when `mutateGitExclude` returns an empty exclude path.

`lyx gate test --tags integration ./internal/fabricengine` is green.

## Self-check

- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-consistency, LOW, the `WireJunctionsWith` doc names only `logger.Warn` while Reconcile reports the same step through the pair's `Detail`: disputed, premise false.
- fix-affected-set, LOW, no subpath-anchor test through the third caller of `withoutRootScratchLink` (the Reconcile stale-removal scan): disputed, every fix worse.

## Disputed

- Self-check fix-consistency LOW: premise false.
  The new sentence describes `WireJunctionsWith`'s own route, and that function warns through `logger.Warn` (`junction.go:101`).
  Reconcile's `Detail` is that caller's reporting, not this function's.
- Self-check fix-affected-set LOW: every fix worse.
  The anchor guard is one branch of the shared `withoutRootScratchLink`, which both new tests already fail without.
  A Reconcile row at a subpath anchor would re-pin the same branch, a redundant test under `PATTERN-test-economy`.

## Deferred

None.
