```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set for sibling or twin text left inconsistent (fix-consistency lens).

**Findings**

1. **LOW** — `internal/pawlengine/doc.go:29`, twin of the F7 change to `retiredmove.go`.
   - Summary: the package doc describes the move as carrying "the run's scratch pull request ... unless GitHub reports it closed or merged". It does not say that a repeated move names the pull request the landing record already holds.
   - Evidence: `MoveOffRetired`'s own doc comment and `carryPullRequest` were updated. `doc.go:29` was not.
   - The doc's wording is also looser than the code. The code treats only an open pull request, or one whose state does not read, as carried.
   - A reader of the package doc would not learn of the new repeat behaviour.

2. **LOW** — `internal/loomcli/landingdeps.go`, the comment on `CommitsMissing` at about line 377, next to the F2 override.
   - Summary: the comment still reads "CommitsMissing reads over the task pair ... `f.CommitLog(merged, task)`". For a pawl run that seam is now replaced by `BeamCommitsPast`.
   - Evidence: the override sits about 290 lines above, in the `if recipe == shedrun.RecipePawl` block. The seam's comment gives no hint that a beam run does not use `CommitLog`. The fixer planned this comment edit but did not make it.

The other sibling texts agree with the changes.
- `internal/landingshed/doc.go:267` ("an open pull request in the run's own record is merged as it stands") matches the F1 behaviour, since a closed or merged one now goes to the resolver.
- `internal/loomcli/doc.go:46` still holds.
- `orch-role.md:143` and the existing bullets are consistent with the scoped wording.

**Calls made**
1. `grep` for "as it stands", "whatever the resolver", `MoveOffRetired`, `CommitsMissing` and `CommitLog` across docs. My context held the code but no package docs that might repeat the changed behaviour.
