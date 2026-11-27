```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set check of the fix change set.

Unchecked: none.

no findings

- **F1:** the new rows cover the reachable base and module refusals. The "neither tier0 nor tier1" message can't be reached through red-first, since `TierList` refuses first (per the `config_test.go` rows).
- **F3:** at base, maps lived in the board dir's `.lym/testcover` and each worktree anchor's `.lym/testcover`. The summary names both.
- **F5:** the non-tier `HolderEnv` callers are cardverify, driftengine, planglyph and `cmd/testtiming`, and `doc.go` names all four. `PATTERN-test-speed.md:71` and `PATTERN-gate-slot.md:15` both point at that list and now agree with it.
- **F6:** no "re-run mark" or `RerunMark` text remains. The other `Rerun` hits are unrelated fields.

Tool calls:
1. `git grep` for re-run mark and `RerunMark` text: my context had no repo-wide search for F6.
2. `git grep` for slot holders in Go and markdown: the parent's search covered only Go acquire and `HolderEnv` sites, not the markdown references.
3. `git grep` on `main` for map directory sites: my context didn't show where the base code wrote the maps.
