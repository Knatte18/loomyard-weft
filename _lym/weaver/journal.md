## Batch 01-board-view-projection, attempt 2

Attempt 1 committed cards 1-3 and left card 4's internal/boardserve and the docs/overview.md line uncommitted when it timed out.
This attempt checked that work against the card, ran its gate and committed it unchanged.
boardserve splits the handler into server.go (checks, routes, long poll) and write.go (applyWrite and its op rules).
The create op also refuses any kind but task or note, as the decision record says the page never creates a project.
Card 5 builds the lifecycle on Server.BoardChanged, beginDrain and lastAPIRequest. ServerConfig.Now is read at NewServer and on every /api request.
The batch gate takes over 4 minutes in tier1, so its first call answers done: false.

## Batch 05-board-server-lifecycle, attempt 1

Cards 5 and 6 are committed and the batch gate passed; TestBoardServeScenario ran and passed in tier1.
`lym board open` (operator) and `lym board serve` (internal) exist in boardcli.Command; their logic is in internal/boardcli/open.go.
boardengine now exports EntryStoreFile (no card named it) so the watch loop does not repeat "board.json".
Card 4's temporary testonlycode allowlist entries are removed.
A test file named *_linux_integration_test.go gets no platform constraint from its name; it needs `//go:build linux`.
A test that needs lock.AcquireWriteLockWithin to succeed needs a generous wait: a 10 ms one can expire before the first attempt under load.
The card gate's comment lint runs a fixer that rewrites comments, so read changed files again after a gate call.
