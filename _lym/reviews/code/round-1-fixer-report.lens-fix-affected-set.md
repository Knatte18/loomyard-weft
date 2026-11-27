```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens, rerunning the searches behind the findings whose premise is a set.

**Findings left unchecked:** none.

**no findings**

What the searches covered:
- **F1 and F8 (sites that describe prior work or scratch files):** the hits I checked agree with the changed behaviour. I found no stale site in `internal`, `contracts` or the searched docs. `internal/darnengine/spec_test.go` only passes `PriorWork` through, so it needs no change.
- **F3 (seam lists):** `Darn.BranchCommits` is now in `darnSeams`, both `wire_seams_test.go` maps, the `shedrecipe` required-seam list and its construction-failure rows.
- Outside this change set: `darnSeams` also lacks `Darn.CheckRecord`. This predates the fix and the review did not name it, so it is not a finding.

Tool calls:
1. `grep` for prior-work, scratch-listing and interrupted-spawn wording across `internal`, `contracts` and the docs, because my context did not show every site that describes them.
2. `grep` for the `Darn.*` seam names across Go files, because my context did not show every seam list.
