```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of decisions that act on the same state.

**Pairs checked:**

- D9 accept (stops the dobby) × D9 failed accept ("restarts only when this accept stopped it") across a failed status write and a crash: **broken** (F1).
- D9 accept writes the Parent-Align status, then re-runs land's checks × D10's check "pawl run not running at Beam-Await" × D9's rule that the hold is read from the board producer half: **broken** (F2).
- D9 Finalize re-runs `BeamLandRefusal` × D10 drops the live-orch refusal × D8 runs Beam-Down after Finalize: **broken** (F3).
- D9 window between the dobby stop and the hold write × D1:38 (no `EnsureLive` caller starts a held beam's dobby) × D3 × D4's hand `lyx batten run` restart: **broken** (F4).
- D9 Finalize backstop's per-member ways forward × D9:154–155 hold refusing a member's Worktree-Create: **broken** (F5).
- D9 interrupted-landing re-entry × D3 landing-hold refusal (the entry write lifts the hold before naming `lyx beam start`): held.
- D9 member Worktree-Create post-pair read × D11:181 (beam flag holds a member, and the pawl status at Worktree-Create is no hold): held.
- D9 "no hold through Beam-Await, halts included" × D11 cancel refusal × D6 skip under hold: held.
- D9 slot rule × D10 create's detached first pass (not a dobby flight) × acceptance "max_runs pawl runs at Beam-Await still starts a ready task": held.
- D9 owner yield removes the request × D10 land refuses a beam this machine does not own: held.
- D6 adopt G then root and root then G × D11 (pawl owner verdict reads the beam's owner) × the owner-pause lift: held. Pause records are hub-local (`pauserecord` doc), so the other machine's owner pause does not block the adopter's resume.
- D6 skip beam steps under hold × D7 watcher stopping the old orch × D8 Beam-Down: held.
- D5 Effective (drops the run-held rule) × D9 "in flight at every row" readers × D5 upgrade owner pause naming `lyx adopt <project>` × D6 task refusal: held.
- D5/D10 owner setter writes the beam flag and `recipe: pawl` × D10:172 board accepts pawl on a project only × `project.go` beam-flag refusal: held.
- D8 Beam-Up removes the land request × D9 request bound to the seed id: held.
- D4 `lyx beam stop` (role-scoped end) × D9 driver keeps running × acceptance "driver re-spawned after `lyx beam stop` posts none": held.
- D11 cancel of a memberless beam × D4 role end: held.
- D12 notice × D9 accept stopping the dobby × D8 Beam-Down: held.
- D2 orch resume × D7 watcher stop (the stop keeps the recorded id): held.
- D1 `--all` × D3 per-beam refusals: held.

**Findings:**

F1 — MEDIUM, design. Location: D9 accept and failed-accept bullets (`decision-record.md:141,144,145`).
Summary: "restarts the beam's dobby only when this accept stopped it" leaves the dobby down for good when the stopping and the failing accept are different attempts.
Two cases reach this: a failed status write keeps the request and the next wake retries, and a crash or re-spawn lands between the stop and the status write.
The retrying accept finds the dobby already stopped, so on a failed re-check it writes Beam-Await and does not restart it.
The beam then sits at Beam-Await with no hold, no dobby and no notice, and its members go undriven.
"Who stopped it" must be persisted, or the failure path must restart a dobby the last `lyx beam stop` did not stop.

F2 — MEDIUM, consistency. Location: D9:141 against D10:169 and D9:153.
Summary: the accept writes the run status naming Parent-Align and then "re-runs `lyx beam land`'s checks (D10)".
D10's checks include "a beam whose pawl run is not running at Beam-Await", and D9:153 says every reader judges the row from the board status's producer half.
Read that way, the re-check refuses its own accept every time.
The record must name which checks re-run (members, pairs, owners) or the source the row check reads.

F3 — MEDIUM, consistency. Location: D9:146 against D10:170 and D8 table rows Finalize and Beam-Down (`decision-record.md:131-132`).
Summary: Finalize re-runs `BeamLandRefusal`, which today also refuses while the beam's orch or watcher is live (`internal/battencli/beamland.go:86-92`).
D10 drops "the refusal of a live orch" only under `lyx beam land`'s bullet, and Beam-Down, which stops the orch, runs after Finalize.
Unless `BeamLandRefusal` itself loses its orch blocker, every pawl Finalize halts on the beam's own live orch.

F4 — LOW, design. Location: D9:141 against D1:38, D3:60 and D4:70.
Summary: between the accept's dobby stop and its hold write, no hold stands.
In that window `lyx beam start` passes D3, and a member's hand `lyx batten run` passes `ensureParentDobby`; either can restart the dobby.
That dobby then stays live through Parent-Align up to Beam-Down, against D1:38.
D9's restart bookkeeping does not see it either, and the record states no bound for the window.

F5 — LOW, consistency. Location: D9:146 against D9:154-155.
Summary: the Finalize backstop halts "with the refusal's own ways forward for each member".
For a member with a pair, that way forward is "land it with `lyx batten run <member>`" (`internal/battencli/beamland.go:80`).
Under the standing hold, that member's Worktree-Create is refused, and nothing returns a held pawl run to Beam-Await, so the named way forward is a dead end.
Only the cancel and the move off the project still work.

**Tool calls:**

1. Read `internal/battencli/beamland.go:1-120`: the wording of `BeamLandRefusal`'s ways forward and its orch blocker, which the manifest named but had not shown.
2. Grepped for `OwnerCaller` writers: the scope of the owner pause, needed for the adopt G then root pair.
3. Read `internal/pauserecord/doc.go:1-40`: whether the pause record is shared across machines, which settles that pair as held.
