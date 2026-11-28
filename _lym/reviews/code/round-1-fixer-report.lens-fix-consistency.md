```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of this round's change set, looking for twins and cross-references left stale.

**LOW** · changed text: `internal/loomshed/recordseed.go` `SeedParentParam` ("the seed parameter that records a loom run's parent branch"), with its use in `loomSeedFor` and `recordSeedDeps` (F1)
- **Summary:** the other writer of the same seed param still uses the bare literal, so the key is still defined twice.
- **Evidence:** `internal/battencli/wire.go:639` returns `map[string]string{"parent": origin.ParentBranch}, nil` from `childSeedParams`. That map becomes a loom child's seed params. `wire.go` already imports `internal/loomshed` (line 45), so `loomshed.SeedParentParam` needs no new import edge.

**LOW** · changed text: the same constant (F1)
- **Summary:** a refusal that renders the re-seed command still spells the param name in its format string, while Record-Seed's reason renders it from the constant.
- **Evidence:** `internal/loomcli/sharedbootstrap.go:255` has `"... then lym shed seed self --recipe %s --driver %s --param parent=%s"`, the twin of `--param %s=%s` with `SeedParentParam` in `recordseed.go`.

The help and example strings spelling `--param parent=...` (`internal/shedcli/seed.go:137-138`, `internal/loomcli/arm.go:95`) are user-facing examples; I don't count them.

I found no other disagreement. The loom-contract and block rules in `shed-recipe-spec.md`, the package docs, the recipe comments and the change description agree with the change set.

Tool calls (2 of 6):
1. A grep for Record-Seed and support-log mentions, `"parent"` literals, battencli's loomshed import and old contract wording: my context did not show the repo-wide twins.
2. A read of `internal/loomshed/doc.go:118-140` and `internal/battencli/wire.go` around line 639: my context held only the first 25 lines of `doc.go` and not `childSeedParams`.
