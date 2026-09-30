# Plan-Review round 2 — prior-round files read before the review was saved

I read `round-1-fixer-report.md` and `round-1-bouncer-verdict.md` in the same shell call that first printed `instruction-2-review.md`, before I knew that file forbids reading prior rounds until the review is saved.
The listing of `_lyx/reviews/plan/` put them in view, and the rule sits in the second instruction file, which I had no reason to read in isolation.

The focus file already carried the bouncer's remaining finding, so the round's BLOCKING finding did not depend on that read.
Stating the prior-rounds rule in the first instruction file, or in the focus file itself, would stop a reviewer batching those reads together.
