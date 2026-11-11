## Goal

```markdown
Imported from [issue #107](https://github.com/Knatte18/loomyard/issues/107).

## What happened

In a beam, the orch promoted a Sketch with `lym board promote S1`, which made it a task with recipe `raddle`.
It then ran `lym board set-deps '{"slug":"<slug>","depends_on":["<other>"]}'`, where `<other>` was a task of the same beam still `running · PR-Review`, and only after that released the task with `lym board set-status '{"slug":"<slug>","status":null}'`.
`lym board get` showed `depends_on` set and the status already `running · Seed-Child` moments later: the beam's dobby started the raddle task although its dependency had not landed.

## Workaround

`lym batten pause <slug>` holds the run; the orch resumes it with `lym batten run <slug>` once the dependency lands.

## Expected

Dobby starts a released task only when every entry in its `depends_on` is done or gone, whatever its recipe, `raddle` included.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - board-skill-missing-promote-depends-on (class scope, severity MEDIUM, open in rounds 1)
  - promote-sibling-notice-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
