# Adopt runs as a detached, recorded worker that pauses adopted runs

## What changes

`lym adopt` used to run every step in the caller's process, so an adopt started from the orch died with the orch's session mid-deploy.
It now validates its arguments, starts a hidden `lym adopt worker` in its own session, and follows the worker's record.
The caller prints each step as it ends, exits 0 when the record ends `done`, and exits non-zero when it ends `stopped` or reads `interrupted`.
In an agent session the follow stops at the keep-warm cap with `still running`, naming `lym beam status` and the re-run of `lym adopt`.

The worker writes an adopt record, `adopt.json`, and holds `adopt.lock` under `fabricengine.AdoptScratchDir`.
A re-run with the same subjects follows a live worker or resumes an unfinished record.
A re-run with other subjects is refused, naming the recorded subjects and `lym adopt --discard`.
`lym adopt --discard` removes an unfinished record and refuses while the lock is held.
The record type, its path and a lock-free read live in the new `internal/adoptrecord`.

The steps run as `reconcile`, `pairs`, `pause`, `owner`, `deploy`, `beam`, `check`, `done`.
The owner write now comes before the deploy, so this machine does not read a foreign owner during the deploy.
A refused step ends the record `stopped` at that step, and the stop keeps the part that step gathered in the end envelope.

The adopt hold and `holds.json` are gone.
The `pause` step writes a `pauserecord.AdoptCaller` entry (`adopt`) for every adopted run and every released, unstarted, non-front task of an adopted subject.
It turns an existing own-scope `owner` entry into an `adopt` entry in place, including an adopted beam's own pair slug.
Only `lym tender resume` or `lym batten cancel` lifts it.
A green adopt check, the dobby's owner lift and a hibernate wake leave it standing.
The run report's `waiting_for_check` and `held_failed_check` states become `paused_by_adopt`, which names `lym tender resume <slug>`.

The green check end now only removes the fix task, resets the try count and posts its notices.
It no longer lifts pauses, brings up the dobby or orch, starts first passes, or resumes owner-halted children.

The `beam` step starts each subject's dobby, and its orch unless the request carries `--no-orch`.
`orchcli.BeamUpOptions` gains the orch-skip field, and the caller sets `--no-orch` when it runs in an orch session.
`lym beam status` gains an `adopt` section for a running, interrupted or stopped record, with the step and the way forward.
`runproc` gains an adopt role, and the docs, `PATTERN-dobby-runs` and `PATTERN-daemon-wakeups` describe the worker, the record and the pause.

## Design decisions

- The worker is a second entry into the adopt steps, callable from any shell.
  A direct call is bounded by the same validation, the subject match against the record, and `adopt.lock`.
- `lym fabric anchor` ends a worker mid-step; the record then reads `interrupted` and a re-run resumes it.
- A resume past `owner` starts at its first step not done.
  A resume at `owner` or earlier restarts at `reconcile`, so the owner write follows a fresh reconcile.
  A call adding `--keep-local` or `--drop-local` also resumes at `reconcile`, since those flags act only there.
- `--discard` lifts no pause and undoes no owner write.
- A check started by an older lym and still running across the upgrade runs the old green end once.
  That end can lift legacy holds, start dobby and orch, and resume owner-halted children.
  A live legacy `adopt-hold` entry reads like an `adopt` entry and is lifted only by hand.
- Open front tasks under the subject can start once the owner write stands, before the deploy.

## To check by hand

- A failed step's gathered part is kept in the record and merged into the end envelope:
  confirm the reconcile report and mutations survive a refused reconcile, and that the clearing of that part when the step later succeeds is tested.
- The reconcile report's own `mutations` field comes back empty in the end envelope, while the top-level `mutations` lists every entry.
- Reconcile warnings: confirm they are attributed to the right step.
- Some planned tests overlap; check the new tests in `internal/adoptcli` for redundancy.
- The way forward for a project held back by a local-commit stop (`--keep-local` or `--drop-local`) can be refused by the standing record; try the re-run in `internal/adoptcli`.
- A resume at `reconcile` clears the steps done; check that this is the intended effect on the record.
- The integration suite's `final` gate failed once on `github.com/Knatte18/loomyard/internal/pattern`, and passed on rerun; triage called it flaky and unattributed to this change.
