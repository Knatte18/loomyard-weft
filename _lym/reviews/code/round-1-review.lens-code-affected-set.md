```yaml
lens: code-affected-set
outcome: reported
```

**Per-change list: set found, covered or not**

1. `validateLabels(t, prior, v)` signature
   - Set: the call in `store.go:428` and `labels_test.go:91`.
   - Covered.
2. `applyDraftRule(loaded, after, released)` signature
   - Set: the one call in `board.go:144`.
   - Covered.
3. `checkNewSlugs(..., store, storeErr)` signature
   - Set: the call in `board.go:168`. I found no other caller.
   - Covered.
4. `RecordWriteDeps.ReadStored` (new seam)
   - Set: the real wiring (`loomcli/wiring.go:426`), the `shedrecipe` constructor non-nil check, the `shedrecipe/fixture_test.go` and `envkit` fixtures, `wiring_darn_test.go`, and the `wire_seams_test.go` loom-only list.
   - Covered.
5. BeamUp's `HandOff`, `Notify` and `Paths.HandOffMarker`
   - Set: `pawlengine/deps.go:94`, `shedrecipe/entries_pawl.go`, `hubgeom/dobbygeom.go:82`, `pawlwiring.go:150`, and the three fixtures.
   - Covered.
6. Declared PATTERN lines in `PATTERN.md`
   - Set: all ten declared entries land with their declared text.
   - Covered.
7. Background files for the trimmed entries
   - Set: `pattern/PATTERN-test-speed.md`, `PATTERN-remote-refs-lock.md` and `PATTERN-shed-recipe-registry.md` already hold the moved rules.
   - `PATTERN-run-usage-record` is an overview-only entry with no background link.
   - Covered.
8. `PATTERN-sole-parsers.md` background
   - Set: it gains a `raddleparser` section.
   - Covered.
9. `docs/overview.md` rows for `_lym/raddle/` and `_lym/usage/`
   - Set: the artifact table at line 142 and the prose at lines 188–189.
   - Not covered (finding 1).
10. Commit messages
    - Set: none of the range's commit messages carry a `Pattern-Departure:` trailer, and the diff differs from the decision record only where it records the departure, in "Added after Discussion". Nothing to raise.

**Findings**

1. **MEDIUM, class `consistency`:** `docs/overview.md:142` still describes `_lym/raddle/` as "Raddle documentation (the raddle nav-doc overlay)", and line 188 still says it has "no `_raddle` junction" and points at "the board's `raddle` note". This diff makes `_lym/raddle/` hold a raddle run's proposals, approval, state, staged store and part records (`lymdirs.RaddleDirName`, `internal/lymdirs/dirs.go:52`). The diff edits other parts of this file but leaves this row and the prose describing the same directory with the old meaning. The `_lym/` artifact table also has no row for the new `_lym/usage/` directory (`lymdirs.UsageDirName`, `dirs.go:50`), although the table lists the other durable `_lym` subtrees. The grep of the file shows no `_lym/usage` mention. Evidence: `docs/overview.md:142`, `docs/overview.md:188`, `internal/lymdirs/dirs.go:49-54`.

I found no other finding from this lens.

**Tool calls (5)**
1. `grep` for callers of the changed signatures and the `ReadStored` wiring. My context held the definitions but not the callers or fixtures.
2. `grep` of the `pattern/` background files for the moved rules. The diff touched only two background files, so I had to check the other three held the rules already.
3. `grep` for `HandOffMarker`, `HandOff:` and raddle producer references. I needed the BeamUp wiring and fixtures one hop beyond the manifest.
4. `grep` for the `lymdirs` names, the interrupt-policy entries and any literal `_lym/usage` or `_lym/raddle` paths outside `lymdirs`. This checked `PATTERN-durable-vs-ephemeral-state`.
5. `grep` and `sed` of `docs/overview.md` plus its diff. I needed to see whether the artifact table and prose changed with the raddle change.
