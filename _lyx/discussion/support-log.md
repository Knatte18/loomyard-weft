# Support log — plan check projects lint-pattern per card

## Interview

A darn run holds no interview: the writer read the board entry and the code and made every call itself.

## Rejected alternatives

- Replacing the whole-plan `pattern-headroom` check with the per-card one: the projection starts at the worktree, so growth already in the worktree against the merge base would go unflagged.
- Reusing the `pattern-headroom` check name with a card attached: a reader counting findings by check could not tell the whole-plan cut from a card's cut.
- Crediting an unnamed removal after the last card: a plan whose removing card describes the entry without its ID would fail at plan review although its card gates pass.
- Rewriting prose card references in `insert-card` and `remove-card`: a prose number may already mean the new numbering, so a rewrite can corrupt a correct reference; the per-card check removes the cause of the repeated renumbering instead.

## Review rounds

_No rounds yet._

## Question ledger

- Should the plan check project per card? Resolved, self-pick: yes, as a new `pattern-card-headroom` check beside the whole-plan one.
- Which card does a removal belong to? Resolved, self-pick: the first pending card naming the backticked ID, else before the first card.
- Should the renumber verbs rewrite prose card references? Resolved, self-pick: no, out of scope; filed as a board note.
