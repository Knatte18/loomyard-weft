# Webster round 1 — fixer report

Every finding in `round-1-review.md` is fixed, one commit each.
After the last commit, the plan's full `## verify:` command exits 0.

## Fixed

- **F1** (`c546edf30`): when `recordFixStrand` fails, `attemptIntegrationFix` now removes the started strand through `removeStrandIfLive`.
  The returned error names the strand GUID, and it also says so when the removal itself fails.
  New test: `TestIntegrationStage_FixAttempt_UnrecordedStrandIsRemoved`, which uses a new `onStart` hook on the fake starter.
- **F2** (`bdd54d3b7`): added `TestRebaseline_AfterRecordBatchBoundBegunCard_Regression330`.
  A record-batch handle binding rewrites begun card 1 and moves its `CardHashes` entry.
  A `Rebaseline` naming only the edited pending card then succeeds.
  The test fails with the hash move disabled.
- **F3** (`0ef21c40c`): the plan fingerprint, the report path, the reports dir and the stale-report removal now run before `recordFixStart`, so a failure in any of them no longer spends the attempt.
  New test: `TestIntegrationStage_FixAttempt_PreSpawnFailureLeavesAttemptUnspent`.
- **F4** (`111d2391e`): `commitsBetween` walks `rev-list --first-parent`.
  A merge commit in the range is still listed, so `checkFixCommits`' parent-count refusal is unchanged.
  New test: `TestFixCommitsSince_ListsMergeButNotParentBranchCommits`.
- **F5** (`1e18719b1`): the integration-fix stencil banner now says the prompt goes to the strand's spawn as its launch prompt, with no prompt file written.
- **F6** (`ca0a4385c`): rewrapped the listed new and changed comments to semantic line breaks.
  The files are `validate.go`, `runlevel.go`, `beginbatch.go`, `planglyph.go`, `render.go`, `wiring.go`, `planglyph_test.go` and `template_test.go`.
  A diff scan also found the doc on `RenderIntegrationFixPrompt` and the doc on the card-4 fork-prompt test, so those were rewrapped too.
  The `render.go` header no longer carries an asset count.
- **F7** (`e6e2c2e35`): `TestValidateCmd_ScopeFollowsRunProgress`'s doc now describes begun-batch scoping.
  The test also has a `BegunBatchScopesToPendingCards` row: batch 1 is begun but not terminal, and the expected result is scope `pending` and exit 0.

## Deferred

None.
