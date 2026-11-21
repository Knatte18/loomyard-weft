```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the fixer's as-proposed fixes (F1–F4, F6–F10) by building counter-cases. I made no tool calls and judged from context alone.

**1. LOW. F3, `sideRecordedMergeGone` / `sideConcludeAlreadyLanded` (`mergeguards.go`, `mergelifecycle.go`).**
- Removing `|| sourceSHA == ""` turns a record with an empty source SHA into a raw git error instead of a refusal.
- `IsAncestor("", head)` fails with `gitrepo: invalid SHA`.
- Evidence: the first tier-1 run failed `TestMergeCrucible_ContinueRefusesAttemptThatNeverReachedBothSides` with `classify recorded merge state: gitrepo: invalid SHA; want *fabricengine.MergeGuardError`.
- The fixer papered over it by adding `WarpSource` to the test fixture.
- The case is reachable by any warp-side record that has `WarpOutcome` staged or conflicted and no recorded source. `mergestate.go` still calls a source "best-effort" in the weft's case.
- The fix removes the exemption without making the `IsAncestor` path fail loudly or informatively. It is a MEDIUM if any production write path can leave `WarpSource` empty.

**2. MEDIUM. F4, reworded refusal in `clone.go`, the `found stale ... marker` error.**
- The text says "in an existing hub's Bolt worktree commit `.lym-anchor` ... at the root of the records branch `main-weft`".
- The Bolt worktree is the board checkout on weft `main`, not the prime's records branch. Committing `.lym-anchor` there lands on the wrong branch, and the retried clone will not adopt it.
- The operator needs the prime's weft worktree, or a clone of the records branch.
- The extra "remove `.fabric-anchor` from the board worktree" step is unnecessary once the prime branch carries `.lym-anchor`, because the adopt path never reaches the stale check.
- `TestCloneHub_StaleFabricAnchorHardErrors` pushes to the bare remote directly. It therefore never exercises the location the message names.
- Asserting the literal `records branch main-weft` also hard-codes the branch name into the test.

**3. LOW. F8, `docs/overview.md`, the lines left after the fixer's deletion at the old 1521–1525.**
- The fixer deleted the old lines 1521–1525.
- The old line 1526, "It drops the retired `merge_*` perch keys and `rework`.", and the lines after it were kept.
- They are in the same migration list. The fixer's own F10 premise says reconcile now keeps retired keys the template lacks, except where `Prune` applies.
- Those kept sentences are likely stale, and the paragraph now opens mid-list.

**4. LOW. F1, `//testtiming:keep` reasons.**
- The reviewer's fix says to compare each candidate's assertions with its covering tests first.
- The fixer's reasons for `TestDraftStatus` ("no covering test asserts"), `TestPromotePreservesRecipeAndLabels` and `TestBoardRefusesUnconfiguredLabel` were not checked against the covering tests.
- The fixer read only the first ~60 lines of `draft_test.go`, and never read the covering tests.
- A keep marker on a test that duplicates its covering tests is the defect F1 names, so the claim is unverified.

**5. LOW. F2, `tapestryPlanKeys` categorised `guard` in `loomengine/config.go`.**
- `ValidateLocal` refuses a retired key of any category other than none, so profile blocks and local files are refused with the new text. That matches `shedOnlyKeys`.
- This hub's live `profiles.yaml` still holds `loom.plan_*` in its `tapestry` blocks. A tuned value there now fails `checkProfiles` and Seed-Child with a pointer to tapestry.
- That is better than before, but the fix does not move the hub's data.
- It rests on the premise that the hub-wide reconcile drops only values equal to the template base.

No other as-proposed fix (F6, F7, F9, F10) yielded a breaking case.

Calls made: none; the context already held every file as changed.
