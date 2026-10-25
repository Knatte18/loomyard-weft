# Support log — plan-refs-auto-rewrite

## Interview

A darn run holds no interview: the writer read the board entry and the code and made every call itself.

## Rejected alternatives

- Pre-resolving a reported new glyph before rewriting, and leaving the plan untouched when it fails: rejected because the operator's decision names the exact tier's path, rewrite then revalidate, and a second order would make two repair routes.
- A separate post-batch step for reported renames beside `DetectDrift`: rejected because the deleted-symbol sweep and the candidate findings must exclude a repaired old glyph, which only the detector itself knows.
- Letting a reported rename override quarry's exact tier for the same old glyph: rejected because the AST-proven pair is the stronger evidence.
- Reusing the digest's `deviations` list for glyphs: rejected because it is the scope guard's file list and other readers depend on that meaning.
- Rewriting bare card numbers on a renumber: rejected because a bare number may already mean the new numbering, as the board entry's open question says.
- A new marker syntax for card references: rejected because the card id `<N>-<slug>` already appears in plans, in `--card` arguments and file names, and is unambiguous.

## Review rounds

_No rounds yet._

## Question ledger

- Q: What form marks a card reference unambiguously for the renumber rewrite? Self-pick: the card id `<N>-<slug>` and its padded file form; resolved.
- Q: Which amendment tier word marks a reported repair? Self-pick: `reported`; resolved.
- Q: Does recover-batch carry the reported renames? Self-pick: yes, through the digest; resolved.
- Q: Does a reported rename beat quarry's exact tier for the same old glyph? Self-pick: no, the exact tier wins; resolved.
