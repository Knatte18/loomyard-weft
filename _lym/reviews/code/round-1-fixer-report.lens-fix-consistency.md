```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set for stale twins and sibling disagreements (lens fix-consistency).

Findings:

1. LOW — `internal/gatecli/testmap.go:82` (the `testmap` Long help, unchanged by the fix). It says "A map with no affected test moves to HEAD with every entry; one with affected tests moves to HEAD without them". F8's `PlanUpkeep` change now also drops the entry of a test no longer declared, even on a map with no affected test. F9's change clears a failure recorded at an earlier commit. The help text was left as it was.
   - Evidence: `upkeep.go` now drops with `!declared[test.Name]` in the loop; the `PlanUpkeep` doc comment was updated, but the help text was not.
   - The help text says "every entry" for a map with no affected test. That is false once an entry belongs to a removed test.

No other disagreement found:
- The `testmapaudit.go` doc ("no recorded measurement time counts as the oldest") matches the dropped `BuiltAt` fallback.
- The three dobby texts (`testmap.go`, `config.go`, `template.yaml`) now agree with `doc.go` line 491.
- The stand-in's doc comment names the new `Fails` and `Skips` suffixes.
- The `mapPackage` doc names the probe run.

Tool calls:
1. `grep` for stale phrases ("with every entry", "when its map was built", "counts as measured", "stale packages", "builds at most") across `internal`, `docs/overview.md` and `pattern`. My context lacked any search for these phrases in files the fix did not touch.
