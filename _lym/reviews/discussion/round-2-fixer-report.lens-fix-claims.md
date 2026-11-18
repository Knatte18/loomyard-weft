```yaml
lens: fix-claims
outcome: reported
```

Task: check every claim the fix added or rewrote against the code at this worktree's HEAD.

**1. MEDIUM. The comparison to `SkipForked` implies a hook that cannot carry the bypass at Darn-Write.**
- Changed text: the new D3 bullet, "the wrap spawns no cold fixer for a result whose findings are the reach note, as it spawns none for a forked failure its `SkipForked` closure reports."
- What is wrong: at HEAD the wrap checks `SkipForked` only on the forked closure. Darn-Write wires no `Forked` and no `SkipForked`. A plan that follows the comparison by setting `SkipForked` would leave the bypass dead on Darn-Write's verify gate, so the cold fixer would still take the reach note. The bullet does not say the wrap needs a new check on the entry's own `Gate` closure, or on both closures.
- Evidence:
  - `internal/gatefixer/rowgate.go:161`: the skip fires only when `forked && w.rf.SkipForked != nil`. `spawning(original.Gate, false)` at `rowgate.go:105` never consults it.
  - `internal/shedrecipe/entries_darnwrite.go:27,41`: Darn-Write builds its request through `splitGateFixer`/`applyGateFixer`, and the file sets no `Forked` or `SkipForked`. In the tree, only `entries_planwrite.go:64` sets `SkipForked`.

**Claims that hold:**
- "Darn-Write's verify entry carries a cold fixer, `darn.gate_fixer`": confirmed by `contracts/recipes/darn-recipe.yaml` (the verify entry's `fixer: darn.gate_fixer`) and `internal/darnengine/template.yaml` (`gate_fixer: sonnet[medium]`).
- The wrap hands the failure to the fixer before the writer hears of it: confirmed at `rowgate.go:153-164`.
- "On an arrival that spawns a fixer, the wrap runs the inner gate twice": confirmed. The first run is at `rowgate.go:153`, and the second goes through `spawnAndJudgeAgain` → `waitThenRun` at `rowgate.go:218`.
- The F2 edits (Goal, D10, D11, D12, Constraints, criteria, support log) make no new claim about code; they restate the parent's ruling.

Tool calls:
- grep of `internal/loomcli/gatefixer.go` and the `shedrecipe` files: my context lacked how Darn-Write wires `Forked`/`SkipForked` into the wrap.
- grep of `entries_darnwrite.go` and read of `entries_gatefixer.go`: my context lacked the fields of Darn-Write's fixer request.
