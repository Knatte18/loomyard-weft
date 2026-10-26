# Card 24 — adopt-beam-steps

**Edit:**
- `internal/adoptcli#adoptDeps`
- `internal/adoptcli#adopt`
- `internal/adoptcli#slugOutcome`
- `internal/adoptcli#outcome`
- `internal/adoptcli#hubDepsAs`
- `internal/adoptcli#Command`
- `internal/adoptcli#world.deps`
- `internal/adoptcli#TestAdopt`
- `internal/adoptcli/doc.go`

**Intent:**
D6: after the owner write and the resumes, adopt runs each adopted project's beam steps (D1), since its own step 1 already reconciled origin: the dobby, the beam window and the orch, so the pair, dobby and orch move with the project.

- `adoptDeps.StartDobbies` becomes `BeamUp func(worktree string) (orchcli.BeamUpReport, error)`, which `hubDepsAs` fills with `orchcli.BeamUp` over the worktree with `Prepare` false; `adopt` calls it for each adopted beam whose owner write stood, and for the prime when root was adopted.
  A beam whose pawl run holds the landing hold gets no beam steps; the hold comes through a new `adoptDeps.LandingHeld func(slug string) (bool, error)` seam, which `hubDepsAs` fills by reading the entry through `boardengine.ReadTasks` and judging it with `boardengine.RunHolds`, since `ReadOwners` carries no status producer, and which `world.deps` scripts per slug; a nil seam reads no hold, and a failed read is the slug's warning naming `lyx beam start <slug>`. Such a beam: its owner is still written, and its outcome names `lyx batten status <slug>` and, when this machine does not also own the beam's parent, `lyx adopt <parent subject>` (`lyx adopt root`), which resumes the landing here by card 4's rule.
  A failed `BeamUp` is the slug's own warning, naming `lyx beam start <slug>`, and the other slugs go on; `outcome.DobbyWarning` becomes the per-slug `beam_steps` outcome, and `Command`'s envelope and help follow.
- The origin reconcile, `--keep-local` and `--drop-local`, the anchor-move refusal, the owner pause lift and the machine check (step 6) are unchanged.

`TestAdopt` gains rows: adopting a beam runs its beam steps after the resumes and reports the outcome; a held beam has its owner written, no beam steps, and names `lyx batten status` and, with its parent owned elsewhere, `lyx adopt <parent>`; a failing beam step warns naming `lyx beam start` while the other slug's steps run; adopting root runs the prime's beam steps.
`adoptcli/doc.go` describes the steps.

**ImpactSummary:** adopt's dobby start becomes the full beam steps through orchcli's export; the seam has one fake in `world.deps`, and no other package calls adopt.

**Commit:** 24: `lyx adopt` brings up each adopted beam's dobby and orch
