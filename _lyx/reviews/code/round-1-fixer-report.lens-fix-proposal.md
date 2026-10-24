```yaml
lens: fix-proposal
outcome: reported
```

No findings.

- F4: the new overview sentence ("reed session of the run's parent worktree, the prime's or a beam's, with the prime as its working directory") matches `internal/battencli/cli.go:343` and `:471`.
- F3: the rename only touches fixture names. I built no case where `grouping` or `grouping-two` collides with another slug in `newRenameBoard`. The boardengine gate passed.
- F1 (guard half): `NearestLiveParent` starts from the recorded parent and walks up. I found no case where it fails to match the destination of notices, since it uses the same walk as `parentOrchStrand`.
- F1 (parent-review half): the fixer did not change `wiring.go`, and the new `hubgeom/doc.go` sentence states that the parent review keeps the recorded parent. Switching it to the nearest-live walk would have been wrong. The prime comes back with an empty `Worktree`, which `newParentReviewConfig` reads as "no parent", so the reviewer would be treated as always live.

Calls made:
1. `grep -n -i window internal/battencli/*.go` (filtered): checked the "window opens in the parent worktree's reed session, prime as working directory" wording against the code, which my context did not hold.
