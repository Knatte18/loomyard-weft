# lyx gate test stops at the first failing tier, so a pre-existing tier0 failure hides every later tier and the guard step

`gate-runs-past-failing-tier` · Task · bug, gate · running · Code-Burler-Review

Imported from [issue #864](https://github.com/Knatte18/loomyard/issues/864).

What happened: a pre-existing tier0 failure (a CLI-form test failing on a command's Long text over its byte cap, unrelated to the batch) made `lyx gate test` stop at the first failing tier.
The card gate and the batch gate therefore never reached the guard step or tier1, so a retag's tier1 compile, its vet steps under every tag set and its guard step got no check from either gate.
The agent covered the gap by hand with `lyx gate test --wait --tier 1 --card <card>`.

Why abnormal: a failure the batch did not cause silently removes the verification of everything after it, and the gate reports no sign that those steps were skipped.

What to do: either let the gate run later tiers and the guard step past a failing tier (reporting all failures), or state explicitly in the gate output which tiers and steps were not reached.
A failure present at the base should not blind the gate for a batch that did not cause it.

Provenance: task gate-runs-once; notes 22-tier-plan-docs-recovery.md