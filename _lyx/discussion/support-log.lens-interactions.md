```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of decisions in the record that acts on the same state, with failure and restart paths included.

**Findings**

1. **BLOCKING — D8 against D2, and D7.** Migration writes the wrong anchor for a markerless pair whose parent or prime has been re-anchored.
   - D8 writes a markerless pair's marker from its parent's anchor first, then the prime's, then the Bolt legacy value.
   - D2 resolves that same pair to the Bolt legacy value, and its weft content and junctions sit there.
   - Case: the parent is migrated, then `lyx fabric anchor --set B` runs on it, then reconcile migrates the markerless child.
   - The child is committed with marker `B` while its files and junctions sit at `A`. Reconcile then wires at `B` over content that is not there, and the junctions at `A` are left behind.
   - Fix: while a Bolt legacy marker exists, the migration value must be what D2 resolves now. The parent's or prime's anchor is only right once the Bolt marker is gone.

2. **MEDIUM — D5's markerless adopt against D2 and D7.** Same flaw on the adopt path.
   - A markerless branch's content sits at the hub's original anchor.
   - "Parent's anchor, else the prime's" is wrong once either has been re-anchored, and it disagrees with D2's Bolt fallback during the migration window.
   - The value must be what D2 resolves while Bolt still has a marker. After Bolt removal there is no correct source at all, so this belongs in Open risks rather than being stated as a guarantee.

3. **MEDIUM — D7's refusal order against D7 step 1.** A dobby can start a run after the run check passes.
   - All refusals run "before anything is touched", and only then does step 1 end the dobbies.
   - A dobby pass that spawns `lyx batten run` between the check and step 1 leaves a live run whose records move under it. For the prime, batten records sit under the prime's anchor.
   - Fix: end the dobbies first, then check for runs; on a refusal, restart the dobbies.

4. **MEDIUM — D7 step 3 failure restore against D1.** The restore does not cover the marker file.
   - D1 reads the marker from the working tree.
   - D7 says a failure before the commit "restores `A/.lyx` and the index", but not the marker file in the working tree.
   - If the marker was written with `B` and the commit failed, every lyx call resolves `B` while the files sit at `A`.
   - The restore must cover the working-tree marker too; alternatively, write the marker last, right before the commit.

5. **MEDIUM — D7's re-run against D8.** A re-run cannot recover `A` for a worktree that had no marker.
   - The re-run reads `A` from "the marker's previous committed value".
   - Case 1: a worktree that was markerless before D7 (unmigrated, so `A` came from D2's fallback) has no previous value.
   - Case 2: when the previous marker commit is D8's migration commit, there is nothing before it.
   - The rule must fall back to D2's resolution when there is no previous committed value.

6. **MEDIUM — D10's pull rewire against D7's refusals and steps 1 and 7.** The pull rewire has no stated bound.
   - The rewire reuses D7 steps 2 and 4–6 but none of D7's refusals: a live reed session, a held driver lock, a live run.
   - It also skips the dobby end and restart, so a beam's or the prime's dobby keeps the old geometry.
   - The record must state the rewire's bound: either the same refusals (leave the pair unwired and point to reconcile or a re-run), or the same dobby end and restart.

7. **MEDIUM — D6 against D5.** The order of the origin-record write and the move is unspecified.
   - D5 writes the origin record at the new pair's Location, which is `Y` when `--anchor` is given.
   - D6 moves the content from the parent's anchor to `Y`.
   - If the origin record lands at `Y/_lyx/fabric/origin.json` before the move, the move of `_lyx` collides with an existing `Y/_lyx`.
   - The record should say the move runs before any write at the new Location.

8. **LOW — D7 recovery against the "converges" acceptance criterion.** After a crash, reconcile only fixes the junctions.
   - Reconcile wires at the marker's anchor, but leaves the launchers at `A`, stale workspace files, the unpushed commit and the dobbies ended in step 1.
   - Only a re-run of the verb finishes these.
   - The criterion's "and by `lyx fabric reconcile`" overstates this; it should say reconcile converges the junctions and the re-run finishes the rest, the dobby restart included.

9. **LOW — D3 against D2.** A pair whose weft is gone gets the wrong anchor.
   - With the weft directory absent, the constructor falls back to the Bolt marker or `"."`. For a re-anchored pair that is wrong, for example in `pairteardown` with the worktree gone.
   - This is harmless only if no consumer acts on anchored paths of a gone pair. That is unstated.

10. **LOW — D7 on the prime against the retired-portal assumption.** After the prime is re-anchored, the sweep keys on the new anchor, so retired portals still under the old anchor are never swept. The record accepts this, but does not state it.

**Pairs checked**

- D1 working-tree read / D7 step-3 failure: broken (finding 4).
- D1 / D10 checkout reading the target marker: held.
- D2 empty-as-absent / D8 per-pair empty refusal: held. Outside reconcile an empty marker silently resolves as absent, as today.
- D2 fallback / D8 migration value: broken (finding 1).
- D2 / D5 markerless adopt: broken (finding 2).
- D2 / D4 clone adopting main-weft-or-Bolt: held, including after the prime is re-anchored.
- D3 constructor / D2 fallback for a gone weft: weak (finding 9).
- D3 scan / D5 new-pair Location: held.
- D4 fresh-bind test / D8 Bolt removal: held, since the census requires the prime's weft to carry a marker first.
- D5 run-records drop / D6 move: held.
- D5 origin record / D6 move order: open (finding 7).
- D6 / D7 shared move step: held.
- D6 present-pair refusal / D7 way forward: held.
- D7 refusals / D7 step 1: broken (finding 3).
- D7 re-run / D8 migration commits: broken (finding 5).
- D7 recovery / D8 reconcile: partial (finding 8).
- D7 step 2 / D9 exclude census: held.
- D7 step 4 / D9: held.
- D7 / D10 checkout refusal and its way forward: held.
- D7 / D10 pull on a non-owner machine: broken (finding 6).
- D7 children keep their anchors / the "weft never merges" constraint: held.
- D7 prime / retired-portal assumption: weak (finding 10).
- D8 / D10 promote of an unmigrated task: held while the anchors agree.
- D9 / D10 pull rewire: held.
- Acceptance "interrupted change converges" / D7, D8: overstated (finding 8).
- Acceptance "reconcile writes the right marker" / D8: contradicted by finding 1.
- Acceptance "a child keeps its anchor and lands" / the constraints: held.
- Acceptance on launchers and workspace at `Y` / D5, D6: held if Add uses the new Location after the move.
- Constraint "dobbies end only through `runproc.End`" / D7, D10: D10 does not address dobbies (finding 6).

No further tool calls were made.
