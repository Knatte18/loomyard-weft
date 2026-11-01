# lym adopt holds taken-over runs until the subject's adopt check is green

`lym adopt <beam|root>...` and `lym adopt --all` become the one takeover verb that leaves every run under the adopted subjects in a known state.
Before this change adopt resumed each moved run as a detached first pass, so adopting also started work, and afterwards neither the operator nor the orch could tell whether a run had finished, should run or should not.

Now the runs wait for the adopt check of their subject and start by themselves when it ends green.
A failed check files a darn fix task in that subject, the only run that starts; when the fix lands the check runs again, up to `adopt_fix_tries` times, then the failure goes to the operator.
Adopt prints one line per run.
There is no release verb and no reed button: the only hand lifts are `lym tender resume` and `lym batten cancel <slug>`.

## What changes

- **Hold.** Adopt writes a `pauserecord.HoldCaller` entry (new, beside `OwnerCaller`) before its owner write, for each moved run not done whose owner is another machine, and for each released, unstarted task under an adopted subject bar a front task.
  The held slugs are listed in a per-subject `holds.json` below `fabricengine.AdoptScratchDir`.
  A run whose own scope holds another caller's entry, such as an operator's pause, keeps it and gets no hold; a run whose pair is live here, or that reads owned elsewhere, gets none either.
  Dobby gives a held run no pass of any kind, and its owner-pause recheck owes no lift pass for it.
- **Adopt check per subject.** `lym adopt check <subject>` runs in the subject's worktree (the prime for `root`, the beam's pair for a beam), with its record, lock and output under a directory of its own.
  `lym beam status` and the prime's status-bar mark show each subject's check.
  Adopt starts the check of each subject it recorded an owner for, and of each subject with a hold still standing, so a re-run after a failure starts a new check.
- **Green end.** Lifts the subject's holds through `tenderengine.Lift`, brings the pair up through `orchcli.BeamUp`, starts the first pass of each lifted run (and `lym loom resume` for a child halted at Finalize's owner check), removes the subject's fix task, and resets the try count.
- **Failed end.** Files a darn fix task `adopt-fix-<subject>-t<N>` (kind task, profile `light`, labels `bug` and `gate`) holding the failing tiers, tests, `suite.log` excerpts and a machine-failure stop condition.
  `<N>` is one more than the highest found among the board, the run records and the task branches, so a slug never repeats across a hand re-adopt, a machine switch or a cleanup.
  The count lives in `fix-tries.json`; only a hand `lym adopt` and a green end reset it.
  With the count at the limit nothing is filed and the notice names the failing tiers, `suite.log` and the re-run.
  The `gate` label is filed only when the hub's board vocabulary knows it; otherwise the task carries `bug` alone.
- **Run report.** The envelope gains `runs`, one item per run, sorted by slug, with the states `owned_elsewhere`, `landed`, `held_failed_check`, `waiting_for_check`, `paused`, `halted`, `ended` and `running_here`, the row, `owned_by` where it applies, and a plain `line`.
  The per-slug outcomes stay.
- **Origin reconcile.** `fabricengine.ReconcileOrigin` stays adopt's only sync.
  A prime or adopted-beam branch left unmoved as `rewritten`, `diverged` or `ahead` is an envelope warning naming the branch, its state and `lym fabric pull` in that worktree.
- **Front tasks (`internal/fronttask`, new).** A general piece for a task lym files ahead of a worktree's other starts, reusable by note `parent-merge-task`.
  The board entry gains `front` and `on_land`; while a front task is open its worktree's dobby starts no other task there, and starts it past `max_runs`.
  Dobby runs the task's `on_land` action once when it lands, from a closed action table whose one row is `adopt-check`, and only while this machine owns the fronted subject.
  `boardengine` refuses a changed `front` or `on_land` on `lym board upsert`, `upsert-batch` and `merge`, and a change of project, kind or slug of a front task, each refusal naming its way forward.
- **Darn writer stop.** The darn writer stops, with no commit, on a condition the task body names, and writes a second marker so `internal/loomshed/darnwrite.go` puts the stop report's own way forward in the halt reason instead of `lym loom resume`.
- `adopt_fix_tries` (default 3) is a new key of `beam.yaml`.
  Help text and package docs of adopt, dobby, the board and the darn writer follow.

## Design decisions to know

- The hold is a pause-record entry of a distinct caller, so dobby's owner-pause lift and the hibernate wake never treat it as theirs.
  It is written through `pauserecord.Update`, never `tenderengine.Pause`, so it waits no grace and sends no Esc.
- A held run takes a `max_runs` slot; the fix task starts anyway because a front task starts past the limit.
- Holds, tries and check record live in separate files so the check's own record writes never touch the list or the count.
- The pause record is machine-local, so a hold never reaches the other machine.
  A path that ignores pauses is not held.
  `lym tender resume` lifts a hold by hand and the run then goes on without the check.
- `lym adopt check <subject>` on a machine that does not own the subject runs nothing and names `lym adopt <subject>`.

Out of scope: a release verb and reed button (ruled out), the `parent-merge-task` command and recipe, keeping a run's binary version across the move (note `run-pins-its-version`).

## To check by hand

- Tiers 2 and 3 of the gate did not run on this change; run them before landing.
- `lym adopt` end to end on a second machine: reconcile, holds, owner write, check, green lift and a red check that files a fix task.
- `internal/fronttask`: confirm the test of `startAction` pins the process spawn of the `on_land` action.
- The fix-task removal in `internal/adoptcli` (the settle step called while the holds-file lock is held): confirm no lock is held across a call that takes it again.
- The adopt and board upsert help pages stay within the Long cap.
