# Round 1 fixer report — creel-cross-hub-mail discussion

F6 is a `decision` finding and is with the parent; this report leaves it out.

## Changes

- F1, fixed as proposed: D3 gains a bullet saying `deliver` and `confirm` skip the sender resolution that `internal/creelengine/server.go` runs before dispatch, and a bullet saying any answer to `deliver` other than `accepted` or `refused`, an unknown-verb refusal included, is transient. The D3 criterion gains the pane-process case.
- F2, fixed as proposed: D3 step 6 repeats the duplicate check under the store lock and answers the held id when it finds one. The D3 criterion gains overlapping deliveries; the replay Bound now cites steps 3 and 6.
- F3, fixed as proposed: D3 step 5 answers `yes` only for a pending message with no remote delivery and no return journaled. The D3 criterion, the rationale and the replay Bound name the return. The self-check then closed the remaining race (see below).
- F4, fixed as proposed: D6 says the text header of `lym creel read` carries the stamp while its JSON keeps `from` bare. The D6 criterion now says so.
- F5, fixed as proposed: D5's rationale is narrowed to committed changes, and D3's list of routes the check does not stop gains the trusting hub's `creel` base file.
- F7, fixed differently: the proposed carry rule ("younger than the seven-day retention counted from its post time") never holds when retention runs, since retention carries only out of files older than seven days (`internal/creelengine/doc.go`). D2 now carries an outbound message only while it is pending, so a delivered message's reply wait lasts as long as its day file. The F10 bound and the support log's bound-breaker 9 line were adjusted to match. D4's late-reply bound is unchanged.
- F8, fixed as proposed: D3 step 1 keeps only the shape checks. The receiver rules now apply in step 3, after the duplicate check, so the step numbers F6 cites still hold.
- F9, fixed as proposed: D1 drops an entry only when its hub path is gone or its shortname record reads another valid shortname, and keeps one whose record cannot be read. The D1 criterion now says so.
- F10, fixed as proposed: D2's outbound-event bullet gains the downgrade Bound.
- F11, fixed as proposed: the expects-reply mark joins the confirm tuple and the sending daemon's match in D3 steps 4 and 5.
- F12, fixed as proposed: D3 gains the probe-and-flood Bound, with the cost stated as "at most" after the self-check.
- F13, fixed as proposed: the second Open risks bullet is dropped. The self-check kept its transport fact in Scope Out.
- F14, fixed as proposed: the D4 criterion gains the replier's replied mark and the settle that sends no notice. The D6 criterion gains the reminder's stamp and `lym creel log`'s hub path and id.
- F15, fixed as proposed: D1's rationale keeps only its first clause. The created-or-opened alternative moved to the support log's Rejected alternatives, and the turn-3 board-scope line points there.
- The support log gains a Round 1 entry under Review rounds.

## Self-check

- fix-proposal, MEDIUM: F3 narrows the return race without closing it, and the rationale claimed it was closed. Fixed differently: rather than only narrowing the rationale, D2 makes an hour-cap return wait for a `deliver` in flight, so a return and an `accepted` never cross. The rationale now says a confirm after a return answers `no`. A D2 criterion was added.
- fix-way-forward, MEDIUM: a `refused` answer to a `deliver` in flight at the return would return the message twice. Fixed differently: the same wait rule makes this case impossible, so no rule for ignoring a late answer is needed.
- fix-way-forward, LOW: an hour-cap return's notice gives no reason. Fixed: D2 names the last transient answer as the reason, and the D2 criterion asserts it.
- fix-claims, LOW: the F7 carry condition never holds. Fixed: see F7 above.
- fix-consistency, LOW: the support log's board-scope 1 line was stale after F15. Fixed as proposed.
- fix-consistency, LOW: the support log's leanness line about the open risk was stale after F13. Fixed as proposed.
- fix-consistency, LOW: no criterion covered the transient answer from F1. Fixed: a D3 criterion was added.
- fix-proposal, LOW: F13 removed the closed area's transport fact from the record. Fixed: the fact is in Scope Out.
- fix-proposal, LOW: the flood cost was stated as exact. Fixed: it now reads "at most".

Every fork returned a report.

## Disputed

None.

## Deferred

None.
