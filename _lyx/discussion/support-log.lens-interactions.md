```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked pairs of decisions in the record that act on the same state or command.

**Findings**

1. MEDIUM — D8 Beam-Up vs D6 adopt vs D1. Beam-Up runs the full set of steps that begin a beam, while adopt explicitly skips part of them.
   - D8 says Beam-Up starts the dobby and orch "through D1's steps", with no restriction. That includes `hubreconcile.Ensure`, the hub-scope origin reconcile, the prime claim and the pausing of owned_elsewhere runs.
   - D6 runs only "D1's steps from the dobby start on", because adopt already reconciled origin.
   - Beam-Up runs inside a live pawl child driver. A hub-scope reconcile there can remove pairs, move branches and owner-pause runs, and the run's own pair is among them, all while that run's driver holds the run lock.
   - Fix: D8 uses D1 from the dobby start on, as D6 does, or the record states why a full reconcile inside a run is safe.

2. MEDIUM — D10 land request vs D11 cancel and restart, and D9 vs D6 ownership round-trip. A land request can outlive the run it was meant for.
   - The request lives in the beam worktree's `.lyx/beam/`. That is dobby's own state directory (`lyxdirs.BeamDirName`), and nothing in the record clears it on cancel or adopt.
   - Path A: `lyx beam land G` is accepted, then `lyx batten cancel G`, then `lyx beam create G`. The new pawl run reaches Beam-Await, finds the old request, its checks pass, and it lands at once with no fresh intent.
   - Path B: the land request is written, then G is adopted elsewhere before Beam-Await consumes it. Beam-Await yields on the owner, and its driver stays dead under the owner pause, so the request stays. G is adopted back later, the owner pause lifts, Beam-Await re-enters and lands on the stale request.
   - Fix: bind the request to the pawl run, for example by run id or start time, or have Beam-Up and Beam-Await clear a request older than the row's entry.

3. MEDIUM — D5 drops the run-held rule and task owners, against runs in flight at the upgrade, with D6 as the only remedy.
   - Today two machines can run tasks under the same root, because ownership is per task: a run-held task carries its own owner, or none.
   - After D5, such a run on a machine that does not own root reads as owned elsewhere. Its next pass is refused and dobby owner-pauses it.
   - The only way forward is `lyx adopt root`, which moves every root run off the other machine as well.
   - The record states no migration, bound or acceptance criterion for this case. It should state the outcome and its way forward, or a one-time migration step.

4. LOW — D5 owner setter vs D10 create's single Bolt write. Create cannot write the recipe through the owner setter as D5 defines it.
   - D5 says the owner setter writes only the root owner and beam owners.
   - D10 records the owner, the beam flag and `recipe: pawl` in one Bolt write. Today's `SetOwners` `ToProject` path clears `recipe` (`turnIntoProject`).
   - D10 also says a beam project with no recipe reads as pawl, so the recipe write is redundant.
   - Fix: either the setter writes the recipe and D5 says so, or create does not write the recipe.

5. LOW — D6 adopt root vs D11, where a pawl run's owner verdict reads the beam's own owner.
   - `lyx adopt root` resumes every run in flight whose parent is the prime, G's pawl run included.
   - When G is owned elsewhere, that first pass is refused as owned elsewhere, and adopt reports it as G's slug error.
   - That error is not a fault, but the record does not say so. Adopt should skip, or report as expected, a run whose entry reads owned elsewhere.

**Pairs checked**
- D1 prime claim / D5 no root owner: held.
- D1 `--all` / D3 per-beam refusals: held.
- D1 owner-pausing of owned_elsewhere runs / D11 pawl owner verdict: held.
- D2 resume of the recorded session / D4 beam stop: held. `orch stop` leaves the recorded session id alone, so a start after a stop resumes it.
- D2 dead-watcher restart / D7 watcher exits on an owner change: held, because the strand is stopped first.
- D3 refusal on a beam owned elsewhere / D7 stale local Bolt: held. D3 reads the Bolt it just pulled.
- D4 `lyx s` / D3 refusal in a task pair: held.
- D4 removal of `orch start` / every way forward renamed: held.
- D5 effective owner / D10 create's two owner refusals: held.
- D5 claim writes status only / PATTERN-owner-machine: held.
- D5 dropping task owners / runs in flight: broken (finding 3).
- D5 owner setter / D10 single Bolt write: broken (finding 4).
- D6 resumes only runs whose parent is adopted / dobby lifting its own owner pause: held, because root's dobby picks up G's pawl run once G reads owned here.
- D6 adopt root / D11 pawl owner verdict: weak (finding 5).
- D6 adopting G / D7 orch on the old machine stopped by its watcher: held.
- D6 adopting G / D11 owner pause through `runproc.End` on the old machine: held.
- D7 / D5 board with no root owner: held.
- D8 Beam-Up / D1 and D6 step sets: broken (finding 1).
- D8 Beam-Down / D4 beam stop / D11 cancel through `runproc`: held. These are two mechanisms, but they act at disjoint times.
- D8 Beam-Up / D3 "no local beam pair": held, because the pair is marked first.
- D9 Beam-Await re-running the land checks / D10 land refusals: held.
- D9 owner yield / D11 owner pause of the batten pass: held.
- D9 snapshot `board-entry.md` / D8 Parent-Align and the perch: held.
- D9 land request / D6 ownership round-trip: broken (finding 2).
- D10 land request / D11 cancel and recreate: broken (finding 2).
- D10 create on a legacy beam / D11 start guard passing a marked pair: held.
- D11 unmarked pair before Beam-Up / dobby starting members: held.
- D12 notice / D7 parent owned elsewhere wording: held.
- D13 promote removal / D10 create's refusal of a task, which today names promote: held, because D13 covers every way forward.
- Acceptance criterion "adopt G does not resume G's pawl run while the parent is elsewhere" / D6: held.
- Constraints PATTERN-run-process-owner and PATTERN-pair-teardown / D8 and D11: held.

**Tool calls**
- grep of `internal/orchcli/stop.go` and `internal/lyxdirs/dirs.go`: my context did not show whether `orch stop` clears the recorded session, or who owns `.lyx/beam/`.
