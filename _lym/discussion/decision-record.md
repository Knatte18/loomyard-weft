## Goal

```markdown
The shipped heddle entries `lym-batching` and `lym-plan-edits-before-gate` (`internal/heddleengine/shipped.json`) render into every worktree's `CLAUDE.local.md` as walls of prose; the operator judges that agents do not take such text seriously and asks for short imperative bullet points (operator, 2026-11-03).

## Expected

- Each shipped entry opens with a `##` heading of its own under the `# LoomYard's own instructions` section heading, and its text is a list of short imperative bullets, one instruction per bullet.
  A numbered list stays numbered where order is the point, as in the round of edits.
- Every instruction the entries give today survives the rewrite with its meaning unchanged, including the override of the Read tool's advice to read only a part, the reread after a command rewrites files, and the four steps of a round.
- Repeated statements are said once: the rule against reading one file in slices appears in both entries today.
- The rendered file has no heading-level skip: the shipped entries' `##` headings sit under the `#` section heading, as a repository entry's `##` heading sits under `# This repository's instructions`.
- Tests that pin the shipped text or its render follow the new text; any stencil or doc that quotes the old sentences is updated in the same change.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
