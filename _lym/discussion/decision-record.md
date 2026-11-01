## Goal

```markdown
## Seen (operator, 2026-11-01)

The README opens with a `## Prime` section ("The root project: the entries under no other project.") that wraps P1 and P2, and further down a `### No project` section holds root's own entries.
"Prime" and "No project" name the same thing, and the `No project` heading looks nothing like the project headings: it has no link to a project file, no facts line with age and owner, and no brief.

## Idea (operator, 2026-11-01)

- Root is a project like the others and always has the handle `P0`; the board's own project handles start at `P1`, so `P0` never clashes.
- Root gets its own project file, `project-root.md`, rendered the way `project-<slug>.md` is rendered for P1 and P2: its tasks by layer, its notes by type and its done entries.
- The README stays the full overview of every project, with every table it shows now, not a summary.
- Every project, root included, opens with the same kind of heading: `P<n> <title>` linked to its project file, then its facts line and brief.
  Root's heading is `P0 Root: entries under no project`; its facts line names the prime owner and has no design link, since root has no board entry.
- Root's section comes last in the README, after every other project.
- The `## Prime` wrapper and the `No project` heading go, and so do the "Own entries" heading in a project file with sub-projects if it duplicates this layout; heading levels are made consistent once the wrapper is gone.
- The README's and project files' tests and goldens change with the layout.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - root-slug-collides-with-root-project-file (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
