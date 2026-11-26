# Webster: plan fixes after begun batches, and recovery that ends asking

Two Webster runs halted on plans that were valid or fixable.
In the first, a plan fix that edited `00-overview.md` passed the plan gate, and Webster's rebaseline then refused it.
A later card's target was also refused as `glyph-not-found`, though an earlier card creates that member.
In the second, a recovery strand stopped to ask a question.
The question was lost, the batch went dead with no retry, and the run needed a manual `webster reset`.

## What changes

- **Plan gate and the overview frame.**
  `runplan.RecordedOverviewFrame` is now the one reader of a run's recorded overview frame.
  It falls back to the stored baseline copy when the record holds no frame hash.
  `loomshed.ValidatePlan` reads the frame through it, as Webster's rebaseline does.
  A record without the hash, which the gate used to skip, is now checked against the copy's frame.
  With no frame either way, any edit to the recorded overview raises `overview-frame-edited`, since the rebaseline refuses it.
  The plan review rubric tells a fixer that the overview outside its Card Index is frozen once a batch has begun.
- **Glyph targets a card creates.**
  `planglyph` accepts a glyph target that spells the member named by a `Create` `plan:` handle.
  It passes on the creating card and on every card after it.
  On an earlier card, the `glyph-not-found` detail names the creating card.
  A missing member of a unit that exists now says that no card creates it,
  and lists the members declared beside it: those of its owner type, or a Go package's owner-less members.
- **A recovery that asks.**
  A recovery strand whose turn ends on a message and writes no report is recorded `waiting`.
  Its message, joined onto one line, is the `need`.
  Master's existing waiting rung sends the question to the parent, and `recover-batch` resumes the batch without spending a recovery.
  `gatefixer.LastTurnMessage` reads the message from the strand's events.
  A turn end with no message stays `dead`/`asking`.
- **A card edit re-arms a dead batch.**
  When a rebaseline accepts a `--card` edit to a terminal failed, dead or stuck batch, it now records the edit in `AmendedCards`.
  `RecoveryRetry` gives a dead recovery that holds such an amendment one more recovery, whatever its count.
  So a plan fix to the dead batch's cards clears the dead end without a reset.
  `RebaselineResult.CardsAmended` still lists only cards of in-flight batches.
- Specs, package docs, the `recover-batch` help and Master's failure ladder describe the new behavior.

## Design decisions

- Of the two routes the entry offered for a recovery that asks, both are taken.
  The question goes to the parent through the existing waiting mechanism, so the operator sees it.
  A plan edit to the dead batch's cards also re-arms one recovery.
  Fork strands that end asking are unchanged: their recovery is the existing answer.
- The created-member check matches the `Create` handle's canonical body against the target's spelling and its parsed ID.
  A target spelled differently from the canonical glyph still gets `glyph-not-found`, which now lists the members beside it.
- The member listing costs one quarry query per missing member, and only when validation would refuse anyway.

## Tests

- `planglyph`: `TestValidateFormat_MissingMemberFindings` covers a use after the creating card, a use before it, and the member lists for a type and a package.
- `loomshed`: rows in the overview-frame-edited gate table cover a frame from the baseline copy, a missing copy, and a record with no overview hash.
- `websterengine`: a recovery whose turn ends on a message is recorded waiting with the exact need.
  `TestRecoveryRetry` has rows for an amendment that has been rendered and one that has not.
  The rebaseline rows for terminal batches assert the recorded amendment.
- `gatefixer`: `TestLastTurnMessage`.

## Merge danger

**Door:** two-way
**Blast radius:** webster
