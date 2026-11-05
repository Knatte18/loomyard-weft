```yaml
lens: fix-proposal
outcome: reported
```

Challenging each reviewer proposal the fixer carried out (fix-proposal lens).

Proposals examined: F1 (drop event and orch notice), F2 (JSON trust fields), F3 (final `deliver` before a return), F4 (self-describing untrusted stamp), F5 (new inbound event), F6 (whole-tuple duplicate match), F7 (stamped reply size check), F8 (Open risks line dropped), F9 (config load in the constraint).

- MEDIUM. Changed text: D3 step 3, "an id clash … is answered `refused` with a reason naming the clash."
  Summary: after a hub is re-created at the same path, every new message whose id collides with one the receiver still holds is refused and returned, and a re-post collides again.
  Evidence: ids are allocated as `p.maxID++` over the folded journal (`internal/creelengine/fold.go:182-185`), so a re-created hub restarts at 1 and climbs through the receiver's held ids one per post. Every collision returns undelivered, and the refusal names no way forward. Before the change the loss was silent. Now it is visible, but the hub cannot reach that receiver until its ids pass the receiver's held maximum or the held messages age out of retention. The proposal needs a way forward in the refusal, or a stated bound.
- LOW. Changed text: D3 step 3's clause "a hub re-created at the same path restarting its ids".
  Summary: it contradicts the Constraints first bullet, left unchanged, which still states "ids per hub that are never reused".
  Evidence: the review's F6 location names "Constraints first bullet", and `fold.go` `newID` confirms that ids restart when the journal folds empty.
- LOW. Changed text: D6's new JSON bullet, "the `reply`, `returned` and `unanswered` fields … when they hold such a message".
  Summary: `returned` and `unanswered` never hold a message from another hub, so the clause asks the plan writer for fields on a case that cannot occur.
  Evidence: a returned message is the waiter's own outbound message (D2). D4's bound says a cross-hub `--wait-reply` ends at its timeout, never as unanswered, since the settle sends nothing back.
- LOW. Changed text: the deleted `## Open risks` bullet (F8).
  Summary: its twin under `## Auto-mode assumptions`, "A confirm that answers `no` drops the message instead of showing it as unverified (D3)", still lists as an auto-pick what the operator's added decision now settles.
  Evidence: the `Added after Discussion (operator, 2026-11-05)` entry, and the Q6 ledger line, which now reads "settled by the operator's added decision".
- LOW, outside this lens: the fixer's own D7 bound says "a trusted one carries the authority the old text gives it anyway". The old trust sentence trusts any orch strand, but the old parent directive grants delegate status only to "any orch strand of this hub". So a pre-deploy session in `lyt` does not treat `ly:orch` as its delegate under the old directive. This is for the fix-claims lens.

F1's notice target is reachable: `checkReceiverLocked` accepts the hub's fixed orch name even with no live strand (`internal/creelengine/pool.go:215-222`). I found no case against F3, F4, F5, F7 or F9.

Tool calls:
- Grepped `internal/creelengine` for `newID` and the unknown-receiver error, to check id restart and whether `<shortname>:orch` is a valid receiver. My context held neither function.
- Read `fold.go:175-195` and `pool.go:215-240`, the bodies of those functions, for the same reason.
