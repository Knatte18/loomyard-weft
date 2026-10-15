## Changes

- F1 (fixed as proposed, commit `0796c0115`): the doc comment of `buildLoomShed` in `internal/loomcli/sharedbootstrap.go` no longer claims it holds the fabric-open and landing-assembly block.
  It now says it assembles `Env.Landing` through `wireLanding` and then constructs the shed via `loomrecipe.New`; the closing sentence about the return values is unchanged.
- Gate finding (fixed): the F1 comment was wrapped at a fixed column.
  It is now one sentence per line, in a follow-up commit.
- Both changes are comment-only, with no build or test run.

## Disputed

None.

## Deferred

None.
