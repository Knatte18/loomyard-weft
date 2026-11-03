## Goal

```markdown
## Seen (operator, 2026-11-03)

Task `comment-cleanup` must start only after project `codex-engine`, a beam, has landed in `main`.
`depends_on` refuses a project ("nothing depends on a project: depend on its members instead"), and a member landing into the beam is not the beam landing into `main`.
So the task was set back to draft and the hub orch watches the project by hand, a watch that dies with its session.

## Idea

Dobby already holds a task until every slug in its `depends_on` has landed; let that hold work for every kind of entry that lands.

- `depends_on` accepts a project's slug, and the dependency is met when the project is done: a beam when it has landed into its parent, a plain project when every member has.
- Any other slug that ends in a landing counts the same way, so the rule reads "a dependency is met when that entry is done", whatever its kind.
- The README's After column and Ready subsection show a project dependency like a task one.
- `set-deps`' overlap notice and its cycle check cover project dependencies too, a project counting as the union of its members.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - after-cell-project-untested (class design, severity MEDIUM, open in rounds 1)
  - membership-recursion-unguarded (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
