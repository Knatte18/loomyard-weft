```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens on the seed-starts-at-start diff.

**Per change, the set found:**

1. Seed writes an empty `current_producer`.
   - `internal/loomshed/seed_test.go` is updated.
   - `internal/loomcli/smoke_test.go` is updated.
   - `internal/loomengine/anomaly.go`'s `DetectCrashResume` doc comment is updated.
   - `contracts/specs/loom-status-spec.md` is updated at lines 34, 40 and 145.
   - `internal/loomrecipe/recipe_test.go` loses `TestRecipe_SeedAndResumeRowNamesExist`, which only guarded the removed hard-coded name; the `loomshed.go` constants comment is adjusted to match.
   - Covered.

2. Step reads an empty `current_producer` as the first row (`RowAt`).
   - `internal/shedengine/doc.go` and the `Status` field comment are updated.
   - New step tests cover the start and pause-before cases.
   - `ProgressAt` is updated and has a new row in `progress_test.go`.
   - The `ProgressAt` callers `internal/shedverbs/human.go:55` and `internal/shedverbs/step.go:266` therefore show the first row before the first pass.
   - Covered.

3. Readers of a seeded status that compare `current_producer` directly.
   - Only `internal/loomcli/decision.go:399` (`recordWritePending`) could be affected, and it already treats an empty row as pending, so it is unaffected.
   - Verbs that move a run (`Goto`, `Relaunch`) refuse a running status, which is what a fresh seed is.
   - The driver's start check, stop keys and tackler goto rows only act on halted states, which always carry a named row after a persist.
   - `internal/beamcli/land.go:148` and `internal/battenshed/innerrun_owner.go` compare against named rows, and an empty value fails that comparison as it should.
   - `RenameProducers` passes an empty value through unchanged.
   - Covered.

4. Stale statements about the seed naming Preflight.
   - Searched `contracts`, `internal`, `PATTERN.md` and `pattern/` for such statements and found none left outside historical `docs/research`.
   - The remaining uses of `NamePreflight` in tests are recipe-row and rename fixtures, not seed assertions.
   - Covered.

5. PATTERN, stencils and commit trailers.
   - The diff declares no PATTERN change.
   - No stencil is touched.
   - The three commits carry no `Pattern-Departure` trailer, and nothing needs one.
   - Covered.

**Findings:** none.

**Tool calls (1 of 6):** one combined grep and sed. It looked for remaining seed/Preflight assertions in the wiring, bootstrap and resume tests, in the docs and in PATTERN, and read the `recordWritePending` body in `decision.go`. My context lacked those lines.
