```yaml
lens: board-scope
outcome: reported
```

Lens: board-scope. I checked the record's fit with the board: the entry's goal points, the touching entries and the peers.

**Findings**

1. **MEDIUM, class `decision` (D9).** The record drops a basis the operator dated for the stop button. A question for the run's parent: does the operator accept that the stop uses no per-run pause or cancel intents?
   - Evidence: the entry's hub-stop-button section says "Built on the run FSM and its pause and cancel intents (operator, 2026-11-15), so a stop leaves every run in a pushed state".
   - D9 instead writes no pause intent and no tracked record, and rests on D1's push-before-act.
   - The departure appears only in the support log's rejected alternatives. Neither `## Open risks` nor D9's rationale names it as a departure from a dated operator call.
   - Suggested fix: add an Open risks line asking the parent, or state the departure in D9's rationale.

2. **LOW, class `consistency` (D8).** D8 departs from the entry's text and does not say so.
   - Evidence: the entry says "Every reader of an empty `parent_beam` (orch role, dobby, board verbs) moves to the named root in the same change."
   - D8 maps only at the printing boundary. `hubgeom.ParentOf` and `boardengine.ParentBeam` keep the empty slug, so the dobby and the board verbs' code-level reads do not move.
   - The rationale explains why the code's slug stays empty. It does not say that the entry's dobby and board-verb readers are left out on purpose.
   - Suggested fix: one clause naming the departure.

3. **LOW, class `consistency` (D10).** D10 changes what the entry's terminal runs, and does not name that as a departure from the entry.
   - Evidence: the entry says the terminals run "`lym s`, which starts its dobby and orch and attaches to its reed session".
   - D10 runs `lym beam start --all` once and then `lym reed attach` in each terminal.
   - D10's bullet gives the reason (`lym s`'s adopt-here takeover). This holds with the entry's "starts only what this machine owns", so it is not a conflict, only an unmarked change.

**Goal points of the entry**

- Merge every remaining member into one design: met.
  - push-before-act-takeover becomes D1 and D2.
  - bolt-holds-catch-up becomes D3 to D5.
  - adopt-foreground-leftovers becomes D6 and D7.
  - prime-is-root-beam becomes D8.
  - hub-stop-button becomes D9.
  - machine-restart-shortcut becomes D10.
- The orch resumes its own session on the same machine only, with no mirror: met. Scope puts these out as landed or ruled out.
- The orch stays up during an adopt hold: met, in D5.
- The stop, the restart and the root-beam fix joined the beam: met. All three are in, and the support log rejects moving them out.
- Split again from scratch under raddle's rules: met. Scope leaves the split to raddle, the Notes give only code dependencies, and the study's shape is not taken.
- Built on what landed: met. Scope names `internal/runfsm`, `Judge` and `Aside`, the intents, `RegisterOrch` and `EndSubjectProcesses` as given.
- The stop's open points (pause first, confirm text): met, in D9.
- The restart's points: met, in D10. Owner skip, one launcher per hub plus one for all, `.desktop` and `.cmd`, beams included and no run resumed are all there. The launcher's removal is bounded to this machine.
- The root-beam fix: partly met (finding 2).

**Entries, peers and parent commits checked**

- `adopt-build`, the beam: none. Its project record holds only the goal, and the record keeps it.
- `adopt-revision`, the grandparent project: none. Its goal of nothing happening until adopt is done fits D5 and D6.
- `orch-auto-remote-control`: none. It is now running Darn-Write and touches orch launch and resume and reed's buttons. D9 touches `orchcli.BeamDown` and the `prime` button group in the same templates. That is a text-only overlap with no design conflict.
- `claude-account-pick-simple`: none. It depends on adopt-build and is out of scope.
- `move-task-by-handback` (note): none. The re-parent verb is out of scope.
- `codex-same-as-claude` (peer): none. Its shell-guard work and the `--take-hold` row are a text-only overlap.
- `weaver` (peer): none.
- Parent commits: `lym loom peers` lists none.

**Decision notes**

The board holds no `Decision for adopt-build-resplit:` note, and every note was reachable.

**Tool calls**

1. `lym board get` for `orch-auto-remote-control` and `adopt-revision`. My context had only their list rows: one is now a running task touching orch and reed, and the other is the grandparent project whose goal binds.
