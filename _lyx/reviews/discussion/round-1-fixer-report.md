# Fixer report — discussion, round 1

Every change is in `_lyx/discussion/decision-record.md`; `support-log.md` needed none.

## Changes

- F1 (BLOCKING), fixed as proposed: D1 reads coverage after `awaitIdleSession` returns and immediately before `typeVerified`, allows an earlier read only as an early exit, and a criterion covers a pause added while a send waits for idle.
- F2 (MEDIUM), fixed as proposed: D6 records the orch with the orch strand's reed GUID (`State.Strand`), and a criterion covers an orch claimed by D7 getting no second Esc and its resume line.
- F3 (MEDIUM), fixed as proposed, second option: the orch runner is told the record path for D1's read only, through a setter that leaves `waitWhileCovered` unarmed, and its own start lines skip the read, so `lyx orch start` keeps starting under a hub pause; a criterion and a Bound sentence state it, and D2's start bullet names the exception.
- F4 (MEDIUM), fixed as proposed: D4 holds a Verify before starting its command, after the dirty and skip checks, with the `paused` marker; a criterion covers it.
- F5 (MEDIUM), fixed as proposed: D4's cancel sends SIGKILL to the group after the delay through `verifyrun`'s own timer, whether or not the shell has exited; the stop criterion covers a member that ignores SIGTERM.
- F6 (MEDIUM), fixed as proposed, first option: the SIGTERM-first cancel applies to every `verifyrun.Run` caller, named in D4, and the Scope Out bullet now excludes only their pause hold.
- F7 (MEDIUM), fixed as proposed: D8's failed lookup has its own refusal text pointing at `lyx tender status`, a Bound sentence, and a criterion.
- F8 (LOW), fixed as proposed: a failed name lookup falls back like an unparsable name with one Warn per change of error, and a Bound sentence states what both fallbacks let through.
- F9 (LOW), fixed as proposed: D1 excepts `SendConditioned` and states that tender revive reads coverage itself; the PATTERN line says the same.
- F10 (LOW), fixed as proposed: D7 states the resume window as an accepted residual and no longer claims the pane lock excludes a resume; added to Open risks.
- F11 (LOW), fixed as proposed: D3 states that an `Expired` finalize under cover waits in D4's held Verify; the Open risk names an `Expired` or liveness finalize and drops the deadline.
- F12 (LOW), fixed as proposed, no-activity option: D4 says `agentActivityFor` and `readAgentActivity` read a `paused` marker as no activity, because the loop watchdog already skips its idle timeout while covered and batten's quiet notice treats a held run the same; a criterion covers it.
- F13 (LOW), fixed as proposed: the plan-writer note says a held Verify at the loomshed and webster gates ends only on the resume or the end of its process.
- F14 (LOW), fixed as proposed: D2 gives `DeliverPendingPrompt` its own bullet, which waits while covered and sends again, and the start bullet keeps the prompt line and the skill-load line.
- F15 (LOW), fixed as proposed: one criterion each for a resume before the grace end, a cancel during the hold, creel interrupt's remaining refusals, a no-running-turn strand staying recorded, record-batch, await-batch and status under a pause, and an unreadable record letting `begin-batch` run.

## Self-check

- fix-consistency, LOW, D1's Rationale still claimed every caller: fixed as proposed (the Rationale names the two exceptions).
- fix-consistency, LOW, the `PATTERN-refusal-way-forward` constraint named one D8 refusal: fixed as proposed (it now covers both refusal texts).
- fix-consistency, LOW, D4's title covered only a running command: fixed as proposed (retitled).
- fix-way-forward, MEDIUM, D8's failed-lookup way forward strands an idle Master: fixed differently. The proposed in-turn rerun on a backoff would have an agent poll with no bound under a pause of any length. The stranding is now an Open risk that needs the operator to prompt Master.
- fix-claims: no findings.
- fix-proposal, MEDIUM, the delayed SIGKILL timer had no stated lifetime: fixed as proposed (`Run` returns only once the group is empty, polled through `kill(-pgid, 0)`, or the SIGKILL is sent; the stop criterion says so).
- fix-proposal, MEDIUM, a SIGTERM-first timeout let batten act while the group lived: fixed as proposed (the same return rule; D4 says callers act on the timeout only once the group is gone).
- fix-proposal, LOW, the PATTERN line had no orch start exception: fixed as proposed.
- fix-proposal, LOW, a pause landing during `typeVerified`'s replay or deferred submit: fixed as proposed (the read repeats before each replay, and the deferred-submit window is an Open risk).

## Disputed

None.

## Deferred

None.
