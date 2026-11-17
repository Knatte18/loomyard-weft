# Fixer report — Discussion-Review round 3

Target: `_lym/discussion/decision-record.md`.

## Changes

- F1 (MEDIUM), fixed as proposed.
  D1 says the fix producer records `opts.DecisionsRouted` as `decisions_routed` in the round's snapshot record.
  D5 says the fix producer writes it before it spawns or skips the fixer, since only the fix row carries `NotifyParent` and `decisions_escalated`,
  and that a record without it leaves its round incomplete for D2.
  D2's completeness test judges the fix check with that flag, and the held mark is now keyed on it.
  D3's round context and the validate-review builder read it.
  Added a D2 acceptance row for a notify-only fix row and for a record lacking the flag.
- F2 (MEDIUM), fixed as proposed.
  D3's `carries` bullet says a carried finding keeps its earlier finding's class;
  the round-check refusals and the D3 acceptance row now cover a changed class.
- F3 (MEDIUM), fixed as proposed, taking the "record the bound as accepted" option.
  D3's held-finding bullet states that Go checks only that the `answer` quotes a listed title, that an unrelated listed entry passes,
  and that the block's content bounds this: only entries added through `lym loom decision add` since round N-1 began.
  Added an Auto-mode assumption for that bound.
  The id-in-title alternative was not taken: a parent that omits the id would leave its answered finding escalated again every round.
- F4 (MEDIUM), fixed as proposed.
  D3's last paragraph now limits the skip to hub, mode and engine wiring.
  It resolves the worktree through `lymcwd` alone, from the seam cwd, and notes that `_lym` links out of the worktree.
  It names the builder's inputs: the run directory, the round, the decision record path, the anchor path and round N-1's `decisions_routed`.
  Updated the D3 validate-review acceptance row, and added a Review criterion for the verb's help text.
- F5 (MEDIUM), fixed as proposed (recommended option).
  D6 says that in a window-open round whose ledger holds no design-wrong entry, the stalemate check yields,
  so the round settles as today.
  Added a D6 acceptance row: auto-accept off, no design-wrong entry, accepted by `softCapRule` and not escalated.
- F6 (LOW), fixed as proposed.
  D5 stores each added entry's title together with a sha256 digest of its heading and body, as `discussionparser.AddedDecisionTexts` returns them, and keys on the digest.
  D2 lists an entry whose digest the snapshot lacks.
  Deleted the second open risk and adjusted the D2 acceptance row on replace and add.
  D2 had no trailing sentence about a repeated title, so there was nothing to delete there.

## Self-check

- fix-claims, LOW: "the parent" was named as the only writer of added entries, but the arbiter and the operator also add them.
  Merged with fix-proposal LOW 3. Fixed as proposed: the bound now reads "entries added through `lym loom decision add`".
- fix-claims, LOW: "`softCapRule` accepts the round" overstates the yield when an escalation record exists.
  Fixed: "the round settles as today, by `softCapRule`'s accept when no escalation record exists".
- fix-consistency, LOW: the Review criterion did not name the carried-class rule. Fixed as proposed.
- fix-consistency, LOW: Constraints lacked `PATTERN-cwd-resolution`. Fixed as proposed: added a Constraints entry.
- fix-way-forward, MEDIUM: a seam cwd in one worktree and a review file in another worktree's run directory would build the wrong context.
  Merged with fix-proposal MEDIUM 1. Fixed as proposed: D3 gives shape checks alone when the run directory is not under the resolved worktree's `_lym` run root;
  the D3 acceptance row covers the case.
- fix-proposal, LOW: the class lock's cost is not stated.
  Fixed: an Auto-mode assumption says a finding whose class proves wrong is marked `outdated` and raised anew within D4's bound.
- fix-proposal, LOW: same as the merged fix-claims LOW above.

## Disputed

None.

## Deferred

None.

## Rulings

None.
