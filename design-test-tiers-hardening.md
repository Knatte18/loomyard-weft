# Test tiers: a stricter comment lint, a reviewer range that covers the merge of main

`test-tiers-hardening` · Note · enhancement, loom, burler

From test-tiers PR-Gate (2026-10-07):

- The comment lint lets every comma-plus-conjunction break through, so the clause-boundary rule is still judged by reviewers.

From shuttle-send's Publish blocks (2026-10-08):

- A Burler reviewer reads only `git diff $(git merge-base main HEAD)..HEAD`, so a defect that arrives through the merge of main is outside its range even when Publish failed on it; a goto from a failed Publish needs the failure handed to the round.
