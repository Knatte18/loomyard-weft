# Fixing webster round 1, F1: a prescribed refusal no plan can reach

Card 4 prescribes a step-keyed zero-batch refusal after an accepted auto-rebaseline, and the review rightly flagged that no test reaches it.
Writing the `Run` row showed that no plan can reach it: the plan parser refuses a cardless plan before `Run` forms any batch, and the recorded batches then cover every card.
PATTERN-refusal-way-forward wants a test to reach the refusal, code-quality forbids paths for impossible states, and the card fixes the signature.
I kept the branch and pinned its text with a golden test of `zeroBatchesError`; a plan-time check that a prescribed refusal can be reached would have caught this before the code was written.
