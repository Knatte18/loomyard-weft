```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: no findings.

- F1 set (optional `PullRequestDeps` seams): the new table names `ParentReviewWanted`, `AskParent`, `BeamRecord` and `DropBeamRecord`. These match `optionalPullRequestSeams` at `internal/landingshed/prreview.go:208`, and the four are the only func fields in `deps.go:452-461` that the grep matched. The table covers every member of the list, and it no longer depends on the list. `optionalPullRequestSeams` is used only at `prreview.go:216` (`checkWiring`) and no longer in the test.
- F2 set (production comment count): `go run ./tools/codestats -root internal/landingshed` reports 29 production files, 5146 code lines and 2576 comment lines. This matches the 2,576 now in `summary.md:23`. The edit to `prreview_test.go` touches no production file, so the count is unchanged.
- Unchecked: none.

Tool calls:
1. grep for `optionalPullRequestSeams` in `internal/landingshed`, plus a grep of the `PullRequestDeps` struct body. The first part found the users of the list. The second part found nothing because the struct is declared elsewhere (call 4 found it). My context lacked a full usage search of the list.
2. `go run ./tools/codestats` (tail). It gave only the markdown table, so I ran it again for the comment figures.
3. `go run ./tools/codestats` filtered for comment lines. This was a fresh measurement of the figure F2 concerns, which my context lacked.
4. grep for the four seam names in non-test code. It showed their declarations in `deps.go` and so confirmed the table's names match real fields.
