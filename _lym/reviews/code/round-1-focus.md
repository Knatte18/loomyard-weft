---
round: 1
focus:
  - "internal/sheddriver/drive.go `handOver` and the amended board entry's Idea: the diff drops the dirty-tree refusal and hands over unconditionally. Settle whether that makes the change the entry asks for (a blocked hand-over must not leave the old binary rendering new board copies), and whether any path still lets the old driver render after BuildChanged is true."
  - "internal/sheddriver/drive.go `handOver`: with the guard gone, settle whether the successor can safely re-step over a producer's uncommitted output, and whether the once-per-stop-key-and-build rule in `handedOver` still prevents a hand-over loop."
  - "internal/loomcli/drivewiring.go and internal/sheddriver/drive.go: settle that no dangling reference, unused import or now-dead helper remains after `DirtyPaths` and `dirtyPathsOutsideRecords` were removed, and that `loomengine.PathsOutsideRunRecords` still has its other caller."
  - "internal/loomcli/doc.go hand-over paragraph and the `handOver` doc comment: settle whether the text matches the new behavior, and whether any other doc, spec or comment still states the clean-tree precondition."
  - "internal/sheddriver/drive_test.go `TestGoDriver_Settle`: settle that the removed dirty-tree row and `dirty` field leave no behavior uncovered, and that the new hand-over behavior (hand-over over uncommitted paths) is covered or reached by an existing row."
  - "Commit messages of the range: settle whether the claims in `0efccb83` are borne out by the diff, and that no `Pattern-Departure:` trailer is needed."
---

The diff is small: it removes the `DirtyPaths` seam field, its wiring, its test row and its report text, and rewords two doc comments.
The decision record holds only the board entry under `## Goal`, with no added entries, so the entry's Idea is the measuring stick.
