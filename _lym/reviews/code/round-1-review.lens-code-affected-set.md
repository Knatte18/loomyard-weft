```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff forces outside its own lines.

**Per change, the set found:**

1. **Pawl recipe gains a `Preflight` row and entry**
   - Covered:
     - `contracts/recipes/pawl-recipe.yaml`, header and rows.
     - `internal/loomrecipe/pawlrecipe_test.go`, row table and `Entry`.
     - `internal/loomrecipe/doc.go`, the rows list.
     - `internal/loomcli/doc.go`, the rows list.
     - `internal/loomcli/wiring_beam_test.go`, the routing rows.
   - `Env.Cwd` is set unconditionally at `internal/loomcli/wiring.go:556`, before the pawl branch at line 833. `preflightEntry` (`internal/shedrecipe/entries_simple.go:23`) therefore gets the absolute `Cwd` it demands.
   - The `Preflight` registry row is reused, so `PATTERN-shed-recipe-registry` is met.
   - Not covered: none found. The other recipe-row listings (`internal/battenshed/doc.go`, `internal/pawlengine/doc.go`, `internal/orchcli/doc.go`, `internal/beamcli/*`, `docs/overview.md:271,450`) describe only Worktree-Create, Beam-Up or the producers `pawlengine` owns, not the recipe's first rows, so they need no edit.

2. **`Preflight` joins `holdFreeProducers`**
   - Covered:
     - `internal/boardengine/runlock.go`, the var and its comment.
     - `internal/boardengine/doc.go:227`.
     - `internal/battencli/dobbyreads.go:179`, the `runTakesNoSlot` comment.
     - `internal/dobbyengine/doc.go:104`.
     - `internal/boardengine/runlock_test.go:366`, the new `BeamAtPreflight` row.
   - `RunHolds` is the only reader of `holdFreeProducers`. Its callers (`runTakesNoSlot`, `internal/loomcli/peers.go:104`, and the beam-fork and layer readers) pick up the change through it.
   - The lift applies only to a beam project (`runlock.go:54`), so a loom or darn task at `Preflight` keeps its hold, as D2 bounds it.
   - Not covered: none found.

3. **Declared PATTERN changes**: the directive declares none, and `PATTERN.md` and `pattern/` are untouched. Nothing is missing.

4. **Commit message**: it carries no `Pattern-Departure:` trailer, and none is needed.

**Findings:** none.

**Tool calls (1):** a grep across the repo for `Beam-Up` and `Beam-Await` outside the diffed packages, to find unlisted docs, stencils or tests. The manifest did not cover those files.
