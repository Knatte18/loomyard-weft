Webster-Burler round 2: deciding what PATTERN-comment-line-breaks allows

`PATTERN-comment-line-breaks` says "one sentence per line, with a further break at an independent-clause boundary".
It does not say whether a break at a comma before a compound predicate, a list item or a relative clause is a violation, or whether the further break is required or only allowed.
The diff's authors break at such commas often, so the answer decides most of this round's comment sites.
I treated breaks at non-clause commas as violations and breaks before an independent clause as allowed, not required.
I settled that only partway through the fix, so the review lists seven sites the fix then left alone.
One sentence in the background file on which comma breaks are allowed would have saved that pass.
