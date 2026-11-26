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

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
