```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism on one concrete case and checked the end state against the decision's purpose.

**Findings**

1. **MEDIUM, D6:** a new autonomous beam with no members yet drains on its first Beam-Await wake.
   - Case: `lym beam create` with autonomy on; Beam-Up starts the dobby; Beam-Await enters before any member task is filed.
   - At that wake no member is open, no child pair exists and no ready record stands, so D6 posts `[beam drained]` at once.
   - D7 then has the orch write a description and run `lym beam pr open` on an empty beam. A beam with no commits beyond its parent gets no PR, so step 3 refuses or does nothing.
   - The purpose, notifying when the beam is ready to merge, is missed.
   - Fix: also require that the beam holds commits beyond its parent, or that at least one member has landed since the last drain.
2. **MEDIUM, D7 step 1 with D6:** the record leaves undefined how the orch learns that the merge task landed.
   - `lym beam merge-parent` files a released darn task in the beam (`internal/beamcli/mergeparent.go`, "a released darn task in the beam's project"). That is an open member, so D6 re-arms.
   - Once that task lands, D6 posts a second `[beam drained]`.
   - The record never says that the second line is how the orch knows the merge task landed. The role forbids polling, so the wait has no other end.
   - The record also never says the orch resumes D7 at step 2 rather than repeating step 1.
   - Fix: name the second drained line as the wait's end and say where the steps resume.
3. **MEDIUM, D5 class 5:** the "settled twice for the same row and cause" bound needs a counter that outlives a context cycle, and none is named.
   - The orch keeps no state across a cycle except the orch note.
   - Case: the orch settles a Drift-Verify stop once, cycles its context, then gets the same stop again. Its count restarts at zero, so the bound never fires and the orch can retry forever.
   - Fix: record the count per slug, row and cause in the orch note, or keep it in lym.
4. **MEDIUM, D8 with D9:** they read the autonomy value at different times.
   - Case: the orch launches with autonomy off, so its launch line says off and it marks nothing. The operator then sets `autonomy=true` through `lym config beam` without telling the orch.
   - D8's hook reads at call time and starts dropping every unmarked reply. That includes the orch's answers to the operator's Slack mail, which the orch does not mark until its next cycle.
   - The operator's Slack conversation goes silent, against D8's own bound.
   - Fix: have the hook use the value the orch was launched with, or have a change of the key mail the worktree's orch.
5. **MEDIUM, D1 with the auto-mode assumption on beam Finalize halts:** a beam's own halts go to its parent worktree's orch, not the beam's.
   - Case: an autonomous beam inside a non-autonomous prime, and the beam's Finalize halts on a pending GitHub check.
   - The beam's pawl run is a batten child of the prime, so its `[batten notice]` reaches the prime's orch. That orch is under the prime's value, off, and asks the operator.
   - So "the beam's own value governs its own landing steps" does not hold for halt routing. The record states it as an assumption without checking which orch receives the notice.
   - Fix: state which orch receives a beam run's halts and whose autonomy value applies.
6. **LOW, D4 acceptance:** the D4 bullet "parent-satisfied … `Continue` set … ends PR-Review" tests nothing new.
   - That ending path already exists, and the new behaviour, the agent running `pr-continue` on the parent's word, is stencil-only. Only the `Interactive` false bullet pins new code.
   - Fix: drop the bullet or mark it as an existing-behaviour check.

**Per decision: case used, purpose met or not**

- D1: a beam worktree layer of `false` over a default layer of `true`. Both layers apply to the non-hub-wide `beam` module (`LayerFile`), and a run_settings value is refused as Guard. Met.
- D2: a member of an autonomous beam, read through the parent location as `parentPullRequestMode` does; an unreadable stack reads off. Met.
- D3: Drift-Verify under autonomy with `judge_ask_parent: false`. Forcing at read time adds the parent-agreement gate. Met.
- D4: a member PR under autonomy. The session spec is non-interactive, the `[pr-review request]` reaches the parent, the satisfied answer leads to `pr-continue` and Finalize merges. Met, with finding 6.
- D5: a repeating Drift-Verify stop across a context cycle. Not met (finding 3).
- D6: an empty new beam. Not met (finding 1). A beam whose last member lands: met.
- D7: a beam lagging its parent. Steps 1 to 4 only partly work (finding 2).
- D8: an orch that stays autonomous with the presence slack. Met. A mid-session flip: not met (finding 4).
- D9: the launch line at start, resume and reload. Met; its timing gap is finding 4.
- D10: two glossary lines added. Met.

**Tool calls made**

1. Grepped `internal/beamcli/mergeparent.go` for draft and release status, and pawl's Deps for a `Notify` seam: my context lacked whether a merge task starts released, which D7 depends on.
2. Listed the `internal/pawlengine/deps.go` seams and the `pawlwiring.go` re-check wiring: my context lacked which board reads Beam-Await can make for D6.
3. Read `pawlRecheck` and the Beam-Await marker and notice code: my context lacked how Beam-Await already detects remaining work (`battencli.ReadBeamLandBlockers`) and how it posts once.
