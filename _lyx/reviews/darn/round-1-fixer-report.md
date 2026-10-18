# Fixer-report, round 1

## Changes

- F1 (fixed as proposed): row `NoticeAReadHoldsReservedIsRemindedNotTypedAndJournaledReadOnce` in `TestScheduler_Reminders` reads a notice without acking, asserts only the reminder is typed, then acks and asserts the log holds posted, reminded and one read.
  The row pins the snapshot's reservation split; `markNoticeRead`'s guard needs a read landing between snapshot and send, which no test hook reaches, so it stays unpinned.
- F2 (fixed as proposed): `postCmd`'s help says agent-strand mail is announced by a reminder and read, and that EXTERNAL or program mail is carried in the typed line.
  The self-check added the line-cap exception and the same wording in the `creel` group help.
- F3 (fixed as proposed): the creel bullet in `docs/overview.md` says non-agent mail is carried in full in the typed line when it fits the line cap.
- F4 (fixed as proposed, the comment option): `markNoticeRead` names the retype after a released read.
- F5 (fixed as proposed): the hold row gained a non-owner program notice (new `webster` constant) that stays untyped under the hold, and the first notice row gained a queued inject typed before the notice.
- F1 and F5 share one commit, since both changed the same test file.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-consistency, LOW: the `creel` group help repeated the stale reminder claim. Fixed as proposed.
- fix-affected-set, LOW: same group-help twin. Merged with the above.
- fix-affected-set, LOW: `tools/sandbox/SANDBOX-CREEL-SUITE.md:114` describes agent mail and is correct. No change.
- fix-code-correctness, MEDIUM: help and overview said carried mail needs no read, but over-cap mail is still reminded. Fixed as proposed in the three texts.
- fix-code-correctness, LOW: the F1 row does not reach the `markNoticeRead` guard. Not fixed, since it needs a race hook; recorded above.
- fix-code-correctness and fix-proposal, LOW: the F4 comment joined two clauses on one line. Fixed as proposed.
- fix-proposal: F1 guard gap, same as above. No other findings on F2, F3 or F5.
- No fork returned without a report.
