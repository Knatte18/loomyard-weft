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

### Added after Discussion (arbiter, 2026-11-26): A removal record names the cancel only when a cancel wrote it

Decision: A pending remove record names `lym batten cancel <slug> --remove` as its re-run only when the removal's caller named that command (the record's Rerun field, set by the cancel's removePairAndBranch). A record written by `lym fabric remove` or another caller with no rerun (dobby, beam-fork, the batten teardown rows) keeps naming `lym fabric remove <slug> --remote`, whether or not its pair is gone. The board entry's Idea condition 'when the record's pair is gone' is not built, and no review round should raise it again.

Rationale: The board entry's title, brief and Seen section scope the bug to a record a killed batten cancel left, which the change fixes directly through the record's Rerun field, so the pair-gone test the Idea named is not needed to recognise such a record. Naming the cancel on a record no cancel wrote would point at a re-run that refuses: refuseNothingLeft counts only a pending record whose Rerun is this cancel's command, and a slug with no batten run has nothing for the cancel to finish. Telling a gone pair from one on origin would also cost `lym fabric status` a network call per record. Building the pair-gone case for other callers would mean a new way to settle their records, which goes beyond the task's scope.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
