## Goal

```markdown
## Seen

The study `docs/research/speed/gate-fixed-cost-2026-11-27/report.md` timed a tier1 call on one small package at about 6 s before its tests.
The `lym` build in `prebuildLym` (`internal/gatecli/prebuild.go`) takes 2.4 to 2.8 s of that on every run of a tier with `prebuilt_lym`,
whether or not a test of the targeted packages uses the binary.

## Change

Build `lym` only when a test file of a targeted package imports `internal/testkit/lymbin`.
First check every route by which a test reaches the prebuilt binary, and count each of them, so no test loses the binary it needs.
A call over the whole set still builds it.

The hub's tier1 command change from the same report (`go vet ./...` in place of `go build ./... && go vet {targets}`) is hub config, set apart from this task.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - prebuild-check-ignores-whole-command-tier (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
