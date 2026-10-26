# Beams: lyx beam start, ownership on projects, pawl recipe

A beam, the prime included, is now one unit of two parts that run together on the machine that owns it: its dobby and its orch.
`lyx beam start` brings it up, ownership sits on projects instead of tasks, `lyx adopt` moves a project, and a beam's life is one batten run over its project entry with the `pawl` recipe.

## What changes

- **Start and stop.**
  `lyx beam start [slug] [--all]` replaces `lyx orch start`: hub reconcile, origin reconcile, root claim, then the beam's dobby (only that beam), window and orch with its watcher.
  The orch resumes its recorded session, else the last handoff note, else the start stencil; a session that does not come up falls back to the note and the envelope names it.
  `lyx s` is `beam start` followed by `reed attach` and replaces `lyx prime`.
  `lyx beam stop [slug]` stops the orch, watcher and dobby and leaves a pawl run's driver and agents running; `lyx beam status` shows both parts.
  `runproc` gains `EndRole` for that.
- **Removed.**
  `lyx orch start` (with `--continue`, `--handoff`, `--adopt`), `lyx orch cleanup`, `lyx prime`, `lyx beam promote` and `lyx adopt --prime`.
  Every refusal, notice, stencil, skill and doc now names `lyx beam start`; `TestRemovedVerbs_NamedNowhere` scans for the removed verbs.
- **Ownership on projects.**
  Only the root (board key `prime_owner`) and beam projects record an owner.
  Any other entry reads the owner of its `hubgeom.ParentOf`.
  A task's leftover `owner_machine` is ignored and dropped on the next write; the run-held ownership rule is gone.
  A run in flight whose old task owner differs from its project's owner is owner-paused naming `lyx adopt <project>`.
- **Adopt.**
  `lyx adopt <project>...` and `lyx adopt root` move the named project alone: owner, pair and the runs under it, then bring up its dobby and orch.
  A task or grouping project is refused naming the beam or `root`.
  A beam's pawl run is resumed once this machine owns both the beam and its parent.
- **Orch watcher.**
  It stops the orch of a beam another machine owns.
- **`pawl` recipe.**
  `beam-recipe.yaml` becomes `pawl-recipe.yaml`, with rows Beam-Up, Beam-Await, Parent-Align, the Code-Review perch, Beam-Describe, Finalize and Beam-Down, implemented in the new `internal/pawlengine`.
  A seed recording recipe `beam` reads as `pawl`.
  `lyx beam create` starts the run; `lyx beam land` writes a land request naming the run, which ends Beam-Await.
  Batten's start guard accepts a beam project and refuses a grouping project; `lyx batten cancel <beam>` reconciles with origin and refuses while the beam has members.
- **Notice.**
  A beam's dobby posts one "beam can land" notice per episode naming `lyx beam land <slug>`, or `lyx adopt <parent>` when another machine owns the parent.

## Design decisions to know

- A grouping project has no owner: it has no worktree or dobby, so its tasks follow the beam or root they fork from.
- The landing hold is judged from the producer half of the beam entry's run status.
  The run holds none from Worktree-Create through Beam-Await, so members can still join and fork; it holds one from Parent-Align on, and a member's Worktree-Create then removes its pair and is refused.
  Finalize re-checks `BeamLandRefusal` under the parent lock as a backstop.
- Beam-Await waits in its own driver process, with no agent, and stops the beam's dobby when it accepts a request; a failed re-check restarts it.
  A dobby-stop marker and an await marker carry the state across a crash.
- `BeamLandRefusal` no longer blocks on a live orch or watcher, since the orch lives until Beam-Down.
- `PATTERN-owner-machine` and `PATTERN-run-process-owner` are changed with the code: `runproc`'s `End`, `EndSparing` or `EndRole` end a run's processes.
- Upgrade bound: an older build refuses a project carrying a recipe, so until every machine of the hub is updated through `update-plugins.sh` it fails to upsert a beam project entry and can overwrite its run status and owner.

## Check by hand

Not run by hand; the following were left open in review:

- Landing-deps reflective test and the `nilLegal` list: the pawl seams are not clearly covered by the reflective landing-deps test, and `nilLegal` does not list the legal nils.
- Orch-start removal and the prime shortcut: confirm no path still depends on `lyx orch start` for the prime, and that `lyx s` attaches only after a successful start.
- Pawl engine tests: confirm the `internal/pawlengine` tests carry the build tags the test tiers require.
- Pawl recipe env nil-seam fixtures: confirm the recipe tests' env fixtures cover every nil seam.
- Beam-Await's dobby restart after a failed re-check is not bounded: a failed restart leaves the marker standing until the next accepted request or `lyx beam start <slug>`.
- A Beam-Await driver that dies repeatedly stops being re-spawned at the respawn cap, and the notice text differs only when the cap is hit before the first driver-dead notice.
- The first verify of the full suite failed on `cmd/lyx` and `boardcli.TestCLI/Profiles`; a fix commit (`72edea22`) set the pawl profile default in the boardcli test and the batten cancel help text under its cap.
  Check both are cleared on a full run.
- Exercise `lyx beam create`, `lyx beam land`, `lyx beam stop` and `lyx adopt` across two machines; only unit and tmux-tier tests ran.
