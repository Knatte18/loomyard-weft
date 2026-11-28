```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun each set-shaped finding's search and check the fix covers every member (fix-affected-set).

**Unchecked findings:** none. F2 and F4 each name a single case or line, so there was no set to rerun for them.

**no findings**

- **F1 (empty-id guards):** non-test code compares or guards the recorded ids in three places: `bouncerstalemate.go:55`, `bouncerverify.go:185` and `roundcontext.go:298`. The first two decide whether a round left the target unchanged, which is F1's premise, and each now has a new row. The third only gates the changes file, so it is outside F1.
- **F3 ("worktree-relative" wording):** the search over the review and recipe code found no remaining statement that calls the overlay target paths or changes-file paths worktree-relative. The remaining hits (`shed-recipe-spec.md:144`, `geometry.go:9`, `roundcheck.go:84`, `bouncer.go:76`) describe other things.

**Outside my lens, not a fix finding:** `roundcheck.go:283-294` strips `WorktreeRoot` from a finding's location, then looks it up in `Changed`. On a hub anchored to a subpath, overlay `Changed` paths are anchor-relative, so an absolute finding location may not match. This comes from the task's earlier change of snapshot root, not from this round's fix. I did not trace how `ctx.WorktreeRoot` is set or run the case.

Tool calls:
1. `git grep` of the `TargetID` and `PostFixTargetID` uses: the context showed only some of the files that use them.
2. `git grep` of worktree-relative wording in the review and recipe code and contracts: the earlier search did not cover that wording across all of them.
3. `git grep` of `WorktreeRoot` and `Changed` in `roundcheck.go` and `burlerreview.go`: the context showed only a short span of `roundcheck.go`.
