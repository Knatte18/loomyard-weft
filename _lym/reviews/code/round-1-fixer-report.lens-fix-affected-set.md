```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Checked fix-affected-set for F1 and F2; no finding left unchecked.

- F1 (set: sentences that still tie the Prio column to project sections). I searched `Prio` and `project sections` across `internal/boardengine` and `docs`. The corrected sentence at `doc.go:75` was the only one. The other Prio mentions already limit the column to Notes rows or Notes tables: `doc.go:193` and `render.go:7`. `render.go:403` is where the column gets added.
- F2 (set: places where a local `heading` shadows the package function). I searched `internal/boardengine/*.go` for assignments, parameters or other declarations named `heading`. There were no matches, so the renamed `sectionName` in `entrySections` was the only site. The one non-test hit for `heading` was the function itself.

Tool calls used (2 of 6):
1. `grep -rnE "Prio|project sections"` over `internal/boardengine` and `docs`: re-ran F1's search, which my context didn't hold.
2. `grep -nE` for `heading` declarations in `internal/boardengine/*.go`: re-ran F2's search, which my context didn't hold.
