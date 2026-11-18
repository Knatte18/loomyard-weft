## Goal

```markdown
## Seen (retro, 2026-11-17)

From `docs/research/retro/retro-20261116-115411/report.md`, proposal 3.
Three refused compound commands opened with a probe or scratch step (`sed -i.bak` on `/dev/null`, `python3 -V`, `rm -rf $d`) followed by useful read-only work; the guard refused the whole command and the next turn re-ran the parts still needed.
Moments: `test-only-functions-in-prod` agent-ae723f78 turn 7, `codex-same-as-claude` agent-acbe71dc turn 3, `decision-record-replace` 2c77f780 turn 21.

## Idea

The shell guard's refusal says that nothing in the command ran and names the refused segment.
First check whether the current text already says so.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - contained-test-missing-keep-marker (class consistency, severity BLOCKING, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
