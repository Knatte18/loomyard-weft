## Goal

```markdown
## What was seen (2026-11-22)

Round 1 of `harden-fabricengine` built a one-repository sandbox hub with `lym harden sandbox new --single-repo` and drove it live.
Every mutating `lym fabric` verb was refused from its round agent: add, push, pull, sync, reconcile, checkout, remove, clone and cleanup --apply.
The refusal is `sandboxedRoleRefusal` in `internal/fabriccli/roleguard.go`, which reads only the strand name, so it also refuses verbs run inside the round's own sandbox hub.
Seven of twelve charter items ended as could not verify: the join, the refusals, two clones, the remote delete, pair crash and rebirth, URL variations and every pair verb.

## Idea

Let a hardener round's agent run the mutating fabric verbs in a hub under its own sandbox root, `<worktree>/.lym/harden-sandbox`, and keep the refusal everywhere else.
The guard then checks the hub the verb resolves, not only the strand name, so a verb aimed at the real hub stays refused.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - sandbox-root-through-records-sibling-link (class design, severity BLOCKING, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
