# A beam in landing cannot get its orch back after a restart, and a ready flag cannot be withdrawn

`beam-landing-orch-and-unready` · Note · bug, pawl, landing, orch, fabric

Imported from [issue #47](https://github.com/Knatte18/loomyard/issues/47).

## What happened

Hub NORCE-DrillingAndWells/Models, beam `duplication-and-import-followups`, flagged with `lym beam ready`; its pawl run went to PR-Review and opened PR #267.
The beam's orch kept running through PR-Review. The machine was then restarted.

1. `lym beam start` in the beam refused: "beam ... is held by its landing run (status running · PR-Review); way forward: ... run lym beam start again once the landing has ended". The orch that ran before the restart cannot come back, although lym let it run during the same landing.
2. The PR-Review reviewer session died with the restart. On the dobby's re-invoke, PR-Review's push of the task branch hit `fabric.push.lock still held after 30s` (a push the restart left running) and the row blocked, so no new reviewer started.
3. The operator then decided not to land yet and to add one more task to the beam. lym has no verb that withdraws a beam's ready flag or a landing in progress. The workaround was `lym loom goto --to Beam-Await` plus `lym loom resume` in the beam, closing the PR by hand with gh, and keeping the ready record in place, so the beam's dobby will land the beam again as soon as no work remains under it.

## Expected

- `lym beam start` brings back the orch of a beam in landing, as the landing tolerates a live one, at least up to Finalize.
- A verb to withdraw a ready flag or a pending landing (e.g. `lym beam unready <slug>`), which takes the pawl run back to Beam-Await and closes the landing's pull request.
- PR-Review waits longer for, or recognises as stale, a push lock held by a push from before a restart.