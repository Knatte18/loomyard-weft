---
round: 4
focus:
  - "Card 3 resolves the prime through `PrimeName(l)`, which runs `git worktree list` in `l.AnchorPath()` (`<worktree>/<AnchorRel>`): settle whether every existing test that calls `Fabric.Commit`, `PushPairAnchored` or the `sync` verb with an anchor subdirectory that exists only on the records side (for example `internal/fabricengine/commit_integration_test.go` and `internal/webstercli/sync_integration_test.go`) is named in some card's Edit list with the fixture change that makes the code worktree contain that directory."
  - "Cards 2 and 3 change statements that the code and docs make about rollback and about pushing both sides: settle whether each comment, `Short` string, header and `doc.go` paragraph that states the old rule (including `internal/fabricengine/coalesce.go`, `internal/fabricengine/commit.go`, `internal/fabriccli/weft_verbs.go` and the `doc.go` paragraph on token-carrying ownership kinds) is named by the card that falsifies it."
  - "Deleted tests: settle whether any surviving comment or doc in the files the plan edits still cites by name a test function that a card deletes."
---

Focus targets the plan as it stands after the round-3 fixes; each entry asks the reviewer to verify against the repository, not against earlier review text.
