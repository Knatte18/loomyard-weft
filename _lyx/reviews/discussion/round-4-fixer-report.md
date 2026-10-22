# Round 4 fixer report — discussion

## Changes

- **F1** (BLOCKING), fixed differently.
  `PATTERN-owner-machine` in PATTERN changes now gates a run on a machine that does not read the entry as owned by another.
  The proposal said "an entry with no owner recorded reads as owned by no other machine". I did not use that wording, because D2 line 47 says a task no run holds and with no owner recorded is owned by its parent's owner.
  The entry instead says that a run-held entry with no owner recorded, and every entry of a hub with no prime owner recorded, reads as owned by no other machine, and that any other unowned entry follows its parent's owner.
  The Goal's "lands only what it owns" now reads "only what no other machine owns", which removes the same gap there.
  I left the Constraints bullet unchanged: it names the entry and lists the reads, but does not restate the gate.
- **F2** (MEDIUM), fixed as proposed and narrowed at the self-check.
  D5 step 1 says the reconcile asks `LivePair` only for fast_forward or needs_replay branches.
  Adopt now calls `LivePair` of its own `OriginSeams` after the reconcile. It does this for every named slug that the reconcile reads as owned_elsewhere and that has a local pair.
  The stop, the `--drop-local` refusal and the `--keep-local` record all read that answer.
  The line-243 criterion now names adopt's own call, and adds the test that reaches the refusal through a hub-only `SeamOptions.HubPaused` cover.
- **F3** (MEDIUM), fixed by the reviewer's first option, with the way forward changed at the self-check.
  D6 now states why the old machine's reconcile never reads an adopter-first slug as ended. A `finalizeEntered` Finalize history row and `parentHoldsTaskWork` holding after the repair together make `withoutFinalizeEntered` drop it.
  It also states that the step 2b records commit is replayed, not set aside.
  The halt note names `lyx fabric remove <slug>` after the repair and leaves the owner pause standing.
  I dropped the proposed `lyx resume <slug>` lift: the self-check showed that lifting the pause frees the old batten, whose run record is not done, to act on the removed pair.
  For the old-machine-first order, D6 now has `lyx resume <slug>` lift the pause so that its Land runs.
  Open risks names the replayed records commit, the batten record left behind and the standing pause.
  The line-252 criterion follows both orders.
- **F4** (MEDIUM), fixed as proposed and widened at the self-check.
  D6 keeps the lock only while both signals hold.
  The first signal is that the child's status reads CurrentProducer Finalize, either running or failed with a non-empty `Transient` and no restep settle record yet for that stop.
  The second signal is `parentHoldsTaskWork`.
  The release rule, criterion 250 and a new row for a run stopped at Finalize with the parent already holding the work, whose lock is released, follow that.
- **F5** (LOW), fixed as proposed.
  D7 limits the skip to the shed driver's in-process re-step. A re-entry through InnerRun's `Spawn` seam meets D3's refusal and pauses the pair, and D7's Bound and Open risks now say so.
  The past-merge skip also covers the unreachable-origin halt. Finalize then proceeds as it does today, and its single transient re-step returns there before the tackler takes the stop.
  The line-254 criterion now covers both cases and the `Spawn` re-entry.
- **F6** (LOW), fixed as proposed.
  The halt note lists the local parent commits origin lacks and names the squash by its SHA.
  It advises `git reset --keep` only when the squash is the only such commit.
  Otherwise it advises `git rebase --onto <squash>~1 <squash> <parent-branch>`, with `git rebase --abort` and a drop by hand if the rebase conflicts.
  The line-252 criterion and Open risks carry the same condition.
- The support log has a Round 4 section and two question-ledger lines for the round's auto-picks.

## Self-check

- fix-consistency, MEDIUM: D6's running-only signal let the owner pause hold the driver's transient re-step that D7 promises. Fixed with my own change: D6's signal gained the failed-with-`Transient` arm, and the criteria followed.
- fix-consistency, LOW: D7's Bound, the Goal and the PATTERN entry did not mention the `Spawn` re-entry. Fixed for D7's Bound. Disputed for the Goal and the PATTERN entry on the ground that the premise is false: their "bar" exempts a past-merge Finalize from the gate without promising that it finishes, so they do not contradict the pause.
- fix-consistency, LOW: criterion 250 still treated "already merged" as the whole condition. Fixed as proposed.
- fix-claims, MEDIUM: the old-machine-first order dropped the pause lift, but the owner pause holds Land. Fixed as proposed: `lyx resume <slug>` restored for that order.
- fix-claims, LOW: the unreachable-origin re-step repeats only once. Fixed as proposed: "one transient re-step … before the tackler takes the stop".
- fix-proposal, MEDIUM: adopt's `LivePair` call would stop slugs that this machine owns and drives. Fixed as proposed: the call is limited to owned_elsewhere slugs.
- fix-proposal and fix-way-forward, MEDIUM (merged): `lyx fabric remove` followed by `lyx resume` frees a watching batten over a removed pair. Fixed with my own change: no lift after the removal; the pause stays and Open risks names the record left behind. Ending that batten is the out-of-scope `run-process-ownership`.
- fix-proposal and fix-way-forward, LOW (merged): the rebase could conflict and the squash was not identified. Fixed as proposed: the squash is named by SHA, with `git rebase --abort` and a drop by hand.
- fix-proposal, LOW: the timing of D6's two-signal rule against D7's in-process re-step, which the fork could not confirm. A duplicate of the fix-consistency MEDIUM finding, and fixed by the same change.

## Disputed

- Part of the fix-consistency LOW self-check finding, the Goal and the PATTERN entry twins, on the ground that the premise is false. Both say "bar finishing a landing already past its parent-side merge" and "bar the rest of a Finalize already past its parent-side merge". That wording exempts such a Finalize from the gate. It does not promise that the Finalize finishes, so a `Spawn` re-entry that D3 refuses and D6 pauses does not contradict either of them.

## Deferred

Nothing deferred.
