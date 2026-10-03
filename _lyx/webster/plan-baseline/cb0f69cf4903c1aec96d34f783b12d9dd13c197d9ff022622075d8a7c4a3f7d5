# Card 15 — the webster verify gate on Merriam's strand

**Create:**
- `internal/websterengine/verifygate.go`
- `internal/websterengine/verifygate_test.go`
- `internal/websterengine/verifygate_integration_test.go`

**Edit:**
- `internal/websterengine/runlevel.go`
- `internal/websterengine/gitwrap.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/runlevel_test.go`
- `internal/shedrecipe/entries_simple.go`
- `internal/loomcli/wiring.go`
- `internal/loomcli/wiring_test.go`

**Uses:**
- `internal/verifytree/verify.go`
- `internal/websterengine/verifygatereport.go`

**Intent:**
D3: the plan-level verify becomes a must-pass shuttle gate entry on Merriam's strand, and a failure goes back to Merriam's live session.
`Run` stops calling `runIntegrationStage`;
the fixer side (card 16) replaces the integration prompt, and card 17 deletes the dead code.

**The gate (`verifygate.go`).**
`NewVerifyGate` is the exported constructor over the told `Geometry`, the attempt cap, the `ParentBranchFunc` and the friction dir, so the parity test (card 18) drives the closure `Run` uses.
`Run` hands `StartMaster` `deps.Gate` plus one entry named `verify` from `NewVerifyGate`, must-pass, with `Attempts` from `Config.VerifyGateAttempts` (card 10).
`Run` refuses a `deps.Gate` that already names `verify`, so a recipe row cannot add a second entry beside it.
The shipped Webster row supplies no gates of its own, so Merriam's spec is that one entry.
`StartMaster` is Merriam's only start: a resumed run's `reclaimEntryTimeStrands` stops the recorded Master strand and `Run` starts a fresh one, so there is no attach path, and every Merriam carries the spec.

The closure, evaluated at each gated Done arrival, does in order:

1. When `outcome.yaml` names an outcome other than `done`, it passes without verifying, so a stuck or paused outcome ends the run as today.
2. `verifytree.DirtyPaths` on `Geom.WorktreeRoot`; a dirty tree fails with the paths as findings and runs no verify.
3. Once a pre-fix head is recorded, the fixer commit check:
   every commit from the pre-fix head to HEAD on the first-parent chain is a non-merge commit or a clean parent merge as `gitwrap.go`'s `parentMergeRejection` accepts it (no merge accepted in standalone mode, where `ParentBranch` is nil);
   anything else fails `Terminal`, naming the commit and the rejection reason.
4. With no `## verify:` section it passes with a warning.
5. Otherwise `verifytree.Verify` with `verifytree.NewPaths(Geom.WorktreeRoot, Geom.VerifyDir)`, site label `webster gate` and the attempt number.
   A pass or skip passes, and the verify records the tree.
   A failure is parsed with `parseVerifyFailures` from the log, then rerun once;
   a rerun pass passes the gate with the existing flaky warning, summary section and friction note (`triageWarnings`, `AppendIntegrationTriage` and `writeTriageFrictionNote` over the flaky identities only), which `Run` applies after `Wait`.
6. A failure that survives the rerun returns findings.

The first failed evaluation of any kind, a dirty tree at step 2 or a surviving failure at step 6, records the HEAD as the pre-fix head, so every commit a fixer makes after it is checked at the next arrival.
Attempt counts and the pre-fix head live in the closure's memory for one shuttle run, as the decision record's constraint states.

**The commit walk (`gitwrap.go`).**
`parentMergeRejection` and `resolveParentTips` take a `*gitrepo.Repo`, and `cmd/lyx/rawgitmutation_test.go` bans `gitrepo.New(` from every websterengine production file but `gitwrap.go`, so the walk lives in `gitwrap.go` and `verifygate.go` never constructs a repo.
`gitwrap.go` gains two helpers:
`commitsSince(worktree, base string) ([]string, error)` returns `commitsBetween` from base to HEAD, oldest first, and feeds both the check and the report's fix commits;
`fixCommitRejection(worktree, base string, parentBranch ParentBranchFunc) (commit, reason string, err error)` walks that list and returns the first commit that is neither a non-merge commit (`CommitParents`) nor a clean parent merge (`parentMergeRejection`, with the tips from `resolveParentTips` resolved on the first merge it meets), or empty strings when every commit qualifies.
A `resolveParentTips` error rejects the merge with that error as the reason, so a nil `ParentBranch` accepts no merge.
`verifygate.go` reaches both through a closure seam, so `verifygate_test.go` stays tier 1.

**Findings and the report.**
On every failed evaluation the closure builds a `VerifyGateReport` (card 14) with the attempt, the cap, the failing identities (or the dirty paths), the log path, the card hint over `accumulatedCardSHAs` and `commitChangedPaths`, and the fix commits from the pre-fix head to HEAD (`commitsSince`).
It writes the report to `VerifyGateReportPath(Geom.ReportsDir)` and returns `renderVerifyGateFindings` as the findings.
`Run` removes a stale report at entry.
No bisect, no baseline run and no detached checkout of the live worktree.

**Run's outcome mapping.**
After `Wait`, a done result whose `Gate` did not pass maps to outcome `stuck`, with a reason naming the failing identities read from the report and the attempts spent, or the `Terminal` failure's `Gate.Reason`.

**Wiring.**
`loomcli/wiring.go` fills `RunDeps.ParentBranch` from the pair's fabric origin, as `webstercli` already does, so a clean parent merge during a fix passes the commit check under loom.
`shedrecipe`'s `websterEntry` doc stops saying no Webster validator exists: `Run` adds the `verify` entry itself and refuses one from the row.
`doc.go` gains the gate and findings sections.

**Tests.**
`verifygate_test.go` (closure seams over verify, dirty and commit checks):
a non-`done` outcome passes without verifying;
a flaky failure that passes on rerun passes with the flaky warning;
a failed evaluation writes `verify-gate.yaml` with the failing identities, the card hint and the fix commits;
a dirty-tree failure records the pre-fix head, so the next arrival runs the commit check.
`verifygate_integration_test.go` (real scratch repo):
a dirty tree at a done arrival fails with the dirty paths and runs no verify;
a merge commit above the pre-fix head other than a clean parent merge fails `Terminal`, and a clean parent merge there does not, both through the real `fixCommitRejection`.
`runlevel_test.go` (fake starter driving the captured gate):
a fake Merriam whose first done arrival fails verify and whose second passes ends `done` with one re-prompt, and the findings name the failing identity and the card whose commit touched its package;
an exhausted gate ends `stuck` with a reason naming the failing identities;
a `deps.Gate` already naming `verify` is refused.
`wiring_test.go`: loom's wired `RunDeps.ParentBranch` is non-nil and returns the pair's parent branch.

**ImpactSummary:** webster's plan-level verify moves from a post-exit Go stage to a gate on Merriam's live session, so only a webster run's end changes: a red tree now re-prompts Merriam and ends stuck at the cap instead of being bisected; loom's Webster row gains a parent branch it did not pass before, and no other row changes; the two new `gitwrap.go` helpers have no caller but the gate.
