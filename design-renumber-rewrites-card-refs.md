# insert-card and remove-card could rewrite prose card references they now only report

`renumber-rewrites-card-refs` · Note · enhancement

Split from task plan-check-lint-pattern-per-card (issue #866).

`lyx loom insert-card` and `remove-card` renumber the cards after the edit and report every prose line naming a moved card by its old number (`planparser.RenumberResult.ProseMentions`), leaving the fix to the writer.
A renumber spanning many cards then needs every reference fixed by hand, and repeating one doubles the cost.

Open question: a prose number may already mean the new numbering, so an automatic rewrite needs a form that marks a reference unambiguously, such as a card id `<N>-<slug>`, rather than rewriting bare numbers.