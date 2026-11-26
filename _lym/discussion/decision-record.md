## Goal

```markdown
## Seen (operator, 2026-11-26)

The board README does not show whether autonomy is on for a beam, for the root, or hub-wide.
The operator has to look in `beam.yaml` files to know which halts and pull requests reach them.

## Idea

In the README's project sections, show for each beam project and for the root (`P0`):

- autonomy: on or off, and where the value comes from (the beam's own `beam.yaml`, the root, or the hub-wide default);
- the pull-request setting of `beam.yaml` `pull_request` as it resolves for the beam's members;
- the beam's parent branch, and the beam's own open pull request as a link when it has one.

State the hub-wide value once at the top when it applies to every beam.
Read the values the way the runs resolve them, from each beam's worktree up, so the README never disagrees with what a run does.
`lym board list --text` may show the same in the project's row.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-26): Mark inherited values apart from local overrides

Decision: Every value the README shows that a beam can inherit from its parent (autonomy, the pull-request setting) shows whether it is inherited or the beam's own local override, for example by colour, with the legend stated once in the README.

Rationale: Operator, 2026-11-26: the operator must see at a glance where each value comes from.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - absent-beam-worktree-shows-parent-values (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
