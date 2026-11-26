# Webster: plan fixes after begun batches, and recovery that ends asking

Two Webster runs halted on plans that were valid or fixable.
In the first, a plan fix that edited `00-overview.md` outside its Card Index was later refused by Webster's rebaseline.
Plan validation then refused targets as `glyph-not-found` on members the plan's cards introduce or modify.
In the second, a recovery strand ended `dead`/`asking`.
Its question was lost, the batch had no retry, and the run needed a manual `webster reset`.

Closes #221
Closes #220

## What changes

- **Plan gate and the overview frame.**
  `runplan.RecordedOverviewFrame` is now the one reader of a run's recorded overview frame,
  and falls back to the stored baseline copy when the record holds no frame hash.
  `loomshed.ValidatePlan` reads the frame through it, as the rebaseline does.
  A record without the hash, which the gate used to skip, is now checked against the copy's frame.
  With no frame either way, any edit to the recorded overview raises `overview-frame-edited`, since the rebaseline refuses it.
  The plan review rubric tells a fixer that the overview outside its Card Index is frozen once a batch has begun.
- **Glyph targets a card creates.**
  A glyph target that spells the member named by a `Create` `plan:` handle passes on the creating card and every card after it.
  On an earlier card, the `glyph-not-found` detail names the creating card.
  A missing member of an existing unit now says that no card creates it,
  and lists the members declared beside it: those of its owner type, or the unit's owner-less members.
- **A recovery with a question.**
  The recovery prompt tells the strand to report `status: NEEDS_PARENT` with its question as the `need`.
  Merriam sends it to the parent and resumes with `lym webster recover-batch NN --answer`, spending no recovery.
  The resumed prompt holds each need beside its answer.
  A resume of a waiting recovery with no answer is refused.
- **A recovery that ends its turn with no report.**
  It stays `dead`/`asking`.
  Its final message, joined onto one line, is the digest's new `last_message`,
  quoted by the `recover-batch` refusals, the dead-end refusal and Merriam's `stuck_reason`.
- **A card edit re-arms a dead batch.**
  A rebaseline that accepts a `--card` edit to a terminal dead batch now records it in `AmendedCards`.
  A failed or stuck batch that already holds an entry takes the edit too.
  A dead recovery holding an unrendered amendment earns one more uncounted recovery, whatever its count.

## Design decisions

- The issue offered two routes for a recovery that asks, and both are taken:
  the question reaches the parent, and a plan edit to the dead batch's cards re-arms a recovery.
- Only an explicit `NEEDS_PARENT` report counts as a question, since almost every turn end carries a closing message.
- A failed or stuck batch holding no amendment is not re-armed by a card edit.
- The created-member match is on the target's spelling or its parsed ID against the `Create` handle's body.
- The member listing costs one quarry query per missing member that no card creates.

## To check by hand

- Finding: every recovery turn end that carried a message became `waiting`, not only a question
  (`internal/websterengine/recoverbatch.go`, `awaitTerminal`).
  The diff keeps such a turn end `dead`/`asking` and only records the message; check that this holds.
  A strand that asks without writing a `NEEDS_PARENT` report therefore gets its question quoted, but not routed.
- Finding: the sibling listing for a missing member of a Go external-test `<dir>_test` unit had no test
  (`internal/planglyph/planglyph_test.go`, `TestValidateFormat_MissingMemberFindings`).
  The diff adds a `sub_test#Missing` row; check that it covers the package-directory fallback in `siblingMembers`.
- The new recovery prompt, Merriam template and plan review rubric wording has not been tried in a live run.

## Merge danger

**Door:** two-way
**Blast radius:** loom
