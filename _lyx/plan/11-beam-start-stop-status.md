# Card 11 — beam-start-stop-status

**Create:**
- `plan:internal/beamcli#beamTarget` -> `type beamTarget struct`
- `plan:internal/beamcli#resolveBeamTarget` -> `func resolveBeamTarget(location *lyxcwd.Location, slug string) (beamTarget, error)`
- `plan:internal/beamcli#startBeams` -> `func startBeams(targets []beamTarget, up func(target beamTarget, prepare bool) (orchcli.BeamUpReport, error)) (startResult, error)`
- `plan:internal/beamcli#stopCmd` -> `func stopCmd() *cobra.Command`
- `plan:internal/beamcli#orchView` -> `type orchView struct`
- `plan:internal/beamcli#readOrchView` -> `func readOrchView(prime *lyxcwd.Location, worktree string) (orchView, error)`
- `plan:internal/beamcli#TestStartBeams` -> `func TestStartBeams(t *testing.T)`
- `plan:internal/beamcli_test#TestStart_RefusesBeforeTheReconcile` -> `func TestStart_RefusesBeforeTheReconcile(t *testing.T)`

**Edit:**
- `internal/beamcli#startCmd`
- `internal/beamcli#startResult`
- `internal/beamcli#startEntry`
- `internal/beamcli#statusEntry`
- `internal/beamcli#dobbyStatus`
- `internal/beamcli#statusCmd`
- `internal/beamcli#Command`
- `internal/beamcli_test#TestStatus_PrintsThePrimesDobbyAndEachLocalBeams`
- `internal/beamcli/doc.go`
- `docs/overview.md`

**Intent:**
D1, D3 and D4: `lyx beam start [slug] [--all]`, `lyx beam stop [slug]` and `lyx beam status` showing dobby and orch.

- `resolveBeamTarget(location, slug)` resolves the beam a verb acts on, before any reconcile:
  no slug names the acting worktree's beam, the prime or a marked beam pair; the slug `root` names the prime; any other slug names that beam from anywhere in the hub.
  It refuses outside a hub with the location's own error and the hub to run in; a task pair, a worktree whose origin record carries no beam mark, naming the beam it belongs to (`hubgeom.ParentOf`, `root` for the prime), that beam's worktree path and `lyx beam start` there; a slug that names no local beam pair, naming `lyx beam create <slug>` from its parent worktree, or `lyx adopt <slug>` when its task branch is on origin (`Topology.TaskBranchOnOrigin`).
  `beamTarget` carries the target's location, its slug (empty for the prime) and its label (`prime` or the slug).
  `startCmd` and `stopCmd` resolve the cwd through `lyxcwd` and call `resolveBeamTarget` themselves, never through `runVerb`, whose `ResolveBattenPrime` would refuse first inside a task pair with no path and inside a held beam with no `lyx batten status`, leaving this refusal and `BeamUp`'s hold refusal unreachable from there.
- `startCmd` takes `[slug]` and `--all`, refuses both together, and runs, after `resolveBeamTarget`:
  `hubreconcile.Ensure` over the target's `hubgeom.ReconcileGeometry`, its failure the verb's error with the reconcile's own way forward, the one config reconcile after a binary change any beam verb makes (`PATTERN-config-layers`); then `orchcli.BeamUp` with `Prepare` and the operator index of `cmd.Root()`.
  `--all` names the prime and every local beam pair this machine owns, from `dobbyengine.OwnedWorktrees(hubgeom.DobbyStart(prime))`; `startBeams` runs `up` for each, the prime first with `prepare` true and the rest with it false, so the hub is reconciled once, and gathers one `startEntry` per beam, a failure in one recorded on its entry and joined into the verb's error while the others still start.
  `startEntry` carries the label, the dobby outcome and detail, the orch action, prompt source and abandoned session, the error, and the prime's reconcile, failed pushes, owner pauses and mutations; the verb exits non-zero when any entry failed.
- `stopCmd` (`lyx beam stop [slug]`) resolves its target the same way and calls `orchcli.BeamDown`, printing what it stopped; it takes no `--all`.
- `statusCmd` prints, per beam, the dobby section as today and the beam's orch, `orchView`: whether the orch strand is live (`orchcli.OrchLive`), whether its watcher is live (`orchengine.WatcherLive` over `orchcli.PrimePaths`) and the recorded session id; an orch view that does not read carries its error on the entry as a dobby view does.
- `Command` adds `stopCmd`, and its help and `doc.go`'s start, stop and status sections describe the verbs; `docs/overview.md`'s beam line names them.

Tests: `TestStartBeams` drives `startBeams` over a fake `up`: one target is prepared, the first; a failing middle target leaves the others started, sits on its entry and is in the error; an empty list starts nothing.
`TestStart_RefusesBeforeTheReconcile`, over a `hubforge` hub, reaches the task-pair refusal from inside a task pair, naming its beam and path; the unknown-slug refusal naming `lyx beam create`; an unknown slug whose task branch is on origin naming `lyx adopt <slug>`; a slug given with `--all`, refused; and, from a directory outside every hub, the outside-hub refusal; each exiting non-zero with no orch state written under the target; the owner, anchor-move and hold refusals are `BeamUp`'s, reached in card 10.
`TestStatus_PrintsThePrimesDobbyAndEachLocalBeams` asserts each entry's orch view: no strand, no watcher and no session for a worktree whose orch never started, and, for a beam whose orch state records a session id and whose watch lock the test holds, that session id and a live watcher.

What it lets through, bounded: `lyx beam start <slug>` acts on another beam from anywhere in the hub, and every refusal of `BeamUp` still applies to that beam; `--all` reaches only beams this machine owns.

**ImpactSummary:** beamcli's start verb is replaced and stop is new; the verbs call orchcli's exported entry points and the status gains read-only orch reads, with no other package calling beamcli.

**Commit:** 11: `lyx beam start [slug] [--all]`, `lyx beam stop` and an orch view in `lyx beam status`
