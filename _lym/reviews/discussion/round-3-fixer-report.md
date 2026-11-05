# Fixer report — discussion round 3

## Changes

- F1 (fixed as proposed): D10 adds the board-wide refusal: `checkNewSlugs` also refuses a slug the raddle store holds as a Sketch's or a filed slug, reading the store as it stood before the write, so D9 step 3's own write passes, and exempting the entry whose `created` stamp the store records for its own Sketch slug.
  D10 states that `lym board raddle <slug>`, D11's lookup and D14's listing rely on this refusal.
  D2's start guard and D5, D6 and D8's free-slug checks are unchanged.
  A new D10 acceptance bullet covers upsert, rename, note promotion and a re-used pruned project slug.
  A new Constraints Fact records that `checkNewSlugs` today consults only `Config.ValidateSlug` and no store.
  Support-log Q21 is revised.
- F2 (fixed as proposed, criterion fixed differently in one clause): D9 step 5 releases each filed part still on the board and unreleased, skips one absent or done, and its done marker is that no filed part still on the board is unreleased.
  The D9 retry acceptance bullet gains the case of a run interrupted after step 5 and resumed after a released part landed and was pruned.
  Not made: the proposal's "after step 4" half of that case. Before step 5 no `beam` part is released, so none can land or be pruned (D9: "for `beam` they are filed unreleased"), and a retry after step 4 is already covered by the bullet's "finishes the remaining steps and repeats none".
- F3 (fixed as proposed): the Open risk on `pawl-orch-landing` is deleted; a Constraints Fact records Beam-Up's re-invoke interrupt policy from that entry's D11 (`_bolt/decision-pawl-orch-landing.md:162`).
  D11 states the order, mail first and then the sent-marker, with the duplicate hand-off accepted on an interruption between them, and its acceptance bullet states both cases.
  Support-log Q23 records the pick.
- F4 (fixed as proposed, placed one sentence later): D12 gains the failure rule: a locate or read failure logs at Warn and still writes the session's file with the cost unknown, and a failed write is logged and stops nothing.
  The sentence follows the no-`TranscriptReader` sentence it refers to, not the one before it.
  The D12 acceptance bullet gains the unreadable-transcript case.

## Self-check

- fix-consistency, LOW: board refusal meets D2's case first with another way forward — fixed: D2 says the board refusal meets such a Sketch at promotion first with the same way forward, and D10 names `lym board rename` for a promotion.
- fix-consistency, LOW: support-log Q11 still says "once" — fixed: Q11 revised.
- fix-consistency, LOW: support-log Turn 3 still names the deleted Open risk — fixed: the Turn 3 line notes round 3 replaced it with a Constraints Fact.
- fix-way-forward, MEDIUM: a refused promotion names a slug promote cannot take — fixed: D10 and its acceptance bullet name `lym board rename` of the note before the promotion. Merged with fix-proposal LOW 1.
- fix-way-forward, LOW: the offered slug may itself be a store slug — fixed: D10's way forward names a slug both the seam and the store admit.
- fix-claims, LOW: `checkNewSlugs` returns at once on a nil seam, so "every write" is false — fixed: D10 says the store refusal runs whether or not the seam is set, and the Fact says the rule checks nothing on a nil seam today.
- fix-proposal, LOW: a filed part made a note again is refused on promotion — fixed: D10 names that case and the rename-first way forward; the refusal is intended, and D2 already refuses any promotion of such a discussed note other than a raddle one.
- fix-proposal, LOW: the pawl Fact cites a record pruned at that entry's Land — fixed: the Fact also names `loomshed.InterruptPolicies`, where that D11 puts the policy.
- fix-proposal, LOW: Claude's `SessionUsage` reads the same transcripts, so the fallback recovers no counts, and a fork's failed read is unstated — fixed: D12 and its bullet say the counts `SessionUsage` holds, each unknown where it could not read, and the rule covers any of the session's transcripts, a fork's included.

## Disputed

None.

## Deferred

None.
