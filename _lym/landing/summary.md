# board upsert --record-file replaces a Sketch's stored record

`lym board upsert` gains `--record-file <file>`, which replaces an entry's stored decision record in the same board write that lands the payload's other fields.
Until now a Sketch's body and record could only drift apart: editing the body left the stored record as the Discussion wrote it.
The operator can now keep both current in one write, and raddle reads the record it has always read.

Every write of a stored record also stamps the entry with `record_body_sha256`, the sha256 of the body that write leaves.
`lym board promote` of a Sketch warns when the body it loaded no longer matches that stamp.

## Behaviour

- `--record-file` is admitted only for an existing entry that carries a stored record.
- It is refused, with a way forward and nothing written, for:
  - a new entry, an entry with no stored record, and a done entry;
  - an entry a run holds, and an entry whose slug has a task worktree in the hub;
  - an entry whose weft branch is live, and an upsert whose origin probe fails;
  - a caller that is a run's task session, by the board's existing strand-name classification of a run's agent;
  - a file that fails the record's format checks.
- The run and worktree refusals point to `lym loom decision replace <slug>` or `lym batten cancel <slug>`.
- A family project whose raddle filing is done is exempt from the worktree and live-weft refusals:
  its run reads no record again.
- The record file is written before `board.json`, as `StoreDecisionRecord` does.
  When the save fails, the former record file is written back.
- The promote warning never refuses.
  An entry with no `record_body_sha256`, stored before this change, gets no warning.

## Design decisions

- `fabricengine.Topology.RecordsBranchLive` is a new exported function that answers `Add`'s weft liveness test alone.
  `Add` now routes through the same `weftLiveness` helper, so the upsert and a raddle start cannot disagree.
  A local branch decides alone, and origin is probed only when `Add` would probe it (not under skip-git or skip-push).
  `internal/boardcli` runs no git; it takes the function through the board.
- `discussionparser` gains a record-only section check, because the board keeps no support log.
  The decision-id and acceptance check is reused.
  `raddleparser` still reads records through `discussionparser` alone.
- `boardengine.Board.PromoteTo` now returns a `PromoteResult` (task, siblings, `RecordWarning`) in place of its three return values.
  The body compared is the one loaded before the promote's write.
- `record_body_sha256` is a new tolerantly read key of `board.json` and is carried by board-sync replay.
- The run-agent refusal reads the strand name from the environment, so it guards against a mistaken call, not a forged or unset name.
  The flag does not see a batten start after its check or a batten run on another machine.
- The warning fires on any body change after the store, including one that leaves the design alone.
  A record stored by an older binary leaves the stamp as it was, so the warning can be wrong until a newer binary stores the record again.
- The `board` skill in `plugins/ly/skills/board/SKILL.md` documents the flag.

## To check by hand

- The family-project exemption (`judgeRecordReplaceIn` in `internal/boardengine/recordupsert.go`) applies only when the project's kind, its raddle filing and the filed project all match.
  Check that tests pin each leg of that condition, not only the all-true case.
- Check that board-sync replay of `record_body_sha256` (`replayFields` in `internal/boardengine/replay.go`) is covered for an entry whose stamp differs between the two sides.
- Nothing here was exercised against a real origin outside the fabricengine integration test for the weft probe.
