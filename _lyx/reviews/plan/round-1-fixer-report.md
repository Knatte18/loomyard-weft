# Fixer report: plan round 1

Every finding is fixed.
`lyx loom validate-plan` answers `ok: true` after the edits, with only informational `create-new-unit` findings.

## Changes

- **F1** (`09-pairteardown-composite.md`): the gone-worktree quiet rule is now a stated, bounded exception.
  "Gone" is defined as the task worktree path not being a registered linked worktree, card 2's `stray_path` test.
  The Bound section says what the exception lets through (a possibly live driver's session ended by name with no wait, on both caller paths) and why it is safe: quiet cannot be probed, a refusal would name an impossible remedy, nothing the driver writes can reach the archive, and D6 commits the sibling's records first.
  The ordinary bound is narrowed to "with the task worktree present".
  Card 13's module doc now covers the exception.
- **F2** (`09-pairteardown-composite.md`): an absent reed session now satisfies only the strand clause; the run-lock probe still runs whenever the task worktree is present.
- **F3** (`11-batten-teardown-through-composite.md`): `Shutdown` passes `EndSession`'s error through `teardownRefusal`.
  `teardownRefusal` drops its `taskAnchor` parameter and names the pair by slug and sibling worktree, since the probe also refuses with the task worktree gone.
  A new `lifecycle_integration_test.go` case (added to the Edit list) checks that the Shutdown-path refusal wraps `ErrPairSiblingDirty`, never names `--force`, and leaves the pair in place.
  The card's `ImpactSummary` now covers the signature change.
- **F4** (`11-batten-teardown-through-composite.md`): the Bound now states the two inherited deletions (the landed task branch on origin under card 3's gate, and finishing a half-removed pair under card 2's bound), and the gone-worktree driver end from card 9.
- **F5** (`06-reed-end-session-by-name.md`): adds a Bound stating what `EndSessionByName` skips and why, which session and server it can reach, and that `internal/pairteardown` is its one caller, only for a gone task worktree.
- **F6** (`11-batten-teardown-through-composite.md`): `internal/battencli#taskWorktreePresent` joins the Edit list, and its doc comment drops the teardown and `PairSiblingRemnant` claims.
  The comment above `Teardown:` in `wire` is rewritten for the composite.
- **F7** (`09-pairteardown-composite.md`): a new integration case commits a stop report, marks the stub driver strand retiring during the wait, and asserts that `Run` proceeds and the archive tag contains the report.
  This needs `Engine.MarkRetiring`, so card 9 lists it in Uses, and card 7 now declares it as a `plan:` Create handle (it already described the method).
- **F8** (`07-reed-retiring-mark.md`): `reedCLI.statusCmd` joins the Edit list, and each strand in the `lyx reed status` envelope gains a `retiring` key, with a test through the card's fake engine seam.
  The `ImpactSummary` now covers it.

## Deferred

None.
