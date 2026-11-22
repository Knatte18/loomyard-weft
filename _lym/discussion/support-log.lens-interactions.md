```yaml
lens: interactions
outcome: reported
```

Task: I reviewed the decision record's decision pairs for interactions (interactions lens).

**Findings**

1. **MEDIUM** (D1 with D9, and D5 with Auto-mode assumptions): when the beam's halts reach a different orch, the two worktrees' values can disagree about who settles them.
   - D1 and the assumption "A beam's own Finalize halts follow D5 under the beam's own value" say the beam's value governs its landing.
   - But the beam's pawl run is a child of the parent worktree, so its batten notices go to the nearest live orch of that parent (`orchcli.ParentStrand`). D9 says that orch follows the autonomy value of its own worktree.
   - Example: the prime is autonomous and the beam is not. A beam Finalize halt reaches the prime orch, which runs under autonomy and settles a halt the beam's operator expected to see. The reverse case also breaks.
   - Fix: name which orch receives the beam's own landing halts and whose value it applies.

2. **MEDIUM** (D9 with D2, D3, D4 and D8): the orch reads its mode once at launch, but everything else reads it on every call, so the two can disagree after a change.
   - D2, D3, D4 and D8 read autonomy fresh each time. D9 has the orch learn it only at launch.
   - After `autonomy` is turned off mid-session, runs go back to an interactive PR-Review waiting for the operator, and the Slack hook forwards every reply. The orch still follows `role-autonomy.md`: it settles design choices itself and never relays them.
   - After `autonomy` is turned on, the orch still follows "the operator ends the review", while the PR-Review session ends on the orch's satisfied answer.
   - The Auto-mode assumption names this window, but no decision bounds it. Fix: have the orch read the value when it handles a notice, or have the config write mail the orch.

3. **LOW** (D6 restart path with D6's AC): the "once per drain" mark may not survive a restart.
   - Beam-Await is a reinvoke row, so a driver restart or interrupt calls it again.
   - D6 does not say whether the "posted for this drain" mark persists. If it lives only in memory, a restart posts a second `[beam drained]` line.
   - Fix: state that the mark persists in the beam's run records, or that a repeated line is harmless because D7's steps can be run again (pr open finds the existing pull request, describe rewrites).

4. **LOW** (D6 with D7 step 1): the record does not say what cues the orch to continue after `merge-parent`.
   - `merge-parent` files a merge task, which is a beam member, so D6 sees the beam reopen and later posts a second drained line.
   - The record says "wait for that merge task to land" but not that this second line is the cue to go on with step 2. Spell it out.

5. **LOW** (D2 with D6): a load failure stalls an autonomous beam with no word to the operator.
   - If the beam's `beam.yaml` stack fails to load, D2 reads it as off, so Beam-Await posts nothing.
   - The operator, trusting autonomy, is told nothing, and the beam's landing is never started. The only trace is a logged warning.
   - D2's "fall back to today's behaviour" assumes an operator is watching, which autonomy removes.

6. **LOW** (D4's second AC): that criterion pins behaviour the merge base already has.
   - "A parent-satisfied pull request whose record has `Continue` set … ends PR-Review, as an operator-ended one does today" describes existing behaviour.
   - The new behaviour that needs pinning is that the autonomous session spec is spawned and the ask fires. The first D4 criterion already covers that.

**Pairs checked**
- D1×D2 (parent-worktree read): held.
- D1×D6 and D1×D7 (beam's own value): held.
- D1×D9 together with the assumption on beam Finalize halts: broken (finding 1).
- D1 local-layer refusal × D1's `run_settings` AC: held (beam is a registered module and Guard is refused).
- D1 default/worktree override × D3: held.
- D2×D3 (forced keys revert on off): held.
- D2×D4 (ask and session spec read at different calls): held, harmless.
- D2×D6 (load failure): weak (finding 5).
- D2×D8 (failure forwards everything): held, safe direction.
- D3×D4 (pr_review_ask_parent forced together with Interactive false): held.
- D3 judge ask × the judge timeout × D5 drift settlement: held.
- D3 `discussion_interactive` × D1 scope: held.
- D4×D5 class 4 (GitHub approval becomes a severe halt): held.
- D4×D7 (member pull requests kept for the description): held.
- D4×D8 (the orch's satisfied answer is a creel post, not filtered): held.
- D5×D8 (severe relay marker is forwarded): held.
- D7×D8 (merge-ready marker is forwarded): held.
- D5 class 5 bound × the D3 asks: held.
- D6×D7 (merge-parent reopens the beam): held, cue unstated (finding 4).
- D6 × restart: weak (finding 3).
- D6 × `lym beam ready` (ready record stops the line): held.
- D6 drafts × D7: held, per the assumption.
- D9 × call-time reads: broken (finding 2).
- Constraints PATTERN-landing and PATTERN-github-auth × D4 and D7: held.
- Constraint on shellguard × D7 merge-parent: held.

**Tool calls**
- Grep of `ParentStrand` in `internal/orchcli/notify.go`, to see which orch gets a beam pawl run's notices. My context lacked the notice routing. The result showed the nearest-live-parent resolver but not whether a beam's own run resolves to the beam's orch or the prime's.
