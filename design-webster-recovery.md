# Webster recovers from a re-begin, its own canonicalization and an integration regression

`webster-recovery` · Planned · bug · running · Plan-Bouncer

## Goal

A webster run recovers on its own from the three states that today need a hand edit or a human: a re-begun batch, its own plan canonicalization, and an integration regression.

## Bugs

- #329: a re-begun batch whose fork landed nothing is validated as if its cards were built, so later cards that Use its Create targets are refused.
- #330: `rebaseline` refuses begun cards that `begin-batch` canonicalized itself, because the recorded card hashes keep the pre-canonical text.
- #332: an integration regression bisected to a card ends the run as `blocked` with no fix attempt, although the batch forks had seen the failures.

## Fix direction

- #329: only a batch that landed commits counts as built; a begun batch with nothing landed validates like a first begin.
- #330: when webster rewrites a begun card, it updates that card's recorded hash in the same step, so its own edits never count as drift.
- #332: a batch fork that sees a test its own card broke fixes it before reporting done.
  An integration regression bisected to a card gets a recovery fork against that card before the run escalates.

## Done when

- Each issue's "Seen" scenario has a regression test that passes.
- CLAUDE.md's TEMPORARY items for #329, #330 and #332 are deleted (the issues are already closed into this task).
