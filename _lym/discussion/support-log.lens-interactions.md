```yaml
lens: interactions
outcome: reported
```

Interactions lens on the decision record: pairs of decisions over shared state, checked through their normal, failure and restart paths, then each criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 × D2 (same spec builder, same Title source): held. loomcli takes the recipe from its seed through `c.recipe` and orchcli reads it with `shedrun.ReadSeed`, so the title comes from one source.
- D1 × D3 (`Title` on the beside spec): held. Before D3, reed ignores `Title` on an own window; after D3 it names the window.
- D2 × D3 (moved strand's window name): held, including the empty-title path where the window is named by its role.
- D2 step 2 × loomcli `resolveStatusStrandAction` (status sidecar): held on the happy path. On the fallback path the sidecar names a GUID that no longer exists, so the next loom start adds the strand with the beside spec; that also holds.
- D2 × loomcli `ensureStatusStrand` running at the same time (same strands, same sidecar, different locks): broken in a narrow interleaving (finding 2).
- D2 step 1 × step 3 (overview slot ordering): held. `ReplaceStrand` removes the old strand, kills its panes and only then adds the new spec, so the overview slot is free before the runs strand is added. Step 3 failing after step 1 succeeded is not covered (finding 3).
- D2 fallback × D4 (window the overview line points at): broken (finding 1).
- D3 × the existing overview `Title` for the runs strand (`beam: &lt;slug&gt;`): held. The overview naming is unchanged.
- D4 × D1/D3 (window name in the line vs the actual window name): held when the seed reads. With no seed, D4 draws no line and D3 falls back to the role name, so they stay consistent.

**Criteria and constraints:** all trace to the decisions they cite. The AC "`statusStrandAddSpec` … returns the same spec orchcli's move uses" means `statusStrandAddSpec` needs the recipe as an input, which D1 implies but does not say (finding 4). The constraints hold against D1–D4.

**Findings**

1. **MEDIUM — D2 fallback × D4.** The overview line can point at a window that does not exist.
   - When D2's move fails, `loom-status` is replaced by the runs strand, so there is no checklist window.
   - The same happens in the older-session case under Open risks.
   - D4 still draws "window `pawl`" whenever the seed and status file are live, because it keys on the run's state, not on whether the strand exists.
   - The record says nothing about this, so the line misleads the operator in exactly the failure case. Either key the line's window pointer on the `loom-status` strand being present, or say that the line names `lym loom start` when the strand is absent.

2. **LOW — D2 × loomcli `ensureStatusStrand` race.** A concurrent move and loom start can leave the session with no beam overview.
   - The two paths use different locks: orch's `start.lock` and loom's bootstrap lock.
   - An operator's `lym loom start` during Beam-Up can compute the overview spec before the runs strand exists. If it then re-adds `loom-status` in the overview window after orchcli's step 1, step 3's runs add is refused.
   - D2's bound ("never fails `start`; every failure is a warning") covers this. Its fallback covers only a step-1 failure, so the session ends with no runs overview and a warning that names only the generic `lym beam start` way forward.
   - Fix: state that a step-3 failure is a warning naming `lym beam start`, which this race resolves, or note the race as accepted.

3. **LOW — D2 steps 1–3.** No failure rule for step 3 after step 1 succeeded.
   - The checklist has already moved, so nothing is lost. But D2 names a fallback only for step 1 and a warning only for step 2.
   - An AC for "step 3 fails → warning, `loom-status` stays in its own window" would pin this.

4. **LOW — D1 × AC6.** `statusStrandAddSpec`'s inputs gain the recipe implicitly.
   - D1 says the beside spec carries the run's recipe as `Title`. Today `statusStrandAddSpec` takes only the command and the strands.
   - AC6 compares its output to orchcli's move, which needs the same title input.
   - The plan writer can derive this, so it is LOW: state that the shared builder takes the recipe.

**Tool calls**
- Read `ReplaceStrand` in `internal/reedengine/replace.go` to check whether replacing an overview strand with an own-window spec frees the overview slot before step 3. My context had no reed replace semantics.
