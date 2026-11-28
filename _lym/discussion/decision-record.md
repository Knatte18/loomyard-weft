## Goal

```markdown
## Seen

The `design-first-route` landing (`9acc400f8`) listed a mis-spaced assignment in `internal/battencli/wire_test.go`,
`err =c.env.SeedChild.WriteSeed`, which gofmt would rewrite.
That line does not stand in the file at `9acc400f8` or at `3fb0787e3`, and `gofmt -l cmd internal tools` lists no file at `3fb0787e3`.
No gate checks formatting, so a mis-formatted Go file passes every tier.

## Idea

Add a gofmt check to tier0 (for example `gofmt -l` over the changed files).

## Orch decision

Check every Go file under `cmd`, `internal` and `tools`, not only changed files: the tree is clean today, so the whole-tree check stays cheap and needs no diff scope.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - gofmt-guard-passes-ending-only-difference (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
