# Round 2 fixer report — discussion

## Changes

All changes are in `_lyx/discussion/decision-record.md`.

- **F1**: fixed as proposed, with one addition.
  `withdraw` now removes the caller's own queued injects and its own unread mail to the receiver, journaled with reason `withdrawn` (`expired` for an inject, `undelivered` for mail), in Scope, D5's Fresh launch and D5's Hold bullet.
  The Bound says a message the orch already read stays read.
  The criterion now says a relaunched, continued or adopted orch is reminded of no note request from the old cycle.
  Addition: `withdraw` answers the messages it removed, and when one is a pause-resume line, `lyx orch start` clears the pause entry's resume-noticed mark, so a continued or adopted session gets the line again and a relaunched one does not.
  D9 says so.
- **F2**: fixed as proposed.
  A failed withdraw or release is recorded in the orch state, and the new watcher retries both as `ORCHWATCH` on every tick, posting and queueing nothing (the bind color included) and starting no cycle until both succeed, with the failure shown as State.Stuck.
  Step injects carry a named TTL of ten minutes (creel's `defaultInjectTTL`), and the Bound says an old `/clear` or `/compact` can reach the session for at most that TTL.
  I added a new criterion for the refused-withdraw retry, and Scope line 22 now lists the new orch-state record.
- **F3**: fixed as proposed, with additions.
  The in-pane test is gone.
  `NotifyPrime` claims the non-orch strand, retries as its program only on creel's "runs in no strand pane" refusal, and posts as its program otherwise.
  The Bound says a non-orch-pane caller never signs as a program.
  Additions from the self-check:
  - a claim to a strand creel no longer lists is signed EXTERNAL;
  - a claim from another strand's pane and the sender-resolution refusal get no retry and are dropped;
  - the detached-caller behaviour is Linux-only.
  The detached-caller sentence and criterion 222 now match.
- **F4**: fixed as proposed.
  D5 names the run's events file as the turn-end source and the transcript as what a turn did.
  The claude engine's event parser returns each Stop's stamp time, and a Scope In bullet lists that shuttle provider-seam change.
  A turn end counts only when its stamp's second is after the `sending` second, and a Stop with no stamp never counts.
  The whole-second Bound covers both directions.
  Criterion 219 pins a turn end stamped in the `sending` second.
  D4's `EventWaiting` source now says the events file, and Scope's doc list adds `shuttleengine`.

## Self-check

- fix-consistency, LOW: the Hold bullet's reset withdraw didn't mention mail. Fixed as proposed.
- fix-consistency, LOW: D9 didn't mention the withdrawn resume line. Fixed as proposed.
- fix-consistency, LOW: Scope line 22 didn't list the new orch-state record. Fixed as proposed.
- fix-consistency, LOW: Scope's doc list was missing a shuttle package. Fixed as proposed (`shuttleengine`).
- fix-consistency, LOW: D4's `EventWaiting` still named the transcript. Fixed as proposed.
- fix-claims and fix-proposal, MEDIUM (merged): withdrawing the resume line loses it for `--continue` and `--adopt`. Fixed with my own change: `lyx orch start` clears the resume-noticed mark when a withdrawn message is a resume line. I didn't spare the mail from withdraw, because the note request needs it.
- fix-claims, fix-proposal and fix-way-forward, MEDIUM (merged): a claim to an unlisted strand is signed EXTERNAL with no retry. Fixed as proposed, as a D6 Bound sentence and a criterion clause.
- fix-claims, MEDIUM: the sender-resolution refusal gets no retry. Fixed as proposed, as a D6 Bound sentence.
- fix-proposal, LOW: a claim from another strand's pane is refused with no retry. Fixed as proposed, as a D6 Bound sentence.
- fix-proposal, LOW: the whole-second truncation still lets a turn ending in the second after `sending` count. Fixed as proposed: the D5 Bound and criterion 219 now say so.
- fix-way-forward, LOW: the detached-caller behaviour is Linux-only. Fixed as proposed.

## Disputed

None.

## Deferred

None.
