```yaml
lens: fix-consistency
outcome: reported
```

No findings.

The module docs still agree with the change set:
- `internal/fabricengine/doc.go` describes Bolt, the prime and the local beams in worktree list order, and says a side that stops never keeps the next one from running. That matches `PullHub`, including a beam whose origin record cannot be read now reporting `stopped`. It also says a pair whose records side could not reach origin is stopped, which still holds.
- `internal/fabriccli/doc.go` describes `sides`, the `fabric-pull` run id and the error envelope. These match `hubpull.go` and the new CLI test.
- The `Bolt.Pull` doc comment no longer names `PullThenCommitWritten`. The engine `doc.go` paragraph still says "the pull-first path", but that is package-level prose, not a symbol comment.
- Restoring the plain `pull` help bullets keeps `weft_verbs.go` in step with `fabricengine/doc.go`, which keeps the longer wording. The `--hub` paragraph is the only addition.
- No caller of the old `localBeamLocations` is left: `grep -rn localBeamLocations internal` found none. Every use is `localBeamCandidates`.

One gap, below the rubric's threshold: the `SkipGit` behaviour is stated in the `PullHub` comment but not in the engine `doc.go` "The hub pull" paragraph or the `--hub` help text. A reader would not be misled by the omission.

Further tool calls made:
1. `git diff e71d1b1a~3 HEAD -- internal/fabricengine/doc.go internal/fabriccli/doc.go`, plus `grep` for `localBeamLocations` and a `sed` of `hubpull.go`: my context lacked the two `doc.go` files and any other callers of the renamed function.
