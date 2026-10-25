# Fixer report — discussion, round 2

Every finding's premise was checked against the code and held.
All ten are fixed in `_lyx/discussion/decision-record.md`, with a Round 2 entry and Q21–Q23 in `_lyx/discussion/support-log.md`.

## Changes

- F1, BLOCKING, fixed as proposed, then widened by the self-check: every owned-elsewhere refusal of a full pass exits with its own code; dobby pauses the run through `tendercli.PauseOwnedElsewhere`, releases its parent lock, sends the owner notice naming `lyx adopt` once, marks the run owned elsewhere and passes it again only at the owner pause's lift. D3's claim that InnerRun's owner check covers a takeover found by a full pass is gone; D7 names the owner notice; one acceptance criterion added.
- F2, BLOCKING, fixed as proposed: the spawn hold runs the owner check on a due probe before yielding, and an owned-here verdict clears `ownedBy` and spawns in the same pass (D4); criterion added.
- F3, BLOCKING, fixed as proposed, first option: a `PATTERN-owner-machine` change says a continue pass is not a resume, and D4's continue-pass paragraph names the entry.
- F4, MEDIUM, fixed as proposed: D4 names a pass's one check as `check()`'s full body with probe-due from the persisted `lastProbe`, and the inventory's last bullet replaces `ownerSleep`.
- F5, MEDIUM, fixed as proposed: an inventory bullet names the respawn count and its status mtime, the api-error episode revived, and the revived-episode count with its status mtime.
- F6, MEDIUM, fixed as proposed: the full pass reports an `origin unreachable` line and the full pass stays owed until one reaches origin (D4); criterion amended.
- F7, MEDIUM, fixed as proposed, first option: `lyx adopt` of a member refuses while its parent beam is owned elsewhere (D2, Scope); criterion amended.
- F8, MEDIUM, fixed as proposed: an upsert turning a note into a task gives `draft`, and a null-status upsert of an existing task or a merge's null `set_status` step is a release (D5); criterion amended.
- F9, LOW, fixed as proposed: Land's deploy (`deployTimeout`, 30 minutes) and the spawn's synchronous `lyx loom start --no-attach` bootstrap are in D4's in-pass waits.
- F10, LOW, fixed as proposed, first option: D10 states the cleanup miss as an accepted bound.

## Self-check

- fix-consistency, MEDIUM, the lift pass of a run marked owned elsewhere is a continue pass by D4's definitions: fixed, the full pass stays owed while marked, and every pass with a full pass owed is a full pass (D4, PATTERN change).
- fix-consistency, LOW, a dobby start and a wake still pass a marked run: fixed, no pass but the lift's while the mark stands.
- fix-consistency, LOW, and fix-way-forward, MEDIUM (merged), a marked run's place in `max_runs` and D13's backoff is unsaid: fixed, it counts in neither (D4, D5, D13).
- fix-consistency, LOW, the two takeover routes are not contrasted: fixed, D4 says InnerRun's own owner finding sets no mark.
- fix-consistency, LOW, Scope and the Question ledger miss the adopt refusal and the F3 pick: fixed, a Scope bullet and Q21–Q23.
- fix-claims, MEDIUM, the exit code covers only `resumeRefusal`'s arm, and `refuseResumeUnreachable` can refuse: fixed, the code covers `resumeOwnerClaim`'s lost claim and the local-Bolt refusal too, and the unreachable sentence says it judges the local owner.
- fix-way-forward, LOW, and fix-proposal, LOW (merged), dobby cannot see the exit of adopt's own first pass: fixed, recovery goes through the lift adopt makes of the owner pause (`internal/adoptcli/doc.go` step 4).
- fix-way-forward, LOW, a failed owner pause is never retried, so no lift can come: fixed, retried on each probe tick while the mark stands.
- fix-way-forward, LOW, the prime has no `lyx adopt <slug>` form: fixed, the refusal names `lyx adopt --prime` for the prime.
- fix-proposal, MEDIUM, F6's owed episode re-runs Open-Window on every offline probe: fixed, later full passes of that episode run with a hidden `--no-reopen` flag; criterion amended.
- fix-proposal, MEDIUM, adopting a beam away leaves members owned by the old machine with no dobby: fixed, `lyx adopt` of a beam refuses while a not-done member records another owner, unless the same call adopts it (D2, Scope, criterion).
- fix-proposal, LOW, F8's merge wording and a creating null-status upsert are ambiguous: fixed, the release names a merge's `set_status` step setting null and an existing task, and a creating write gives `draft` regardless.

## Disputed

None.

## Deferred

None.
