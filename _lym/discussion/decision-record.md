## Goal

```markdown
## Seen

The landing of `hardener-study-faults` (`7cb004818`) says `lym fabric clone --reset` still removes unpushed records and board commits,
uncommitted Bolt changes and ignored files, and leaves open whether that is acceptable.

## Operator decision (2026-11-28)

It is not acceptable: `--reset` refuses when it would lose work, and names what would be lost.

## Idea

Before `--reset` removes anything, check for unpushed records and board commits, uncommitted Bolt changes and ignored files that hold work.
When any is found, refuse with the list and the way forward: push or commit them first, or pass an explicit flag that accepts the loss.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - bug-fix-lacks-root-cause-trailer (class design, severity MEDIUM, open in rounds 1)
  - discard-work-cli-pass-through-untested (class design, severity MEDIUM, open in rounds 1)
  - reset-misses-bolt-checkout-head-only-commits (class design, severity MEDIUM, open in rounds 1)
  - reset-unreadable-store-refusal-has-no-way-forward (class design, severity MEDIUM, open in rounds 1)
  - sandbox-core-suite-claims-idempotent-reset (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
