```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the sets behind F3, F4 and F1 (sibling serial subtests, stale `resetHub` pre-flight wording, summary claims).

no findings

- F4 (stale `resetHub` attribution): the search for `resetHub` near "pre-flight" in `.go` and `.md` files matched a doc under `docs/research` that is unrelated. In `internal/fabricengine`, the only remaining `resetHub` pre-flight attribution was `livestate_doc_test.go:183–190`, which the fix updated. `livestate_matrix_test.go:312` already names `checkResetTarget`. The other "pre-flight" mentions in the livestate files describe pull, remove, Reconcile and RefusedBefore, not the reset guard. The generic "the pre-flight itself is load-bearing" at line 190 still reads correctly.
- F3 (serial subtests): `shortnamebinding_clone_integration_test.go` has four serial subtests: `SecondCloneDerivesShortname`, `DisagreeingShortnameRefuses`, `ResetWithDisagreeingShortnameKeepsHub` (with its inner subtests) and `ResetKeepsRecord`. The moved comment at line 71 sits above the first and says "the subtests below". The three subtests of `TestCloneHub_BoundWeftWithoutRecord` (line 142) are not covered by it. Those two subtests are serial too, but the review did not name them and the fix did not change them, so they are out of scope.
- F1 (summary claims): the summary's new `--subpath` claims match `resetsubpathcheck.go` and the tests added in the fix. The staged check runs before the reset. `fetchWeftBranchTip` is called from `recordedAnchorOnWeft`, which supports the "shared with the probe" statement. The four tests it lists exist.

Further tool calls (3):
1. Searched `resetHub` near "pre-flight" across `.go` and `.md` files, to find stale attributions beyond `livestate_doc_test.go`; my context lacked a repo-wide search.
2. Read the `t.Run` and `t.Parallel` lines of `shortnamebinding_clone_integration_test.go`, to confirm the full set of serial subtests; my context lacked the post-edit line positions. The `git diff --stat` in the same call failed on a pathspec error and returned nothing.
3. Searched the `livestate_*_test.go` files for "pre-flight" and `looksLikeHub`, to find other `checkResetTarget`-related wording; my context lacked that cross-file view.
