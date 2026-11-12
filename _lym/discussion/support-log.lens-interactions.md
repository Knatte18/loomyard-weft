```yaml
lens: interactions
outcome: reported
```

**Findings**

1. **MEDIUM — D9 × D4 (restart path).** D9 removes `<bolt>/.lym/testcover/` only "after the commit", and imports only "while the prime's `_lym/testcover/` holds no map". If a pass is killed after the copy and before the commit, the prime's directory holds (uncommitted) maps, so the next pass imports nothing — and never reaches D9's removal step, so the old bolt directory stays forever. Evidence: D9 "After the commit the pass removes…" + "only while the prime's directory holds no map"; D4 says the next pass commits a killed pass's leftovers, but nothing re-arms the removal. Fix: make the removal conditional on the old directory existing, not on the import having run in this pass.

2. **MEDIUM — D9 × D6 / D2 (format and disk claim).** D9 copies `map.json` files across byte-for-byte (indented); D6 compacts a map only when its content changes. So the imported set is committed at ~56 MB and every fork per D2 checks out ~56 MB, not the "about 41 MB" D2 and Open risks state, until each map happens to change. Fix: D9 writes the imported maps compact, or correct the size figures.

3. **MEDIUM — D7 × D4 × Constraint `PATTERN-fabric-boundary`.** A pass run by hand in a task worktree (D7) commits that task's weft and spawns a records push carrying the task's code branch (D4) outside any run boundary, possibly while the task's own run is mid-step. The constraint says "the weft commit is Go calling the engine at a boundary the run controls". The record does not say how a hand-run pass in a live task fits that, or whether the pass waits on or refuses under the run lock.

4. **LOW — D8 × D9 × D1 (empty state).** With upkeep off by default (D8), the import runs only once someone runs a pass in the prime. Until then every D1 reader finds the prime's `_lym/testcover/` empty → `MissNoMap` for every package → the gate scope runs whole packages, and pairs forked in that window inherit no maps (D2). The record doesn't name this window.

5. **LOW — D5 (claim conflation).** "A machine whose stamp differs … never rewrites the maps, since D5's refusal stops its pass" holds only for a non-owner machine. An owner machine whose stamp changes (a Go upgrade) gets `MissEnvironment` on all 211 maps and rebuilds and commits them all — about 14 MB compressed in one commit. The intent is right, but the sentence ties stamp difference to ownership; state the owner-upgrade cost.

6. **LOW — Acceptance D1 × D9.** The D1 criterion says none reads `<bolt>/.lym/testcover/`, but D9's import reads it. The criterion needs "bar D9's one import".

**Pairs checked**

- D4 × D6 (commit set vs unchanged maps): held.
- D4 × D9 (import commit and removal order): broken on restart (finding 1).
- D6 × D9 (compact form vs copied maps): broken on size claim (finding 2).
- D2 × D6 (inherited maps at mixed old commits): held — the child branch carries the parent's warp history, so every map's commit stays reachable.
- D2 × D4 (fork mid-pass): held, stated.
- D3 × D6 (parent pass after landing, unchanged maps keep old commit): held — `Files` and `Commit` both stay old, consistent.
- D3 × D7 (task's own pass, then landing/archive): held.
- D4 × D7 (commit/push in task outside run): open (finding 3).
- D5 × D7 (task owner read through parent): held.
- D5 × D9 (import on owner machine only): held — a non-owner's old bolt dir is never imported/removed, harmless.
- D5 × weft pull (only owner writes map files): held.
- D5 × stamp claim: wording broken (finding 5).
- D6 × dobby work-left check (exit record `planned_head`): held.
- D6 × audit rewrites (`measured_at` changes on audited entry): held, small churn.
- D8 × D9 × D1 (default off, empty prime maps): gap (finding 4).
- D1 × D9 vs acceptance wording: inconsistent (finding 6).
- D4 vs acceptance criterion (commit holds only `_lym/testcover/` paths; exit record/locks untracked): held.
- PATTERN-owner-machine change × D7: held (task reads parent's owner).

**Tool calls:** none — my context already held the code facts this lens needed (`PlanUpkeep`, the fork behaviour, the weft commit API).
