## Goal

```markdown
Imported from [issue #98](https://github.com/Knatte18/loomyard/issues/98).

In a hub whose anchor is a subdirectory (Models, anchor src/csharp/NORCE.Models), the mo:FABRIC notice for a failed async push says: way forward: run `lym fabric push` in <hub>/knut-rheology-merge-in. Running it there fails with: cwd is outside the recorded fabric anchor subtree: cwd <hub>/knut-rheology-merge-in does not equal the anchored directory <hub>/knut-rheology-merge-in/src/csharp/NORCE.Models (recorded by .lym-anchor).

Workaround: run `lym fabric push` in the anchored directory instead; it pushed.

Expected: the way forward names the anchored directory (or uses env -C with it), so the command it gives works as written.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - lymverbdir-calls-lack-anchored-tests (class design, severity MEDIUM, open in rounds 1)
  - unconverted-lym-steps-weftdirtystop-upstreamanchorstop (class scope, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
