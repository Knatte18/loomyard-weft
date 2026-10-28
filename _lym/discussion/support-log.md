# Support log — fixer-overcap-review

## Interview

A darn run holds no interview: the writer read the board entry and the code and made every call itself.

## Rejected alternatives

- A new recipe row for the solo review, between Finalize and its reroute target: rejected because each recipe would need a new row and edges, and the review would run outside the parent lock, letting the parent move under an approved change.
- Running the full Code review with a fan of one: rejected because the Code segment reviews the whole task's diff, not the fixer's range, and its Bouncer and Burler rows carry far more machinery than one verdict needs.
- A separate `fix_reviewer` model-spec and timeout in `landing.yaml`: rejected as unasked; the fixer's keys serve.
- Approving on a review that ends without a verdict: rejected, since an uncertain end must fall back to today's full review.
- Sending a spent fixer to the solo review too: rejected, since a spent fixer leaves a failing gate that no approval can land.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (self-pick): Where does the review run? Inside Finalize under the lock (D1).
- Q2 (self-pick): Which ends of the review land? Only an explicit approval (D3).
- Q3 (self-pick): Which model? The fixer's model-spec and timeout (D5).
- Q4 (self-pick): Does a resume re-review? Not at the approved HEAD (D6).
