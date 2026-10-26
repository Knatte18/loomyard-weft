# Round 2 fixer-report — discussion

Every finding was fixed in `_lyx/discussion/decision-record.md`; `support-log.md` is unchanged.

## Changes

- F1 (BLOCKING), fixed as proposed: D10 replaces "The refusal of a live orch goes" with `BeamLandRefusal` losing its orch and watcher blocker for every caller (`lyx beam land`, Beam-Await's re-check, Finalize, cancel, the can-land check). Premise checked at `internal/battencli/beamland.go:86-92`.
- F2 (MEDIUM), fixed as proposed: D9's accept writes a dobby-stop marker under `.lyx/beam/` before stopping the dobby; the failure path restarts the dobby whenever the marker stands and then removes it; the success path removes the request, then the marker. The interrupted-landing re-entry also removes a marker left standing, which closes the crash window between the two removals. New acceptance criterion for a re-entered accept whose re-check fails.
- F3 (MEDIUM), fixed as proposed: the accept now writes Parent-Align first, then the marker and the dobby stop, then the re-check. It states that a member a still-live dobby starts between those steps is refused by its Worktree-Create's post-pair read.
- F4 (MEDIUM), fixed as proposed: the re-check lists `BeamLandRefusal`'s members and member pairs, the beam's owner and its parent's owner, and says the run-at-Beam-Await check is not re-run.
- F5 (MEDIUM), fixed as proposed (first option): D9's InnerRun bullet keeps quiet-notice suppression and sends the driver-dead notice at Beam-Await once `respawnCap` is reached (premise checked at `innerrun.go:186,445`, `notice.go:384`). New acceptance criterion.
- F6 (MEDIUM), fixed as proposed: D9's hold bullet now says a beam project entry with a pawl run in flight refuses at every row a change of `project`, `kind`, `recipe` and `beam`, and its removal and merge, naming `lyx batten cancel <slug>`. Title, brief, body, order and labels stay editable. New acceptance criterion for a refused re-parent. Premise checked at `internal/boardengine/runlock.go:68-125`.
- F7 (MEDIUM), fixed as proposed: D5's upgrade bound states that an older build refuses a beam project carrying `recipe: pawl` (`checkProjectFields` through `normalizeTask` on upsert and replay). The `recipe: pawl` write is kept.
- F8 (LOW), fixed as proposed: D6 adds the bound for the window when the beam and its parent are owned apart, along with its two adopt ways forward.
- F9 (LOW), fixed as proposed: D11's cancel runs the hub-scope origin reconcile first and judges `BeamLandRefusal` over the pulled Bolt. When origin is unreachable it refuses with the reconcile's way forward.
- F10 (LOW), fixed as proposed: D10's re-run bound says that after a first pass that died, the same `--anchor` is repeated.
- F11 (LOW), fixed differently. The proposed text offered every member both cancel and the `project` move. The self-check showed that the run lock refuses the `project` move for a run-held member (`runlock.go` `scopeChanged` compares `Project`). It also showed that cancel must run in the parent worktree, since `ResolveBattenPrime` refuses inside the held beam. The Finalize bullet now words the halt's own ways forward per blocker:
  - a member with a pair or a run in flight, and a forked pair: `lyx batten cancel <slug> --remove`, run in the beam's parent worktree, or on the machine holding the pair;
  - a member with no run status: the `project` move.

  Both are followed by `lyx loom resume` in the beam worktree, and neither is ever the land-it wording. The matching acceptance criterion was updated to match.
- F12 (LOW), fixed as proposed: D1's `EnsureLive` caller list adds Beam-Up and Beam-Await's failed accept.
- F13 (LOW), fixed as proposed: the criterion now reads "a Beam-Await re-entered with its status naming Beam-Await and no request standing posts none".

## Self-check

- fix-claims, MEDIUM: D5's bound overstated that an older build's writes to the whole board fail. Fixed as proposed: the bound now says only that entry's changes and writes fail.
- fix-consistency, LOW: the `lyx beam create` criterion said "without … driver-dead notices", which contradicted F5. Fixed as proposed: it is now scoped to "below `respawnCap`".
- fix-consistency, LOW: the Beam-Await editing criterion said "the project entry can be edited", which contradicted F6. Fixed as proposed: it now names the editable fields.
- fix-consistency LOW, fix-proposal MEDIUM and fix-way-forward MEDIUM, merged: these concern the Finalize halt's ways forward.
  - The source of the halt's wording was ambiguous.
  - The `project` move is refused by the run lock for a run-held member.
  - The halt named no place to run the cancel.

  Fixed, with the proposal's per-member split as the base. The halt now words its own ways forward per blocker, with cancel for a pair or a run in flight and the `project` move only for a member with no run status. It names where to run the cancel. D10's land refusal now reads "naming each with `BeamLandRefusal`'s ways forward" instead of "as today". The criterion was updated.
- fix-proposal, LOW: the driver-dead notice was claimed to name "the beam worktree" (the code says "task worktree"), and the cap-tripping criterion might be unreachable. Fixed with my own change:
  - The notice is now said to use its any-row wording.
  - The InnerRun bullet states that a status write on re-entry resets the count, so the cap trips only for a driver that dies before that write.
  - The criterion was reworded to that case.

  The proposal's alternative was to state that re-entry writes or does not write the status file. I did not take it, because that depends on shed internals the record does not own.

All four forks returned a report.

## Disputed

None.

## Deferred

None.
