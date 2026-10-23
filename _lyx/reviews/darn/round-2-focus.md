---
round: 2
focus:
  - "internal/boardengine/board.go, ownerset.go and replay.go: each production call that renders the board README passes the loaded store's prime owner; settle whether a test asserts the rendered README's prime-owner line at the board write path, the owner setter and the replay, so an empty owner at any of the three fails a test"
  - "internal/boardengine and internal/boardcli: settle whether Board.PrimeOwner on a board directory that does not exist is covered by a test, and whether the code keeps a branch the loader already makes redundant"
  - "internal/boardcli/cli.go find verb help text: settle whether it still claims the same JSON as lyx board list, now that list JSON carries prime_owner and find JSON does not"
---

Round 2 re-reviews the fixes the writer made for round 1's findings, as fresh text.
