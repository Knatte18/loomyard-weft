```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set through the fix-consistency lens, looking for twins left stale.

**Findings**

1. **LOW** — `docs/overview.md:378`: "seeds absent board files sparse, prunes but never fills a present one, carries a template change into a seed-only board file…".
   - The F9 fix scoped the wording in `internal/configcli/hubwide.go:163` and `internal/configcli/doc.go:92` to bases that are not seed-only. This sentence is the unscoped twin.
   - The same sentence adds that a template change is carried into a seed-only file, so a reader can work out the meaning. "Never fills a present one" still contradicts it on its face.
   - Evidence: `grep -rniE "never fills"` shows this as the one remaining unscoped use outside the two files the fix changed.
   - A fix would read "never fills a present base that is not seed-only".

2. **LOW** — `docs/shared-libs/yamlengine.md` has no mention of `FollowTemplate`, `ErrInvalidBase` or the dropped blank lines.
   - The fix adds `yamlengine.ErrInvalidBase` as a public symbol and a documented re-encode behaviour on `FollowTemplate`.
   - The doc has a "Differs from `Reconcile`" section, so it covers sibling merge functions.
   - Evidence: `grep -n "FollowTemplate\|ErrInvalidBase\|blank line" docs/shared-libs/yamlengine.md` returned nothing.
   - This gap existed before the fix. The fix only adds a symbol to it, so it is borderline for the rubric.

**Checked and consistent**
- The `configsync.go:5` and `configsync.go:934` comments and the `configengine` docs that say "never pruned down to the template" stay true.
- The `follow.go` doc comment now matches the passing 80-column row, which also confirmed that blank lines are dropped.
- The rewritten restore row in `ensure_integration_test.go` has its rename and fixture change in the same row.
- `ErrInvalidBase` is wrapped in `FollowTemplate` and unwrapped in `configsync.go`, and the error rows assert `FileError.Path == basePath`.

**Tool calls**
- `grep` for "never fills" and similar phrases across `.go`, `.md` and `.yaml`: my context held only the two edited comments, not other twins of the wording.
- `grep` of `docs/shared-libs/yamlengine.md` for `FollowTemplate`, `ErrInvalidBase` and "blank line": my context lacked that doc.
