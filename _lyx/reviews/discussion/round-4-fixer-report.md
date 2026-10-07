# Round 4 fixer report

All six findings are fixed in `_lyx/discussion/decision-record.md`.
`support-log.md` is unchanged.

## Changed

- F1 (resume branch, MEDIUM): D6's resume paragraph now says the review-ready function returns the bytes it parsed, that an attached half's outcome is judged by the fresh-round rules (an attached fixer done before the marker, or an attached half that dies or times out while the other lives, stops the other half and ends the round), and that D7's comparison runs once the attached fixer is done.
  It also covers the neither-live case (archive, spawn as attempt 1).
  The resume acceptance criterion gained a bullet for the failure branches and the post-handoff review-file comparison, and the neither-live case.
- F2 (failed removal, MEDIUM): D6 now says stopping a half is remove-if-live, that a failed removal or liveness probe becomes the round's error in place of the failing half's outcome, and that nothing is spawned beside an unremovable strand and the retry does not run.
  The round-failure acceptance criterion carries the same.
- F3 (usual resume state, LOW): D6's Bound now says a stopped fixer's target edits stay for the next reviewer, and D6's Rationale gives the reason a live fixer beside a finished review is restarted rather than attached.
- F4 (early-release bound, LOW): D5's Bound now says lyx code writes the marker, admits that the fixer could create it or name another existing file to the verb, and names the prompt as the bound for both.
  The Open risks entry names the second route.
- F5 (marker location, LOW): the Constraints entry now states the standalone location as D5 does.
- F6 (restated gate behavior, NIT): D5 keeps the mechanism sentence; D6 and D7 state only their own consequence.

## Deferred

None.

## Notes

No constraint conflicted with anything else in the prompt.
