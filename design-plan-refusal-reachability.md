# Plan review does not check that a prescribed refusal is reachable

`plan-refusal-reachability` · Note · enhancement

Imported from [issue #516](https://github.com/Knatte18/loomyard/issues/516).

A plan card prescribed a refusal branch (a zero-batch refusal after an accepted auto-rebaseline) with a fixed signature.
Review of the implementation flagged that no test reaches the refusal; the fixing agent then found that no plan can reach it, because the plan parser refuses a cardless plan before `Run` forms any batch.
The branch was kept and its text pinned with a golden test of the error constructor, which conflicts with the rule against code for impossible states.

This is abnormal because the plan passed its Plan-Bouncer and Plan-Burler rounds with a prescribed refusal that is unreachable by construction, and the contradiction surfaced only after the code was written.

What to do: have plan review (or the plan-time checks) verify that every refusal a card prescribes is reachable from some input that passes upstream validation, or require the card to say how a test reaches it.

Provenance: task `webster-replan-state`, from note `burler-webster-r1.md`.
