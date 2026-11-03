```yaml
lens: fix-affected-set
outcome: reported
```

No findings from the affected-set lens except the one below. Searches were on F1's stale-description sites, F1's callers and F2's alias lists.

**MEDIUM — `docs/overview.md:423` (the shuttle entry) still describes the removed link derivation.**
- The entry says codex runs "with `-a never -s workspace-write` and the checkout its anchor's `.lym` and `_lym` links resolve into as a writable root".
- The fix replaced that with the told `Dirs.Weft`. The sibling texts it updated were `codexengine.go`, `doc.go` and `summary.md`.
- The entry is the one site of the old description that the fix left unchanged.
- Suggested rewording: "...and the weft checkout it is told through its config Dirs as a writable root".

**Checked, no gap:**
- **F1, callers.** The `shuttleengines.New` callers are `loomcli`, `orchcli`, `shuttlecli`, `pairteardown`, `battencli` and `burlercli`'s hub wiring. All pass `fabricengine.ConfigDirs(...)`, so they get `Weft`.
- **F1, standalone.** `burlercli/wiring.go:211` builds `Dirs{Anchor: res.StateDir}`. It sets no `Weft`, which matches "empty outside a hub".
- **F1, tests.** No test compares `Dirs` by whole-value equality, so the new field breaks none.
- **F2.** No other doc comment enumerates the aliases as the seed template's content. `modelspec/registry.go:13` lists only the Claude aliases, and says it holds exactly those, so it is correct. `contracts/specs/llm-model-spec.md:56` already lists `sol`, `astra` and `luna`.
- **F3.** Not checked. Its fix is rows in the existing tests, with no set to rerun.

Tool calls made:
1. `grep` for "resolve into" and "writable root" across go, md and yaml. I needed the stale sites of the old link description.
2. `grep` over `shuttleengines.New` callers and their `Dirs`. I needed whether every caller reaches `Weft`.
3. `grep` for alias enumerations and `Dirs` equality in tests. I needed the F2 set and whether the new field breaks any comparison.
