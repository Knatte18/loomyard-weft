```yaml
lens: fix-consistency
outcome: reported
```

No findings.

The remaining "marker directory" hits are in other packages (shuttleengine, pawlengine, loomcli, gatecli), where they refer to real marker directories. No sibling of the three reworded comments in `start.go` or the `DobbyGeometry` doc in `dobbygeom.go` was left stale.

Tool calls:
1. `grep` for "marker directory", "marker dir" and `MarkerDir` across the repo, plus `git diff HEAD~3` on `start.go` and `dobbygeom.go`. My context did not show the surviving stale-wording hits or the committed diff.
