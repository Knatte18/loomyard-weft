```yaml
lens: fix-proposal
outcome: reported
```

No findings.

- **F1:** The decoy `archive/root/<tip12>` tag does catch a prefix regression. `ownerArchiveSlug` also filters on `OwnerArchiveTagPrefix`, so the test fails only if that constant collides with `ArchiveTagPrefix`. That is the regression the review named, and the test still asserts that no local `archive/*` tag is fetched.
- **F2:** With the defect in place the row fails. A records tag judged against the code branch name finds no such branch on the records origin, because the records branch carries `RecordsBranchName`. The tip is then read as absent and the tag is listed, contradicting `wantListed: false`.
- **F3:** Prime and beam rows assert different `subject` values (`root` and `b`), so a wrong `Worktree` name fails one of them. Both rows end `owned_here`, which does not run the adopt path, but the review asked only for that.
- **F4 and F5:** The new rows go through `hereRequest` and `mergeRequest`. Dropping `NoOrch` from the merge removes `--no-orch` from the spawned args, so the F4 row fails. The F5 refusal rows hit the guards before any pull, and the PullBolt row asserts one pull and no record.
- **F6:** The pairs are passed as `root` then `b`, so the expected order `b/code, root/code, root/records` exercises the sort. The `b` row also takes the non-prime branch path.
- **F7 and F9:** These are text-only changes with no interaction.
- **F8:** `ErrRemoteNotConfigured` is wrapped alongside the go-git error with two `%w`, so `errors.Is` still matches `errRemoteHasNoURLs`.
  - `archiveOwnerTip` got the same change, which the review did not name.
  - Its doc comment ("a repository with no origin remote … archive nothing") still holds.
  - A real config-read failure now fails that pair instead of skipping it. That matches the finding's premise.
  - The gate passed on tiers 0 and 1, so no fixture breaks.

Tool calls made: none. My context already held every changed file and the gate result.
