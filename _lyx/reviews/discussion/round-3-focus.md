---
round: 3
focus:
  - "Gate and parse order in the reviewer/fixer split: `ReviewGateEntry` in `internal/burlerengine/verdict.go` is `PassOnCap: true`, so an exhausted budget still reports the gate as passed.
    Settle whether every Decision and acceptance criterion that depends on a reviewer gate failure, a marker precondition or the point where the review is parsed matches what that entry and the strict parse after it can actually produce."
  - "Stopping a blocked run: a `shuttleengine.Run` offers `Wait`, `Interrupt`, `Send` and `StrandGUID`, and `burlerengine`'s `Shuttle` seam carries no strand removal.
    Settle whether the record names the capability Go needs to stop a peer run and remove its strand, and which half's outcome the round returns when the stopped half then ends as `died` or errors."
  - "Marker lifecycle: the marker path is derived from the review path, written by the goroutine waiting on the reviewer, and removed before a new attempt.
    Settle whether a write from a stopped attempt can release a later attempt's fixer, and whether the marker path and its pre-existing-marker cleanup are defined for both a hub round and the standalone `lyx burler run`."
  - "Resume classification: `AttachGated` and `AttachIfLive` wait on a live run they find.
    Settle whether the record says how each half's liveness is observed without waiting, how both live halves are attached concurrently, and what happens to the marker in the both-live branch."
  - "Fixer write surface: the overlay fix scope names the review path as writable, and the verdict is parsed from that file.
    Settle whether a Decision bounds what the fixer may write to the files that the verdict and the segment judge read."
---

Focus entries carry the constraint-bearing questions only; the rubric's `Do not flag` list was checked against each.
