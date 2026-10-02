---
verdict: APPROVED
rationale: "All seven round-1 findings were valid and each is resolved by its own commit (c546edf30, bdd54d3b7, 0ef21c40c, 111d2391e, 1e18719b1, ca0a4385c, e6e2c2e35) with a test where behavior changed; the fix commits introduce no new defect, the plan has no Rename or Delete groups, and the affected packages' unit and integration tests pass at HEAD."
---

Review range: no `_lyx/loom/rework/` exists and `_lyx/shed/webster-recovery/status.json` names `product.parent` = `main`.
So the subject is `git diff $(git merge-base main HEAD)..HEAD`, with merge base `812600b3a`, and it now includes the seven fixer commits.
The round-1 report has no `## Focus departures` section, so there is nothing to ratify or reject.

## Findings

- **F1** is valid and resolved.
  On a `recordFixStrand` error, `attemptIntegrationFix` (`integrationfix.go:155`) now calls `removeStrandIfLive`.
  The error names the GUID and states whether the removal succeeded.
  `TestIntegrationStage_FixAttempt_UnrecordedStrandIsRemoved` forces the vanished-record guard and asserts that exactly that strand is removed.
- **F2** is valid and resolved.
  `TestRebaseline_AfterRecordBatchBoundBegunCard_Regression330` binds a begun card's handle through `RecordBatch` and asserts that the `CardHashes` entry moved.
  It then asserts that a `Rebaseline` naming only the edited pending card succeeds.
- **F3** is valid and resolved.
  The fingerprint, report path, reports dir and stale-report removal now run before `recordFixStart`.
  So the removal also runs on the spent path, but nothing on that path reads the report: `ParseReport(reportPath)` is reached only from `fixStrandVerdict`.
  `TestIntegrationStage_FixAttempt_PreSpawnFailureLeavesAttemptUnspent` pins that the attempt stays unspent and that no strand is started.
- **F4** is valid and resolved.
  `commitsBetween` uses `rev-list --first-parent`, and the merge commit stays listed, so `checkFixCommits`' parent-count refusal is unaffected.
  `TestFixCommitsSince_ListsMergeButNotParentBranchCommits` pins this.
- **F5** is valid and resolved.
  The banner now says that the rendered prompt is handed to the spawn as its launch prompt.
- **F6** is valid and resolved.
  Every rewrapped comment is one sentence per line.
  The remaining in-sentence breaks sit at clause or phrase boundaries, not at a column.
  Some old fixed-column comments in `template_test.go` now have a one-word "five"→"six" edit.
  Rewrapping them would rewrap untouched lines, which the convention forbids, so they are not a defect.
- **F7** is valid and resolved.
  The test's doc now describes begun-batch scoping.
  A `BegunBatchScopesToPendingCards` row has batch 1 begun but not terminal and expects scope `pending` and exit 0.

## Rubric checks

- Per-card mechanical checks: the plan's groups are Create, Edit, Prosa and Uses only, with no Rename or Delete.
  No reference to webstercli's deleted `completedCards` remains.
- `go build ./...` passes.
  So do `go test` and `go test -tags integration` over planglyph, websterengine, webstercli and contracts/stencils at HEAD.
