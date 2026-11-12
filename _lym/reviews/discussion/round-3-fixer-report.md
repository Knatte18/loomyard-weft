# Round 3 fixer-report — discussion

Every finding was checked against `internal/creelengine/outbound.go`, `deliver.go`, `fold.go` and `internal/slackengine/keeper.go`; each premise holds.

## Changes

- F1 (BLOCKING), fixed as proposed, in `_lym/discussion/decision-record.md` D3:
  - the hour is one hour from the post, as for a message to another hub;
  - a timer of creel's scheduler returns a machine-outbound message at its hour whether or not the relay runs;
    it wakes at the earliest pending hour and is armed by a machine-outbound post and the scheduler's start scan;
  - the start scan and post wake now "start no per-hub worker for it", instead of skipping it outright;
  - take returns only messages not yet past their hour, and returns a message past its hour before it answers;
  - new D3 criterion: return at the hour with the relay not running, under an injected clock; take never returns a message past its hour;
  - a Notes line says the hour timer is a scheduler loop of its own, apart from `outboundWorker.run`.
- F2 (MEDIUM), fixed differently, in D4:
  - the impossible "rename the archived channel" way forward is gone, and the evidence line (conversations.rename, `is_archived`, read 2026-11-12) is added;
  - not made as proposed: the unarchive, rename, archive-again sequence for every channel.
    The self-check (fix-proposal 2, fix-way-forward LOW) showed it splits a pair:
    unarchiving a channel either bot made lets the next try adopt it, then the rename leaves the peer computing a free name and creating a second channel.
    The way forward is instead: unarchive, and the next try adopts a channel either bot made;
    for a channel anyone else made, also rename it, then archive it again if wanted;
  - the relay still never unarchives;
  - the D4 criterion now covers both paths;
  - a Notes line keeps the plan writer from copying `keeper.go:196`'s rename text.
- F3 (MEDIUM), fixed as proposed, in D3:
  - report journals a `machine-held` event only on a change of reason, and that record is the carried last hold reason;
  - settle delivered journals `machine-delivered` with the Slack timestamp in its own field;
    settle returned and creel's own return journal `machine-returned`;
  - settle delivered is accepted for a returned message;
  - an older binary folds these events to nothing;
  - the retention criterion names the three events, and a new criterion requires one `machine-held` record for an unchanged reason;
  - the late-settle sentence names `machine-delivered`.
- F4 (MEDIUM), fixed differently, in D4 and D8:
  - not made as proposed: notice marks as bridge-map fields.
    The self-check (fix-consistency MEDIUM, fix-way-forward MEDIUM, fix-proposal 1) showed that D7's `peer-unapproved` hold and D4's `bridge-name-taken` and `bridge-archived` holds come before the pair has a map entry.
    All marks instead sit in a bridge-notice record beside the bridge map:
    a send-side hold's mark is keyed by pair (local orch and peer address), a read-side notice's mark by channel id;
    dropping a channel from the map leaves its marks;
  - Constraints name the bridge-notice record under `PATTERN-tolerant-records` and `PATTERN-lym-files`;
  - D8: an approval that delivers held messages moves the delivery cursor past each one;
  - new criteria: D4, a restart mid-episode posts no second notice, pair-keyed holds and unmapped channels included;
    D8, approve then revoke under a held message leaves a reread posting no second notice.
- `_lym/discussion/support-log.md`: Round 3 entry under Review rounds.

## Self-check

- fix-claims, MEDIUM: "as the `stalled` event is written once per change" misstates `noteStall`, which also rewrites a stale reading.
  Fixed as proposed: the text now reads "written on a change of reason".
- fix-consistency MEDIUM + fix-way-forward MEDIUM + fix-proposal 1 MEDIUM (merged): marks of holds that come before a channel had no home.
  Fixed as proposed: a pair-keyed mark in the bridge-notice record, and the restart criterion names those holds.
- fix-consistency, LOW: dropping a map entry silently cleared its marks.
  Fixed as proposed: the marks are out of the map, and dropping a channel leaves them.
- fix-proposal 2 MEDIUM + fix-way-forward LOW (merged): the unarchive-and-rename way forward splits a pair whose channel either bot made.
  Fixed as proposed: unarchive alone for a channel either bot made, and a rename only for one anyone else made; the criterion is updated.
- fix-proposal 3, LOW: a re-sent settle delivered after a crash could post a second notice.
  Fixed as proposed: a settle delivered for a message already journaled delivered writes nothing; the late-settle criterion covers it.

Every fork returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
