```yaml
lens: fix-affected-set
outcome: reported
```

Task: run the fix-affected-set lens over the fixer's change set and rerun each set-premise finding's own search.

One finding, LOW:

- **F2 (LOW).** `internal/battencli/overviewdobby.go:22` declares `dobbyLandNotice = "land_notice"` as a literal. The review named four sites for F2 and this one is outside them. `beamsetting.LandNoticeKey` now exists, but this display-label constant still repeats the string.
  - The constant is a text label for the overview line, not a config key, so it could be argued to be a separate concern.
  - `internal/battencli/overviewdobby.go` was last changed in `299db87cb`, the very change under review, so it is in scope.
  - F2 asks that the key be "declared once, referenced everywhere", and this site declares it a second time.

For the rest, the fix covers every member:

- **F2:**
  - The remaining `pull_request`, `land_notice` and `merge-in` string literals in non-test Go are either unrelated or the intended retired-key constants. The unrelated ones are `fabricengine` merge verbs and `hubissues`'s JSON tag. The intended ones are `boardengine/store.go:254-255`.
  - No `pullRequestOn`, `pullRequestMerge`, `pullRequestMergeIn` or `pullRequestModeMerge` declaration remains.
- **F3:** The only non-test caller of `configengine.TopLevelValue` outside `configengine` is `dobbyengine/tasksetting.go:49`, inside `ownTaskSetting`. Both the lookup and the reinterpretation go through it.
- **F4:** `hubgeom/tasksetting.go` no longer calls `ResolveWorktree`. `hubgeom/spawnorder.go:82` and `hubgeom/nearestlive.go:43` still do. The review declined to cover those, as "other production code calls ResolveWorktree too", so I report no finding.
- **F25:** No `parentPullRequestMode` or `parentMode` remains in any Go file.
- **F16/F17:** No `notify_on_land` remains in any `.md`, `.go` or `.yaml` file outside the intended legacy-key code and comments. The old resolver wording "decides from the parent's beam.yaml" is gone from the docs.

Unchecked from my list: none.

Calls made:
1. Bash: combined greps for the F2 literals, F3 `TopLevelValue` callers, F4 `ResolveWorktree` callers, F25 old names, and F16/F17 stale text. The context did not hold the current whole-repo state of these sets.
2. Bash: read `internal/battencli/overviewdobby.go` lines 15-30 and its call sites and history, to judge whether the `land_notice` literal found in call 1 is a config-key declaration. The fixer never read that file.
