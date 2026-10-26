```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: I traced the existing code paths that read or write the state each decision changes, and checked whether each decision's result survives them.

**Findings**

1. BLOCKING — D8/D9/D11, and D1/D3 by inheritance. **A pawl run keeps the beam's board entry run-held for the beam's whole life, and four existing paths treat a run-held beam entry as "being landed".**
   - `battencli/beamfork.go` `beamForkPrecondition` refuses every member's Worktree-Create while `IsRunStatus(beamEntry.Status)`, saying "which its own landing holds".
     So no member could ever fork from a beam at Beam-Await.
   - `battencli/refusal.go` `ResolveBattenPrime` (around lines 85–100, via `beamLanding`) refuses every batten, adopt, cleanup and beam verb run from inside a beam whose entry holds a run status.
     That blocks the beam's own orch, and D10's "`lyx beam land` runs anywhere".
   - `orchcli/refusal.go:38` refuses the orch start in a beam "held by its landing run".
     That logic moves into `lyx beam start` and Beam-Up (D1) and would refuse every beam start, including Beam-Up's own.
   - The record removes only the live-orch land refusal and says nothing about these three.
   - Fix: replace "the entry is run-held" with "the pawl run is past Beam-Await" (or "the land request was accepted") in all three places.

2. MEDIUM — D8/D10, the run lock. **The project entry cannot be edited for the beam's whole life.**
   - `boardengine/runlock.go` `checkRunLocks` refuses any field change but `Status` on a run-held entry.
     Under pawl the project entry is run-held for weeks, so its body, brief and labels are frozen.
   - Before, it was frozen only during the short landing run.
   - Beam-Await's snapshot (D9) assumes the entry may change during the beam's life, so the record contradicts itself unless the lock exempts a pawl run before Beam-Await ends, or the record states and accepts the freeze.

3. MEDIUM — D5, the README layer. **A beam project would move out of the Project layer into Running.**
   - `boardengine/layer.go:35` places any run-held entry in the Running layer.
   - The record is silent on whether that is intended.
     It also touches note `readme-per-project-files`.

4. MEDIUM — D5. **Two run-held readers outside `Owners.Effective` keep the old rule.**
   - `tendercli/deps.go:177` (`entry.RunHeld && !local[slug]`) and `adoptcli/adopt.go:254`/`370` (`entry.Owner != "" || entry.RunHeld` as "started") are not covered by D5's change to `Effective`.
   - Once a task's `owner_machine` is dropped on load, adopt's "started task" test and its pair-add rest on `RunHeld` alone.
   - Tender's owner-pause selection keeps its own run-held exemption, which D5 says goes.
   - D5 should state that every reader of the run-held owner rule follows it, not only `Effective`.

5. LOW — D5, the drop on load. **Owners can come back from the other machine.**
   - The board replay merges `owner_machine` origin-wins (`replay.go`).
   - A machine still on the older binary writes task owners again, and the new binary drops them again on its next load.
   - This is harmless only if no path reads them in between; the record says "never read", which holds only for the new binary.
     The mixed-version window is unstated.

6. LOW — D7. **The watcher change waits for a watcher restart.**
   - The orch watcher outlives a deploy (orchcli doc: it runs `help index` from the recorded `lyx` path).
   - So D7's owner check reaches a running watcher only after it restarts.
     The bound should say so.

7. LOW — D8/D9. **The pawl run's records are now present while members fork, and members carry them.**
   - The pawl child's tracked `_lyx/shed/<beam>/` (and later `_lyx/discussion/board-entry.md`) sits on the beam's records branch while members fork from it.
   - Today's landing run starts only once no members are left, so no member ever carried a parent's run records.
   - I could not confirm whether a member's records fork filters `_lyx/shed`, so this is unverified.

**Paths traced**

- D5 `Effective`: pairteardown `originseams`, loomcli `ownercheck`, landingdeps' parent-owner read, fabriccli `anchordeps` and dobby owned all survive, since they call `Effective`.
  Tender `deps` (`RunHeld`) and adopt's started test break, as in finding 4.
- D5 drop on load: `checkOwnerPayload` survives (an effective-owner payload is a no-op); the replay survives only with every machine on the new binary (finding 5).
- D8/D9 run-held beam entry: `beamForkPrecondition`, `ResolveBattenPrime`/`beamLanding` and the orch start refusal break (finding 1); the run lock breaks (finding 2); layer placement changes (finding 3); `cancel.go`'s "other left" read survives.
- D1/D3 orch start moved: the landing-held refusal breaks (finding 1); the prime claim, owner pause and reconcile survive as moved.
- D7 watcher: survives only after a watcher restart (finding 6).
- D10 `lyx beam land` anywhere: breaks inside the beam through `ResolveBattenPrime` (finding 1).
- D11 start guard: survives once the land refusal is removed; Worktree-Create survives for the member fork only after finding 1 is fixed.

**Tool calls**

- Read `boardengine/runlock.go` lines 1–40 and grepped `orchcli/refusal.go`: the run lock rule and the orch landing refusal were not in context.
- Read `boardengine/runlock.go` lines 40–80 and grepped run-held readers: the lock's scope and the other `IsRunStatus` readers were not in context.
- Read `battencli/beamfork.go` and `battencli/refusal.go`: the member-fork and verb refusals on a run-held beam were not in context.
- Grepped `Effective(` and `RunHeld` across `internal/`: the owner verdict readers outside `boardengine` were not in context.
- Grepped `fabricengine/doc.go` for the records fork: what a member's records fork carries was not in context; it was inconclusive.
