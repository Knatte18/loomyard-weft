---
round: 2
focus:
  - "internal/boardengine/raddlefile.go FileRaddle writes board.json first and the raddle store directory, store.json last, in afterSave: settle whether a retry after a failure or crash between those two writes finishes the store write without re-running the board check, and whether the loomshed raddleFile call's verify step still bounces the run's own already-filed slugs as taken."
  - "internal/loomshed/recordwrite.go Call chooses between the stored-record support log and the board-rendered one: settle whether the choice follows whether the entry has a stored record on every path where the log is missing, including a resume where the record file already exists."
---

The first entry targets the resumable filing path and the second the support-log choice on resume; both are code paths the round's fixes change.
