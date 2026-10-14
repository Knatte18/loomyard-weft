# Recovering batch 07: webster rejects a finished batch whose last card renames packages

Every card in the batch (7–12) was committed, and card 12 moved `internal/pausecli` and `internal/pauseengine` to `internal/tendercli` and `internal/tenderengine`, as the plan intends.
Webster's create and rename checks then looked up cards 8–10's `plan:internal/pausecli#…` and `plan:internal/pauseengine#…` handles at batch HEAD and reported them `not_found`.
Every flagged unit resolves under the new package names.

The batch was rejected again on each recovery at the same HEAD, with nothing left to change in code: the only way to satisfy the check is to undo card 12.
The check likely needs to read each card's handles at that card's own commit, or follow a later card's package rename.
