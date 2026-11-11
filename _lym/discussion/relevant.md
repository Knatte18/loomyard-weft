# Relevant files

- `internal/loomcli/decision.go` `decisionVerb`, `checkDarnRecord`, `copyRecordToBoard`, `decisionStoreVerb`, `perchFilingRefusal`, `committedLacksEntry` — the `decision` verbs that `replace` joins and whose guards it shares
- `internal/discussionparser/append.go` `AppendDecisions`, `CheckAddedDecisionsKept`, `checkDroppedAddedDecisions`, `checkDroppedPatternBullets` — the appended-entry writer and the dropped-entry history check
- `internal/loomshed/gates.go` `DiscussionCheck`, `ValidateDiscussion` — runs the history check over the versions the History seam returns
- `internal/loomcli/projectrecord.go` `recordHistory` — the history seam that D2 bounds
- `internal/loomcli/subjects.go` `documentSubject` — hashes the record through its identity text for Bouncer re-entry
- `internal/boardengine/decision.go` `Board.StoreDecisionRecord` — the only writer of a stored record and the discussed flag
- `internal/boardcli/bodyfile.go` `applyBodyFile` — the `--body-file` pattern that `--record-file` follows
