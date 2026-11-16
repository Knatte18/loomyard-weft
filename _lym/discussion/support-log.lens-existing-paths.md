```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: I traced the code paths that already write or read what each decision changes.

**Findings**

1. **MEDIUM, D9 step 6.** Lifting the stop's pause starts new processes after the reap.
   - Evidence: `tenderengine.Lift` calls `liftEntries` (`internal/tenderengine/resume.go`), which runs `spawnOwed`. That starts detached `pair:<slug>` pushes, a `prime-records` push and a `board-sync`, and sends resume lines to the sessions the entry held.
   - Step 6 runs after step 5's reap, so lym processes outlive the stop. That breaks D9's criterion "leaves no … run process". It also pushes pairs whose sessions were just taken down.
   - The record says nothing about this. Either the stop lifts without the owed spawns, or the record says those spawns run and are allowed.
2. **LOW, D9 step 3.** `orchcli.BeamDown` has callers besides `lym beam stop`.
   - Evidence: `internal/battencli/cancelcmd.go` calls it when a beam is cancelled, and `internal/loomcli/pawlwiring.go` calls it for pawl's BeamDown row when a beam lands.
   - With the change, both callers now end the beam orch's registered Claude process. That is probably intended, but the record names only `lym beam stop`.
3. **LOW, D9 step 1.** A hub pause that already stands could be lifted by the stop.
   - Evidence: `tenderengine.Pause` over a live hub entry sets `AlreadyPaused` and hands back the existing entry (`internal/tenderengine/pause.go`).
   - If the stop takes that entry's identity as its own, step 6 lifts the operator's or hibernate's hub pause. The shared pause-and-confirm function must own only an entry it added, as hibernate's `req.Added` does. The record should say so.
4. **LOW, D1.** Retiring the respawn removes a safety net that still has users.
   - Evidence: `respawnPairPush` in `internal/fabricengine/teardownlock.go` respawns any detached pair push that exited on a held teardown lock, not only a transition's.
   - The anchor-change push and resume's pair push stay detached, at least until D7 lands, and lose that respawn. The next synchronous pair push carries the branch, so it heals itself, but the record doesn't say so.
5. **LOW, D10.** Launchers for a hub that disappears are never found.
   - Evidence: `hubregistry.Live` drops an entry whose path or Bolt shortname is gone, so a verb that only iterates `Live` never sees that hub.
   - Removing such a hub's launcher needs the launchers on disk compared against `Live`. The record says "finds the hub … gone from `hubregistry.Live`" without that mechanism.

**Paths traced**

- **D1**
  - Detached pushes the transition push replaces: survives.
  - The teardown-lock respawn for the pushes that stay detached: breaks quietly, finding 4.
  - Tender resume's `spawnOwed`: survives; it is detached and carries no transition.
- **D4.** The origin-owed marker's writer `markOriginOwed`, its readers in `battencli/arm.go` and `dobbyengine/passes.go`, and `clearOriginOwed`: all retire with it, and the record names the marker. Survives.
- **D5**
  - `lym reed down` under a `hold` slug entry: accepts, since a pair entry covers the pair. Survives, so the stop works under a hold.
  - The hibernate wake's handling of a hub entry: survives.
- **D8.** Go readers use `boardengine.ParentBeam` and `hubgeom.ParentOf` (beamcli, battencli startguard and beamfork, loomcli projectrecord), which stay unmapped. Survives.
  - The printed field is read only by `boardcli` `outputGetTask`, the `BriefTask` JSON tag, `contracts/orchrole/orch-role.md` and `plugins/ly/skills/board/SKILL.md`, and D8 names all of them. Survives.
- **D9**
  - `Lift` side effects: breaks, finding 1.
  - `BeamDown` callers: survive, finding 2.
  - `Pause` over an existing hub entry: risk, finding 3.
- **D10**
  - `beam start --all` claiming an unowned prime through `OwnedWorktrees` and `EnsureLive`: survives, because the verb skips such a hub first.
  - `Live` dropping a hub that is gone: finding 5.

**Tool calls**

1. Grepped the `BeamDown` callers, the `parent_beam` readers and `Lift`. My context had only `BeamDown`'s existence, not its callers or `Lift`'s body.
2. Read `liftEntries` and `spawnOwed` in `tenderengine/resume.go`, to see what a lift starts.
3. Grepped every reader of the printed `parent_beam`, Go, contracts and skills, to check D8's reader list.
4. Grepped the Go readers of the `.ParentBeam` field, to check whether the printing-boundary mapping leaks into code.
5. Read `tenderengine.Pause` over an existing hub entry, to check D9's "owns by identity" against an operator's pause that already stands.
