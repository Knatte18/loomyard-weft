# Fixer-report, round 1

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  `originTaskBranch` and `originPairBranch` in `internal/fabricengine/pairbranch.go` fall back to the new `originOtherPrefixBranch` when the current prefix's name and the bare slug both miss.
  It takes the one origin branch ending in `/<slug>` whose records branch the weft origin holds, and errors naming every candidate when more than one matches.
  `Add` in `internal/fabricengine/add.go` takes the warp branch from `PairTaskBranchOrOrigin` under `AdoptOnly`, and keeps `BranchPrefix+slug` for a new pair.
  `TestBranchPrefixChange_PairMadeUnderAnotherMachinesPrefix` in `pairbranch_integration_test.go` covers a pair pushed under `earlier/`, resolved and re-adopted under `hanf/`, and the two-candidate refusal.
  `_lym/landing/summary.md` "Open risks" now describes what the code does.
  Doc comments of `Add`, `PairTaskBranchOrOrigin`, `TaskBranchOnOrigin` and the `doc.go` lookup paragraph name the new fallback.
- **F2 (MEDIUM), fixed as proposed.**
  `slugOfBranch` trims the current prefix when the branch carries it, and otherwise everything through the last `/`.
  A new `TestTaskBranchSlugs` row reads `other-machine/adopt-fix-x-t5` under prefix `hanf/` as slug `adopt-fix-x-t5`.
- **Self-check change (fix-proposal lens).**
  `keptReason` in `cleanupremotewarp.go` counts an archive tag as weft evidence only for a branch the current machine's prefix names (`pairBranchNames`).
  Without it, `slugOfBranch` turning `feature/foo` into `foo` let a tag of pair `foo` mark an unrelated branch as fabric-managed.
  No test covers it: `keptReason` is unexported and no existing export reaches it.

Commits: `7d6d8cfc` (F1, F2) and the follow-up for the self-check change; the F2 edits share `pairbranch.go` with F1's, so one commit holds both.
The summary edit sits under `_lym` and is left uncommitted by the discipline rule.
Gate: `lym gate test --tier 1 ./internal/fabricengine` passes after the last edit.

A gate pass over the added comments then rewrapped four of them at sentence boundaries, in `doc.go`, `pairbranch.go` (two) and `pairbranch_integration_test.go`.
No code changed in that repair.

A later gate run failed `slackengine.TestRunRelay/WatchThatFailsToOpenFallsBackToTheOnDiskCheckWithItsOwnBackoff` (`relay_test.go:1091`, pending waits `[8s 10s]`).
This round changes nothing under `internal/slackengine`, and `lym gate test ./internal/slackengine` passes in isolation (every test of the package), so the failure is a flake of that test under load.
No change was made for it.

## Disputed

- **Self-check, fix-consistency #1 and fix-code-correctness #2 (`RecordsBranchLive`, `remoteleftover.go:84`): premise false.**
  `RecordsBranchLive` is documented as the test a raddle start's pair creation adopts a weft branch by (`boardcli/cli.go:82`), which is a new-pair `Add` without `AdoptOnly`.
  That `Add` builds `t.cfg.BranchPrefix + slug`, so the two agree; only the re-adopt `Add` reads the pair's own branch.

## Deferred

- **Self-check, fix-affected-set: `originreads.go:41,100` (`OriginRunStatus` and the origin-branch name check).**
  Both read remote-tracking refs locally and match `pairBranchNames` only.
  Making them find another machine's prefix needs a new lookup over remote-tracking refs, a design the review did not propose and that goes beyond F1's named sites; the parent should decide whether to extend F1 to them.
- `addparent.go:53` is not a gap: it names a parent pair whose worktree stands, and its branch is read from that worktree's HEAD.
- `battencli/wire.go:1170` builds a branch for a new pair, where the current prefix is correct.
- **fix-code-correctness #1 and fix-proposal #2 (`originTaskBranch` now errors on an unresolvable records root or two candidates for `TaskBranchOnOrigin` callers).**
  Left as the review's fix names `originTaskBranch`; the two new error cases are the same refusals `originPairBranch` already gives.

## Rulings

- **F3 (decision-class), parent ruling #1575.**
  The ruling records that the review of the other personal keys is done and none moves to the machine layer.
  It asks for no change to the target: `_lym/landing/summary.md` already states these reasons under "The other personal keys stay where they are", so it is redundant for the code and the summary.
  Nothing was changed for it, and I did not flag the parent, since the ruling is correct and already held.

## Self-check

- fix-code-correctness, MEDIUM, `TaskBranchOnOrigin` errors and cost: left as is, see Deferred.
- fix-code-correctness, MEDIUM, `RecordsBranchLive` twin: disputed, premise false.
- fix-consistency, MEDIUM, `RecordsBranchLive` twin: disputed, premise false.
- fix-consistency, MEDIUM, `Add` doc comment: fixed as proposed.
- fix-consistency, LOW, `doc.go`, `PairTaskBranchOrOrigin` and `TaskBranchOnOrigin` docs: fixed as proposed.
- fix-consistency, LOW, `wire.go:1170`: no change, a create path.
- fix-proposal, MEDIUM, `slugOfBranch` and archive-tag evidence in `keptReason`: fixed with my own change, which guards the tag path rather than leaving `slugOfBranch` unchanged.
- fix-proposal, LOW, ambiguity error for task-branch-only callers: left as is, see Deferred.
- fix-affected-set, MEDIUM, `originreads.go` and `addparent.go`: `originreads.go` deferred, `addparent.go` no change.
- fix-affected-set, LOW, `remoteleftover.go:84` and `wire.go:1170`: disputed and no change as above.
