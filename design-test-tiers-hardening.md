# Test tiers: an llm-tier gate, a guarded publish_verify, a stricter comment lint

`test-tiers-hardening` · Note · enhancement, loom

From test-tiers PR-Gate (2026-10-07):

- No gate runs the `llm` test tier.
- A task can empty its own `publish_verify` in the pair`s `landing.yaml`, and the hub learns it only after landing, since per-worktree config never lands.
- The comment lint lets every comma-plus-conjunction break through, so the clause-boundary rule is still judged by reviewers.

From shuttle-send's Publish blocks (2026-10-08):

- The Webster-Burler verify gate runs build, vet, `go test ./...` and the integration tier, never the `publish_verify` (tmux) tier, so a defect in that tier surfaces only at Publish, after the review loop has converged.
- A Burler reviewer reads only `git diff $(git merge-base main HEAD)..HEAD`, so a defect that arrives through the merge of main is outside its range even when Publish failed on it; a goto from a failed Publish needs the failure handed to the round.