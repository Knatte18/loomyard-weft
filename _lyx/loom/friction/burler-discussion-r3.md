# Burler fix, discussion round 3: the review cites text the target does not hold

The round-3 review quotes and line-numbers a decision record with the round-2 fixes applied (D16's `origin/prod` sentence, D17's deploy-key-gated fallback, D11's "every Warn line and" wording).
The `decision-record.md` on disk holds none of them, and the support log still reads "No rounds yet", so the round-1 and round-2 fixer edits are missing from the target while their fixer reports are present.

Each finding had to be revalidated against the older text.
F1's premise still held and was fixed in an adapted form, and F2's premise was false and was disputed.
Whoever owns the loop should check why the earlier rounds' edits did not survive to this round's target.
