# Fixer report — discussion, round 1

## Changes

All changes are in `_lyx/discussion/decision-record.md`; `support-log.md` is unchanged.

- F1, fixed as proposed: Acceptance criteria bullet 3 now allows a waiting inject's stall notes before the answer, and bullet 4 says "one `creel: ` answer line after any stall notes".
- F2, fixed as proposed: the last Scope Out bullet and the third Open risks bullet now cover every refusal the client builds from an error after the daemon stored the post or inject, the wait timeout and a connection lost after the store, naming `Client.CallWatching`; the way forward stays `lyx creel log --since`.
- F3, fixed as proposed: D1 states its line goes straight to the command's stderr, never through `writeRefusal`, so no error envelope follows the failed success write.
- F4, fixed as proposed, then extended by the self-check: D1's "stored nothing" label is gone, and the Bound says the exit 1 does not undo what interrupt, forward, hold and release already did.
  The self-check added a `<done>` of `the <verb> took effect` for forward, hold, release and an interrupt that stopped a running turn (D1, D2, Auto-mode assumptions, D5 and a new hold acceptance criterion), so the caller's stderr line says the action took effect.
- F5, fixed as proposed: a Scope Out bullet says refusals cobra raises before a verb body runs are written by `clihelp` and keep their behaviour, and D5's `doc.go` bullet limits the fallback claim to refusals the verbs write.

## Self-check

- fix-consistency, LOW, Acceptance bullet 1 and D5's doc bullet kept the bare "one `creel: ` line" twin of F1: fixed as proposed, both now say one answer line after any stall notes.
- fix-way-forward, LOW, the no-id D1 line tells the caller of interrupt, forward, hold or release nothing about the action having taken effect: fixed with my own change, a `the <verb> took effect` `<done>` rather than a `lyx creel log` pointer, since the verb knows the outcome and the line can state it directly.
- fix-proposal, LOW, the cobra Scope Out bullet's "a wrong argument count" read as covering post, which checks its count in its body (`internal/creelcli/cli.go:275-277`): fixed as proposed, now "under `cobra.ExactArgs`".
- fix-proposal, LOW, Acceptance bullet 1 twin of F1: duplicate of the fix-consistency finding, fixed there.
- fix-claims, LOW, the D1 Bound's "the interrupt, forward, hold or release has taken effect" overclaims for an interrupt that found no running turn (`internal/creelengine/verbs.go:514-515` returns false with no journal entry): fixed as proposed, the sentence ends at "does not undo what the daemon already did", and the new `<done>` covers an interrupt only when it stopped a running turn.

## Disputed

None.

## Deferred

None.
