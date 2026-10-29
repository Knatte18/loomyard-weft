# Fixer report — plan, round 1

## Changes

- **F1** (fixed as proposed): card 6's Tests gain the tier1 bullet changing the superseded hook-wait subtest's newer-waiter stderr from `creelengine.BellLine` to `hookLine(1, alpha, "mail")` and `hookLine(2, alpha, "more mail")`, each with its newline, exit 2 unchanged.
- **F2** (fixed as proposed): card 6 lists `contracts/stencils/orch/orch-template-role.md` under **Edit:**, and its Intent rewrites step 4: the mail-line sentence adds the wake, an idle orch is woken with its mail under `[creel] Mail that arrived while you were idle:` and acts on it in that turn, and only an orch no waiter reaches gets the fixed bell line, after which its hooks hand it the mail as that turn ends.
- **F3** (fixed differently): the proposal put the three client-side D3 Bounds and the Constraint's two exceptions into card 4's `internal/creelengine/doc.go` text, with the loss window "recorded by the acknowledged log line".
  I did not carry that out.
  Card 4 builds only the daemon side; the hook showing the mail arrives with cards 5 and 6, and the `acknowledged` line arrives with card 9, so card 4's doc would state behaviour its tree lacks (the fix-consistency fork found this after the first pass).
  Instead, card 6 adds `internal/creelengine/doc.go` to its targets and writes the loss window, the double showing with the bell that may follow, the tag escaping, and the claim paragraph's two delivered-once exceptions.
  Card 4 keeps the pause-window Bound and names the ring's claim among the delivered-once paths.
  Card 9's doc sentence adds that the `acknowledged` line records the ids the loss window loses as read.
- **F4** (fixed as proposed): card 4's Ring paragraph says the locked step claims only when the pause read ran for a marked waiter before it.
  A marked waiter found without that read, one that superseded an unmarked waiter in between, rings nothing and stays registered, and its registration's wake has the worker probe again.
- **F5** (fixed as proposed): card 4's Intent adds the attack-surface sentence.
  The mark lets a strand's own waiter take that strand's claimable mail, the boundary claim's set, journaled read at its acknowledgement and skipping only the Stop cap and the Stop count.
  It is bounded by `HookWait`'s strand-only refusal, the pause read, the hold, `hookFieldCap` and the 8 KiB budget.
- **F6** (fixed as proposed, option 1): card 3 confines the release to claim and ring readings.
  `Reading` gains the unexported `read bool`, which `Store.Read` sets, and a read's Ack keeps today's failure branch.
  Card 3's targets gain `Reading`, `Store.Read` and `TestStore_Mail`, the last with the row `AJournalFailureAtAReadsAckKeepsTheIdsItLeftUnjournaledReserved`.
  The doc sentence and the ImpactSummary follow, and so does the overview's Card Index entry for card 3.
- **F7** (fixed as proposed): card 9's `released` bullet logs only when the reading held ids or echo items, and `TestStore_ClaimLog` gains the row for an empty claim released, which logs the Debug `nothing` line alone.
- **F8** (fixed as proposed, first option): card 7's row `ThePausedOrHeldStrandsClaimStillHandsOutTheEcho` releases the paused claim, so the echo stays owed.
  The held claim, with the pause cleared, hands it out again.
- **F9** (fixed as proposed): card 10's new subtest narrows its assertion to the stop-hook summary.
  The first post's token appears once in its feedback lines, and no second `Stop hook feedback:` row and no `Stop says:` line carries any post's token.
- **F10** (fixed as proposed): card 1 replaces "seven `HookOutput` rows" with "a `HookOutput` row for each of the following".

`lym loom validate-plan` returns `ok` after every change.

## Self-check

- fix-consistency, LOW, the overview's Card Index still states card 3's old scope: fixed as proposed (entry now says "a claim's acknowledgement").
- fix-consistency, LOW, card 3's `read` field and card 9's missing path encode the same fact: fixed as proposed (card 9's `awaitAck` Warn keys on card 3's `read` mark).
- fix-consistency, LOW, card 4's doc text describes hook-wait behaviour only card 6 builds: fixed as proposed, by moving the client-side Bounds and the exceptions to card 6, whose targets gain `internal/creelengine/doc.go` (see F3).
- fix-affected-set, LOW, `postCmd`'s Long still says "an idle one woken first": fixed as proposed (card 6 targets `internal/creelcli#postCmd` and its Intent rewrites that sentence).
- fix-claims: no findings.
- fix-proposal, MEDIUM, card 10's "second Stop hook feedback line" reads as line two of the one clamped row: fixed as proposed (now "no second `Stop hook feedback:` row").
- fix-proposal, LOW, card 4's F5 sentence goes stale at card 7, since a ring hands out no echo: fixed with my own change.
  The proposal did not name an edit, so I added a sentence to card 7's Intent.
  It says a ring measures the budget over mail alone, and that card 4's "the set a boundary claim would hand" holds from then on for the mail with the echo set aside.

## Disputed

None.

## Deferred

None.
