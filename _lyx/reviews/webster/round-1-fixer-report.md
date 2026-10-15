## Changes

- F1 (commit `d96064700`): fixed as proposed.
  `internal/shuttleengine/spec.go` now says `whatever claude.yaml's deny_agent_tool says`.
  `internal/shuttleengine/claudeengine/settings_test.go` names `DenyAskUserQuestion` and `DenyAgentTool` in the two comments.
  Comment text only; the tests were not re-run.
- F2 (commit `10c4a59bf`, empty): the trims stand, the default the review names.
  The background files still carry every trimmed rule, and the headroom lint needed the room for the new entry.
  The choice is recorded in the commit message; no file changed.

## Disputed

None.

## Deferred

None.
