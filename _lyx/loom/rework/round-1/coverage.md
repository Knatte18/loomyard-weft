# Rework coverage — PR #504, round 1

One finding, one card.

- **Finding 1 (MEDIUM): every advisor gets every question batch** — card 10.
  The chair stencil's advisors section drops the area assignment and sends every batch, formed as Step 3 says, to every advisor, settling each question in autonomous mode after weighing every advisor's answer; interactive mode stays as it is.
  The chair-stencil row of `wordingClaims` pins the new wording with `must` claims and the retired area wording with a `mustNot` claim; no other test asserted the old text.
  The decision record's D5 and D6 sentences about areas are superseded by the review and not edited.
