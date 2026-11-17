## Goal

```markdown
Imported from [issue #99](https://github.com/Knatte18/loomyard/issues/99).

## What was seen

`fabric.branch_prefix` can be set only on the hub's default layer, which is committed in the board repository and applies to every machine and user of the hub.
`lym config fabric --layer machine --set branch_prefix=x/` is refused with "fabric has no machine key",
and `--layer local` is refused with "key branch_prefix is cross-pair and cannot be set in a local override".

A branch prefix usually names the person who owns the branches, such as `<user>/<slug>`.
On a hub shared by several people, or one person on several machines with different conventions, one hub-wide value does not fit.

## Idea

- Let `branch_prefix` be set per machine, or per user, with the hub default as the fallback.
  Since #59, teardown uses the recorded branch name, so a pair's branch no longer depends on the prefix in force later; a pair created on one machine keeps its name when adopted on another.
- Review which other keys are personal rather than hub-wide and give them the same layer, for example:
  the identity a pull request is opened under or assigned to, the Slack operator, the default model or effort a session opens with, and editor or window settings such as `batten.open_ide`.

## When (operator, 2026-11-16)

Planned after the beam `adopt-build` lands, as it reworks how a hub moves between machines and touches fabric and the config layers.
Then it becomes a task: the per-machine or per-user layer for `branch_prefix`, and the review of the other personal keys above, built on what the beam left.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-17): The other personal keys keep their layers

Decision: The review of the other personal keys is done and none of them moves to the per-machine layer: the pull request identity is gh's login, the Slack operator is recorded by lym slack setup in a per-machine credential file, and batten.open_ide, seed_ide, open_window, landing.pr_reviewer and agents.efforts stay category None.

Rationale: The machine category closes a key to the local layer and to a task's run_settings, so moving those keys would take away the per-worktree and per-run setting they have now; a per-machine default for category-None keys would need a change to configengine's layer rule, which is outside this task.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
