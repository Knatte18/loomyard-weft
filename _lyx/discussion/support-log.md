# Support log — beam-start-and-ownership

## Interview

Autonomous run: no operator answered; each turn is the writer's own exploration and pick.

### Turn 1 — the board

Read the entry and the board list.
No `Decision for beam-start-and-ownership:` note exists.
Touching notes read: `beam-promote-keeps-discussion` (premise is promote, which this task removes), `loom-finalize-merges-itself` (loom and darn Finalize flags, leaves pawl to this task), `adopt-check-visible` (adopt's step 6, untouched here), `loom-beam-darns` (a task worktree turned into a beam), `darn-drift-await-row` (no overlap), project `workspaces`.

### Turn 2 — what is already landed

`afd030a4` already names the window `beam: <slug>` / `beam: prime`, so the brief's window point is landed; `lyx beam start` only ensures the window.
`9c3085c4` gives each beam its own anchor and `lyx beam create --anchor`; the anchor must survive create no longer making the pair (D10).

### Turn 3 — the orch verbs

`orchcli`'s doc: `start` reconciles config, reconciles origin, claims the prime, owner-pauses, starts every owned dobby, then the strand and watcher; `--continue` resumes the recorded session and refuses without one.
`lyx prime` is a shortcut `[[orch, start], [reed, attach]]`, so `lyx s` is the same shape.
`lyx cleanup` already runs `orchengine.CleanupHandoffs` over every orch, so `lyx orch cleanup` is redundant.

### Turn 4 — ownership today

`boardengine.Owners.Effective`: recorded owner, else none for a run-held entry, else the `hubgeom.ParentOf` parent's owner, root's at the top.
`ParentOf` already walks to the nearest beam, skipping grouping projects, so making tasks ownerless reuses it unchanged.
`adoptcli` refuses a child whose parent is owned elsewhere and a beam whose members are; both rules lose their object once tasks have no owner, and the operator's example (adopt G alone) contradicts the first.

### Turn 5 — the beam recipe and how a child waits

Today's `beam` recipe is Beam-Describe then Finalize, with `LargeMergeLands` and no reroute.
A shed `Yield` ends the Go driver's pass; only batten's own Run-Shed lives on yields, re-run by dobby.
Drift-Await waits in-process; so Beam-Await waits in its driver process (D9).
InnerRun's quiet notice would fire on an agentless child, so D9 suppresses it at Beam-Await.

### Turn 6 — the cross-machine walk

Walked the operator's G/H example through batten records: G's pawl batten run belongs to root's prime, its child to G's worktree.
Adopting G alone must not resume G's pawl run (its driver is root's dobby elsewhere), so adopt resumes only runs whose parent is an adopted worktree (D6), and the pawl run follows `lyx adopt root`.
The example, moved here from D6: with G and H beams under root and all on Neve, OSL-1046 runs `lyx adopt G` and works G's members there; Neve's pass of G's pawl run reads G owned elsewhere and owner-pauses it; `lyx beam land G` on OSL-1046 refuses naming `lyx adopt root`; `lyx adopt root` then resumes G's pawl run on OSL-1046; H stays on Neve.

### Turn 7 — lens review

Seven forks returned; reports in `support-log.lens-<lens>.md`.

- existing-paths 1 (BLOCKING, run-held beam entry read as a landing hold by `beamForkPrecondition`, `ResolveBattenPrime`, orch start): fixed, D9's last bullet rebases the hold on "past Beam-Await", D3 gains the refusal.
- existing-paths 2 (run lock freezes the entry for the beam's life): fixed, D9 leaves the fields editable until Beam-Await ends.
- existing-paths 3 (beam moves to the Running layer): fixed, D9 keeps it in the Project layer at Beam-Await.
- existing-paths 4, interactions 3-adjacent (other run-held readers): fixed, D5 names tender's and adopt's readers.
- existing-paths 5, bound-breaker 7 (mixed-version window): fixed, D5's upgrade bound.
- existing-paths 6 (watcher from an older build): fixed, D7's bound.
- existing-paths 7 (members carry the pawl run's records): rejected, run directories are per slug and a member reads only its own.
- interactions 1, bound-breaker 3 (Beam-Up running the reconcile): fixed, D1 defines "the beam steps", Beam-Up and adopt run those alone.
- interactions 2 (stale land request): fixed, D9 binds the request to the run's seed id, Beam-Up and the owner yield remove it.
- interactions 3, walkthrough 4 (runs in flight at the upgrade): fixed, D5's upgrade bullet and an acceptance criterion.
- interactions 4 (owner setter vs recipe write): fixed, D5 and D10 say the setter's beam write sets `recipe: pawl`.
- interactions 5 (adopt root resuming a nested pawl run owned elsewhere): fixed, D6 reports it as owned elsewhere.
- walkthrough 1 (BLOCKING, batten records on the prime's records branch): rejected, run directories are per slug and the records push replays a rejected push (`fabricengine` doc), as two machines owning different tasks already do; D11 states it.
- walkthrough 2 (BLOCKING, `runproc.End` of the slug kills the pawl driver): fixed, D4 ends the dobby by its role alone.
- walkthrough 3 (late member after Beam-Await): fixed, Beam-Await stops the dobby and the rebased hold refuses the member's Worktree-Create.
- walkthrough 5 (re-spawn churn while owned elsewhere): fixed, D9's bound.
- walkthrough 6, goal-scope F3 (fresh orch after a stop): fixed, D2 states it.
- bound-breaker 1 (BLOCKING, cancel with members): fixed, D11 refuses the cancel while `BeamLandRefusal` refuses.
- bound-breaker 2 (can-land trigger at Land Done): fixed, D12 checks at a run's done after teardown or a removing cancel.
- bound-breaker 4 (create under a pause): fixed, D10's re-run and bound.
- bound-breaker 5 (beam owner blanked with no root owner): fixed, D5 keeps it and D10 claims root.
- bound-breaker 6 (marked task pair after promote's removal): fixed, D11 keeps the refusal.
- goal-scope F1 (grouping projects narrow the entry): fixed, D5 says so and why.
- goal-scope F2 (D7 unlicensed): fixed, D7's rationale ties it to the entry's sentence.
- goal-scope F4, board-fit 1 (promote open risk already settled): fixed, risk dropped; the notes' premise is a board matter, no parent question.
- board-fit 2 (README note): fixed, Out names `readme-per-project-files`.
- leanness (file pointers in Notes, D6 example, auto-mode restatements, two PATTERN-only constraints, second open risk, Goal repeating Scope): fixed, each cut or moved here; D8's rationale cut.

## Rejected alternatives

- Beam-Await as an `Awaiting` halt released by `lyx beam land`: an awaiting stop enters the stop chain (rule table, tackler, arbiter, parent) and notices, which a planned weeks-long wait should not.
- Beam-Await yielding and InnerRun re-spawning the driver per pass: spawns a driver every probe and couples batten to one recipe's row.
- Renaming `prime_owner` to `root_owner` in `board.json`: churn in the replay and on a machine still on an older binary, for a name only the views show.
- Keeping a per-task owner as an override: the entry says a task has no owner of its own.
- Owners on grouping projects: a grouping project has no worktree and no dobby to drive it.
- `lyx s <slug>`: the shortcut routing sends extra words to the last argv, `reed attach`, so a slug would miss `beam start`.
- Dobby, not the watcher, stopping the orch on an owner change: a `runproc.End` from the owner pause can kill the dobby first, so the orch would survive by race.
- Keeping `lyx beam land` refusing from inside the beam: land now only writes a request, so the beam's own orch may issue it.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Do grouping projects carry an owner? No; they follow their nearest beam or root. Why: no worktree, no dobby.
- Q2 (auto-pick): How is root named? `root` in every verb taking a beam slug; `board.json` keeps `prime_owner`. Why: no replay churn.
- Q3 (auto-pick): Does `lyx s` take a slug? No. Why: shortcut word routing.
- Q4 (auto-pick): `lyx beam stop --all`? Not built. Why: not asked; `lyx beam stop` per beam suffices.
- Q5 (auto-pick): Where does `lyx beam create` run? The parent worktree only, refusing elsewhere naming it. Why: the entry says so, and the parent's dobby drives the run.
- Q6 (auto-pick): Who stops an orch on the old machine? Its watcher. Why: deterministic, independent of dobby's exit order.
- Q7 (auto-pick): How does Beam-Await wait? In-process, wakeups bounded. Why: see Rejected alternatives.
- Q8 (auto-pick): How does a legacy beam (made by today's create) get a pawl run? Re-run `lyx beam create <slug>`. Why: create's steps are idempotent.
- Q9 (auto-pick): A seed recording `beam`? Read as `pawl`. Why: `PATTERN-tolerant-records`, and a landing in flight keeps its rows.
- Q10 (auto-pick): Pawl perch answer key? A project entry snapshot written when Beam-Await ends. Why: darn's perch already reads such a snapshot, and the entry may change during the beam's life.
- Q11 (auto-pick): Do notes `beam-promote-keeps-discussion` and `loom-beam-darns` need a parent question? No; the entry already removes promote, and their premise is a board matter for whoever next takes them up.
- Q12 (auto-pick): Where do a beam's members' batten records live? In the prime's records, unchanged. Why: per-slug paths, and the records push already replays.
- Q13 (auto-pick): Does cancel of a pawl run with members proceed? No, it refuses. Why: it would orphan the members.
- Q14 (note, perch cost): a pawl perch reviews a whole project's diff, whose cost is unmeasured.
