# Test tiers: an llm-tier gate, a guarded publish_verify, a stricter comment lint

`test-tiers-hardening` · Note · enhancement, loom

From test-tiers PR-Gate (2026-10-07):

- No gate runs the `llm` test tier.
- A task can empty its own `publish_verify` in the pair`s `landing.yaml`, and the hub learns it only after landing, since per-worktree config never lands.
- The comment lint lets every comma-plus-conjunction break through, so the clause-boundary rule is still judged by reviewers.