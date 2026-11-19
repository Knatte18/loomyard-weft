---
round: 2
focus:
  - "internal/boardserve/run.go watchBoard: settle whether a burst of fswatch events can cause more than one read and hash of board.json per second, and whether the idle backoff doubles from one second to eight and resets on a change."
  - "internal/boardengine/board.html poll and internal/boardserve/server.go serveBoard and waitForChange: settle whether a board with no board.json (empty revision) can make the page re-request and re-render without waiting."
  - "internal/boardserve/server.go ServeHTTP: settle whether a request that fails the bearer check can advance the idle clock."
  - "internal/boardserve/write.go checkRequest and internal/boardengine/expect.go checkExpected: settle what a create with a revision, and a create naming a project's slug, each answer."
  - ".gitattributes: settle whether every go:embed target, board.html included, is pinned to LF."
  - "internal/boardcli/serveopen.go serveCommand and productionOpenDeps: settle whether the orch-strand pass, the empty strand name on the server's board, the OpenerCap mapping and the Register role are each pinned by a test."
---

Round 1 left these areas changed by fixes; each entry restates its own sites.
