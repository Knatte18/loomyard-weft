## Batch 01-board-view-projection, attempt 2

Attempt 1 committed cards 1-3 and left card 4's internal/boardserve and the docs/overview.md line uncommitted when it timed out.
This attempt checked that work against the card, ran its gate and committed it unchanged.
boardserve splits the handler into server.go (checks, routes, long poll) and write.go (applyWrite and its op rules).
The create op also refuses any kind but task or note, as the decision record says the page never creates a project.
Card 5 builds the lifecycle on Server.BoardChanged, beginDrain and lastAPIRequest. ServerConfig.Now is read at NewServer and on every /api request.
The batch gate takes over 4 minutes in tier1, so its first call answers done: false.
