## Goal

```markdown
## Seen

Live drive of the one-repository sandbox (`docs/research/single-repo-undriven-cases.md`).
`lym batten cancel <slug> --remove`, killed while its detached `lym fabric remote-teardown` child stalled on an origin branch delete,
leaves a pending record of kind `remove`.
`lym fabric status` lists it with the way forward `... run \`lym fabric remove <slug> --remote\` again`.
Run once the pair was gone, that command answers `{"error":"pair not found: nothing of \"disc1\" remains","ok":false}`.
The command that finished the teardown was `lym batten cancel <slug> --remove`.

## Idea

When the record's pair is gone, or the record came from a batten cancel, name `lym batten cancel <slug> --remove` as the re-run,
the way the `discard` record's `rerun` field already does.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
