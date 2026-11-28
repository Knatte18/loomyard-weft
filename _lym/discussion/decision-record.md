## Goal

```markdown
## Seen

`docs/research/hardener-works-study.md` (`c6618b538`), section "Faults found by this study", read these in the code; none was driven:

- a refused credential during a one-repository seed keeps the hub and says origin may still take the push;
- an empty teardown job clears an earlier teardown's pending record;
- the `cleanup` remote sweep counts local tags as landed and never pushes them;
- `--reset` is advised without regard to unpushed work;
- `fetchRunForCatchUp` reads no records layout;
- a `lym fabric remove` pending record can outlive its pair (`cd8090733` fixed only the cancel's records, by choice).

## Idea

Confirm each against the code with a test that fails first, and fix the confirmed ones.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - catchup-no-pair-call-site-untested (class design, severity MEDIUM, open in rounds 1)
  - doc-overstates-settle-callers (class consistency, severity MEDIUM, open in rounds 1)
  - existing-hub-advice-partial-clone (class design, severity MEDIUM, open in rounds 1)
  - reset-refusal-local-tags-and-pending-teardowns (class design, severity MEDIUM, open in rounds 1)
  - reset-refusal-misses-prime-uncommitted-stash-detached (class design, severity MEDIUM, open in rounds 1)
  - summary-merge-danger-understated (class consistency, severity MEDIUM, open in rounds 1)
  - sweep-failed-tag-push-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
